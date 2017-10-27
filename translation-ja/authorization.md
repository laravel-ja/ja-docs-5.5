# 認可

- [イントロダクション](#introduction)
- [ゲート](#gates)
    - [ゲートの記述](#writing-gates)
    - [アクションの認可](#authorizing-actions-via-gates)
- [ポリシーの作成](#creating-policies)
    - [ポリシーの生成](#generating-policies)
    - [ポリシーの登録](#registering-policies)
- [ポリシーの記述](#writing-policies)
    - [ポリシーのメソッド](#policy-methods)
    - [モデルを持たないメソッド](#methods-without-models)
    - [ポリシーフィルタ](#policy-filters)
- [ポリシーを使ったアクションの認可](#authorizing-actions-using-policies)
    - [Userモデルによる認可](#via-the-user-model)
    - [ミドルウェアによる認可](#via-middleware)
    - [コントローラヘルパによる認可](#via-controller-helpers)
    - [Bladeテンプレートによる認可](#via-blade-templates)

<a name="introduction"></a>
## イントロダクション

Laravelは組み込み済みの[認証](/docs/{{version}}/authentication)サービスに加え、特定のリソースに対するユーザーアクションを認可する簡単な手法も提供しています。認証と同様に、Laravelの認可のアプローチはシンプルで、主に２つの認可アクションの方法があります。ゲートとポリシーです。

ゲートとポリシーは、ルートとコントローラのようなものであると考えてください。ゲートはシンプルな、クロージャベースのアプローチを認可に対してとっています。一方のコントローラに似ているポリシーとは、特定のモデルやリソースに対するロジックをまとめたものです。最初にゲートを説明し、次にポリシーを確認しましょう。

アプリケーション構築時にゲートだけを使用するか、それともポリシーだけを使用するかを決める必要はありません。ほとんどのアプリケーションでゲートとポリシーは混在して使われますが、それで正しいのです。管理者のダッシュボードのように、モデルやリソースとは関連しないアクションに対し、ゲートは主に適用されます。それに対し、ポリシーは特定のモデルやリソースに対するアクションを認可したい場合に、使用する必要があります。

<a name="gates"></a>
## ゲート

<a name="writing-gates"></a>
### ゲートの記述

ゲートは、特定のアクションを実行できる許可が、あるユーザーにあるかを決めるクロージャのことです。通常は、`App\Providers\AuthServiceProvider`の中で、`Gate`ファサードを使用し、定義します。ゲートは常に最初の引数にユーザーインスタンスを受け取ります。関連するEloquentモデルのような、追加の引数をオプションとして受け取ることもできます。

    /**
     * 全認証／認可サービスの登録
     *
     * @return void
     */
    public function boot()
    {
        $this->registerPolicies();

        Gate::define('update-post', function ($user, $post) {
            return $user->id == $post->user_id;
        });
    }

コントローラのように、ゲートは`Class@method`形式のコールバック文字列を使い定義することも可能です。

    /**
     * 全認証／認可サービスの登録
     *
     * @return void
     */
    public function boot()
    {
        $this->registerPolicies();

        Gate::define('update-post', 'PostPolicy@update');
    }

#### リソースゲート

`resource`メソッドを使用すれば、一度に複数のゲートを定義できます。

    Gate::resource('posts', 'PostPolicy');

これは次のゲート定義とまったく同じです。

    Gate::define('posts.view', 'PostPolicy@view');
    Gate::define('posts.create', 'PostPolicy@create');
    Gate::define('posts.update', 'PostPolicy@update');
    Gate::define('posts.delete', 'PostPolicy@delete');

デフォルトとして、`view`、`create`、`update`、`delete`アビリティが定義されます。`resource`メソッドに第３引数として配列を渡し、デフォルトのアビリティのオーバーライドや追加ができます。配列のキーでアビリティの名前、値でメソッド名を定義します。例として、以下のコードで新しい`posts.image`と`posts.photo`のゲート定義を作成してみます。

    Gate::resource('posts', 'PostPolicy', [
        'image' => 'updateImage',
        'photo' => 'updatePhoto',
    ]);

<a name="authorizing-actions-via-gates"></a>
### アクションの認可

ゲートを使用しアクションを認可するには、`allows`と`denies`メソッドを使ってください。両メソッドに現在認証中のユーザーを渡す必要はないことに注目しましょう。Laravelが自動的にゲートクロージャにユーザーを渡します。

    if (Gate::allows('update-post', $post)) {
        // 現在のユーザーはこのポストを更新できる
    }

    if (Gate::denies('update-post', $post)) {
        // 現在のユーザーはこのポストを更新できない
    }

特定のユーザーがあるアクションを実行できる認可を持っているかを確認するには、`Gate`ファサードの`forUser`メソッドを使用します。

    if (Gate::forUser($user)->allows('update-post', $post)) {
        // 渡されたユーザーはこのポストを更新できる
    }

    if (Gate::forUser($user)->denies('update-post', $post)) {
        // ユーザーはこのポストを更新できない
    }

<a name="creating-policies"></a>
## ポリシー作成

<a name="generating-policies"></a>
### ポリシーの生成

ポリシーは特定のモデルやリソースに関する認可ロジックを系統立てるクラスです。たとえば、ブログアプリケーションの場合、`Post`モデルとそれに対応する、ポストを作成／更新するなどのユーザーアクションを認可する`PostPolicy`を持つことになるでしょう。

`make:policy` [Artisanコマンド](/docs/{{version}}/artisan)を使用し、ポリシーを生成できます。生成したポリシーは`app/Policies`ディレクトリに設置されます。このディレクトリがアプリケーションに存在していなくても、Laravelにより作成されます。

    php artisan make:policy PostPolicy

`make:policy`コマンドは空のポリシークラスを生成します。基本的な「CRUD」ポリシーメソッドを生成するクラスへ含めたい場合は、`make:policy`コマンド実行時に`--model`を指定してください。

    php artisan make:policy PostPolicy --model=Post

> {tip} 全ポリシーはLaravelの [サービスコンテナ](/docs/{{version}}/container)により依存解決されるため、ポリシーのコンストラクタに必要な依存をタイプヒントすれば、自動的に注入されます。

<a name="registering-policies"></a>
### ポリシーの登録

ポリシーができたら、登録する必要があります。インストールしたLaravelアプリケーションに含まれている、`AuthServiceProvider`にはEloquentモデルと対応するポリシーをマップするための`policies`プロパティを含んでいます。ポリシーの登録とは、指定したモデルに対するアクションの認可時に、どのポリシーを利用するかをLaravelへ指定することです。

    <?php

    namespace App\Providers;

    use App\Post;
    use App\Policies\PostPolicy;
    use Illuminate\Support\Facades\Gate;
    use Illuminate\Foundation\Support\Providers\AuthServiceProvider as ServiceProvider;

    class AuthServiceProvider extends ServiceProvider
    {
        /**
         * アプリケーションにマップ付されたポリシー
         *
         * @var array
         */
        protected $policies = [
            Post::class => PostPolicy::class,
        ];

        /**
         * アプリケーションの全認証／認可サービスの登録
         *
         * @return void
         */
        public function boot()
        {
            $this->registerPolicies();

            //
        }
    }

<a name="writing-policies"></a>
## ポリシーの記述

<a name="policy-methods"></a>
### ポリシーのメソッド

ポリシーが登録できたら、認可するアクションごとにメソッドを追加します。たとえば、指定した`User`が指定`Post`インスタンスの更新をできるか決める、`updata`メソッドを`PostPolicy`に定義してみましょう。

`update`メソッドは`User`と`Post`インスタンスを引数で受け取り、ユーザーが指定`Post`の更新を行う認可を持っているかを示す、`true`か`false`を返します。ですから、この例の場合、ユーザーの`id`とポストの`user_id`が一致するかを確認しましょう。

    <?php

    namespace App\Policies;

    use App\User;
    use App\Post;

    class PostPolicy
    {
        /**
         * ユーザーにより指定されたポストが更新可能か決める
         *
         * @param  \App\User  $user
         * @param  \App\Post  $post
         * @return bool
         */
        public function update(User $user, Post $post)
        {
            return $user->id === $post->user_id;
        }
    }

必要に応じ、様々なアクションを認可するために、追加のメソッドをポリシーに定義してください。たとえば、色々な`Post`アクションを認可するために、`view`や`delete`メソッドを追加できます。ただし、ポリシーのメソッドには好きな名前を自由につけられることを覚えておいてください。

> {tip} ポリシーを`--model`オプションを付け、Artisanコマンドにより生成した場合、`view`、`create`、`update`、`delete`アクションが含まれています。

<a name="methods-without-models"></a>
### モデルを持たないメソッド

ポリシーメソッドの中には、現在の認証ユーザーのみを受け取り、認可するためのモデルを必要としないものもあります。この状況は、`create`アクションを認可する場合に、よく現れます。たとえば、ブログを作成する場合、どんなポストかにはかかわらず、そのユーザーが作成可能かを認可したいでしょう。

`create`のように、モデルインスタンスを受け取らないポリシーメソッドを定義する場合は、モデルインスタンスを受け取る必要はありません。代わりに、その認証済みユーザーが期待している人物かをメソッドで定義してください。

    /**
     * 指定されたユーザーがポストを作成できるかを決める
     *
     * @param  \App\User  $user
     * @return bool
     */
    public function create(User $user)
    {
        //
    }

<a name="policy-filters"></a>
### ポリシーフィルダー

特定のユーザーには指定したポリシーの全アクションを許可したい場合があります。そのためには、`before`メソッドをポリシーへ定義してください。`before`メソッドはポリシーの他のメソッドの前に実行されるため、意図するポリシーメソッドが実際に呼び出される前に、アクションを許可する機会を提供します。この機能は主に、アプリケーションの管理者に全てのアクションを実行する権限を与えるために使用されます。

    public function before($user, $ability)
    {
        if ($user->isSuperAdmin()) {
            return true;
        }
    }

ユーザーに対して全認可を禁止したい場合は、`before`メソッドから`false`を返します。`null`を返した場合、その認可の可否はポリシーメソッドにより決まります。

> {note} クラスがチェックするアビリティと一致する名前のメソッドを含んでいない場合、ポリシークラスの`before`メソッドは呼び出されません。

<a name="authorizing-actions-using-policies"></a>
## ポリシーを使ったアクションの認可

<a name="via-the-user-model"></a>
### Userモデルによる確認

Laravelアプリケーションに含まれる`User`モデルは、アクションを認可するための便利な２つのメソッドを持っています。`can`と`cant`です。`can`メソッドは認可したいアクションと関連するモデルを引数に取ります。例として、ユーザーが指定した`Post`を更新を認可するかを決めてみましょう。

    if ($user->can('update', $post)) {
        //
    }

指定するモデルの[ポリシーが登録済みであれば](#registering-policies)適切なポリシーの`can`メソッドが自動的に呼びだされ、論理型の結果が返されます。そのモデルに対するポリシーが登録されていない場合、`can`メソッドは指定したアクション名に合致する、ゲートベースのクロージャを呼びだそうとします。

#### モデルを必要としないアクション

`create`のようなアクションは、モデルインスタンスを必要としないことを思い出してください。そうした場合は、`can`メソッドにはクラス名を渡してください。クラス名はアクションを認可するときにどのポリシーを使用すべきかを決めるために使われます。

    use App\Post;

    if ($user->can('create', Post::class)) {
        // 関連するポリシーの"create"メソッドが実行される
    }

<a name="via-middleware"></a>
### ミドルウェアによる認可

送信されたリクエストがルートやコントローラへ到達する前に、アクションを認可できるミドルウェアをLaravelは持っています。デフォルトで`App\Http\Kernel`クラスの中で`can`キーに`Illuminate\Auth\Middleware\Authorize`ミドルウェアが割り付けられています。あるユーザーがブログポストを認可するために、`can`ミドルウェアを使う例をご覧ください。

    use App\Post;

    Route::put('/post/{post}', function (Post $post) {
        // 現在のユーザーはこのポストを更新できる
    })->middleware('can:update,post');

この例では、`can`ミドルウェアへ２つの引数を渡しています。最初の引数は認可したいアクションの名前です。２つ目はポリシーメソッドに渡したいルートパラメータです。この場合、[暗黙のモデル結合](/docs/{{version}}/routing#implicit-binding)を使用しているため、`Post`モデルがポリシーメソッドへ渡されます。ユーザーに指定したアクションを実行する認可がない場合、ミドルウェアは`403`ステータスコードのHTTPレスポンスを生成します。

#### モデルを必要としないアクション

この場合も、`create`のようなアクションではモデルインスタンスを必要としません。このようなケースでは、ミドルウェアへクラス名を渡してください。クラス名はアクションを認可するときに、どのポリシーを使用するかの判断に使われます。

    Route::post('/post', function () {
        // 現在のユーザーはポストを更新できる
    })->middleware('can:create,App\Post');

<a name="via-controller-helpers"></a>
### コントローラヘルパによる認可

`User`モデルが提供している便利なメソッドに付け加え、`App\Http\Controllers\Controller`ベースクラスを拡張しているコントローラに対し、Laravelは`authorize`メソッドを提供しています。`can`メソッドと同様に、このメソッドは認可対象のアクション名と関連するモデルを引数に取ります。アクションが認可されない場合、`authorize`メソッドは`Illuminate\Auth\Access\AuthorizationException`例外を投げ、これはデフォルトでLaravelの例外ハンドラにより、`403`ステータスコードのHTTPレスポンスへ変換されます。

    <?php

    namespace App\Http\Controllers;

    use App\Post;
    use Illuminate\Http\Request;
    use App\Http\Controllers\Controller;

    class PostController extends Controller
    {
        /**
         * 指定したポストの更新
         *
         * @param  Request  $request
         * @param  Post  $post
         * @return Response
         */
        public function update(Request $request, Post $post)
        {
            $this->authorize('update', $post);

            // 現在のユーザーはブログポストの更新が可能
        }
    }

#### モデルを必要としないアクション

既に説明してきたように、`create`のように、モデルインスタンスを必要としないアクションがあります。この場合、クラス名を`authorize`メソッドへ渡してください。クラス名はアクションの認可時に、どのポリシーを使用するのかを決めるために使われます。

    /**
     * 新しいブログポストの生成
     *
     * @param  Request  $request
     * @return Response
     */
    public function create(Request $request)
    {
        $this->authorize('create', Post::class);

        // 現在のユーザーはブログポストを生成できる
    }

<a name="via-blade-templates"></a>
### Bladeテンプレートによる認可

Bladeテンプレートを書くとき、指定したアクションを実行できる認可があるユーザーの場合のみ、ページの一部分を表示したい場合があります。たとえば、実際にポストを更新できるユーザーの場合のみ、ブログポストの更新フォームを表示したい場合です。この場合、`@can`と`@cannot`系ディレクティブを使います。

    @can('update', $post)
        <!-- 現在のユーザーはポストを更新できる -->
    @elsecan('create', App\Post::class)
        <!-- 現在のユーザーはポストを作成できる -->
    @endcan

    @cannot('update', $post)
        <!-- 現在のユーザーはポストを更新できない -->
    @elsecannot('create', App\Post::class)
        <!-- 現在のユーザーはポストを作成できない -->
    @endcannot

これらのディレクティブは`@if`や`@unless`文を使う記述に対する、便利な短縮形です。上記の`@can`と`@cannot`文に対応するコードは以下のようになります。

    @if (Auth::user()->can('update', $post))
        <!-- 現在のユーザーはポストを更新できる -->
    @endif

    @unless (Auth::user()->can('update', $post))
        <!-- 現在のユーザーはポストを更新できない -->
    @endunless

#### モデルを必要としないアクション

他の認可メソッドと同様に、アクションがモデルインスタンスを必要としない場合、`@can`と`@cannot`ディレクティブへ、クラス名を渡すことができます。

    @can('create', App\Post::class)
        <!-- 現在のユーザーはポストを更新できる -->
    @endcan

    @cannot('create', App\Post::class)
        <!-- 現在のユーザーはポストを更新できない -->
    @endcannot
