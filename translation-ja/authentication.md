# 認証

- [イントロダクション](#introduction)
    - [データベースの検討事項](#introduction-database-considerations)
- [認証クイックスタート](#authentication-quickstart)
    - [ルーティング](#included-routing)
    - [ビュー](#included-views)
    - [認証](#included-authenticating)
    - [認証済みユーザーの取得](#retrieving-the-authenticated-user)
    - [ルートの保護](#protecting-routes)
    - [認証回数制限](#login-throttling)
- [手動のユーザー認証](#authenticating-users)
    - [継続ログイン](#remembering-users)
    - [その他の認証方法](#other-authentication-methods)
- [HTTP基本認証](#http-basic-authentication)
    - [ステートレスHTTP基本認証](#stateless-http-basic-authentication)
- [ソーシャル認証](https://github.com/laravel/socialite)
- [カスタムガードの追加](#adding-custom-guards)
- [カスタムユーザープロバイダの追加](#adding-custom-user-providers)
    - [ユーザープロバイダ契約](#the-user-provider-contract)
    - [Authenticatable契約](#the-authenticatable-contract)
- [イベント](#events)

<a name="introduction"></a>
## イントロダクション

> {tip} **さくっと始めたいですか？** インストールしたてのLaravelアプリケーションにて、`php artisan make:auth`と`php artisan migrate`を実行するだけです。それから、ブラウザで`http://your-app.dev/register`、もしくはアプリケーションに割りつけたその他のURLへアクセスしてください。この２つのコマンドが、認証システム全体のスカフォールドを面倒みます。

Laravelでは簡単に認証が実装できます。実際ほとんど全て最初から設定済みです。認証の設定ファイルは`config/auth.php`に用意してあり、認証サービスの振る舞いを調整できるように、読みやすいコメント付きでたくさんのオプションが用意されています。

Laravelの認証機能は「ガード」と「プロバイダ」を中心概念として構成されています。ガードは各リクエストごとに、どのようにユーザーを認証するかを定義します。たとえば、Laravelにはセッションストレージとクッキーを使いながら状態を維持する`session`ガードが用意されています。

プロバイダは永続ストレージから、どのようにユーザーを取得するかを定義します。LaravelはEloquentとデータベースクリエビルダを使用しユーザーを取得する機能を用意しています。しかし、アプリケーションの必要性に応じて、自由にプロバイダを追加できます。

混乱しても心配ありません。通常のアプリケーションでは、デフォルトの認証設定を変更する必要はありません。

<a name="introduction-database-considerations"></a>
### データベースの検討事項

デフォルトとしてLaravelは、`App\User` [Eloquentモデル](/docs/{{version}}/eloquent)を`app`ディレクトリに用意しています。このモデルはデフォルトEloquent認証ドライバで使用しています。もしアプリケーションでEloquentを使用しなければ、Laravelクエリビルダを使用する`database`認証ドライバを使用する必要があります。

`App\User`モデルのデータベーススキマー構築時に、確実にパスワードカラムを最低６０文字確保してください。デフォルトの２５５文字にするのが、良い方法でしょう。

さらに`users`、もしくは同等の働きをするテーブルには、１００文字の`remember_token`文字列カラムも含めてください。このカラムはログイン時に、アプリケーションで"remember me"を選んだユーザーのトークンを保存しておくカラムとして使用されます。

<a name="authentication-quickstart"></a>
## 認証クイックスタート

`App\Http\Controllers\Auth`名前空間下に多くの組み込み済み認証コントローラがLaravelにより用意されています。`RegisterController`は新ユーザーの登録、`LoginController`は認証処理、`ForgotPasswordController`はパスワードリセットのためのメールリンク処理、`ResetPasswordController`はパスワードリセット処理を行います。各コントローラは必要なメソッドを含むトレイトを使用しています。多くのアプリケーションでは、これらのコントローラを変更する必要は全くありません。

<a name="included-routing"></a>
### ルート定義

簡単なコマンド一つで、認証に必要となるルート定義とビューをすべてスカフォールディングできる、簡単な手段をLaravelは提供しています。

    php artisan make:auth

このコマンドは新しくインストールしたアプリケーションでのみ実行すべきで、レイアウトビュー、登録ログインビューをインストールし、同時にすべての認証エンドポイントのルートも定義します。`HomeController`も、ログイン後に必要となる、アプリケーションのダッシュボードのために生成されます。

<a name="included-views"></a>
### ビュー

前のセクションで説明したとおり、`php artisan make:auth`コマンドは`resources/views/auth`ディレクトリへ、認証に必要なすべてのビューを生成します。

`make:auth`コマンドは`resources/views/layouts`ディレクトリにアプリケーションのベースレイアウトビューも生成します。これらのビューはすべてBootstrap CSSフレームワークを使用していますが、自由にカスタマイズしてください。

<a name="included-authenticating"></a>
### 認証

これで、認証コントローラを含め、必要なルートとビューの準備が整いました。アプリケーションに新しいユーザーを登録し、認証できるようになりました。ブラウザでアプリケーションへアクセスしてください。認証コントローラは（内部で使用しているトレイトにより）、既存ユーザーの認証と、新しいユーザーをデータベースへ保存するロジックをすでに備えています。

#### パスのカスタマイズ

ユーザーが認証に成功すると、`/home`のURIへリダイレクトします。これをカスタマイズするには、`LoginController`、`RegisterController`、`ResetPasswordController`の`redirectTo`プロパティで、認証後のリダイレクト先の場所を定義してください。

    protected $redirectTo = '/';

リダイレクトパスをカスタマイズするロジックが必要な場合は、`redirectTo`プロパティの代わりに`redirectTo`メソッドを定義してください。

    protected function redirectTo()
    {
        return '/path';
    }

> {tip} `redirectTo`メソッドは、`redirectTo`プロパティよりも優先されます。

#### ユーザー名のカスタマイズ

デフォルトでLaravelは`email`フィールドを認証に利用します。これをカスタマイズしたい場合は、`LoginController`で`username`メソッドを定義してください。

    public function username()
    {
        return 'username';
    }

#### ガードのカスタマイズ

更に、登録済みユーザーを認証するために使用する「ガード」をカスタマイズすることも可能です。`LoginController`、 `RegisterController`、`ResetPasswordController`で`guard`メソッドを定義してください。メソッドからガードインスタンスを返してください。

    use Illuminate\Support\Facades\Auth;

    protected function guard()
    {
        return Auth::guard('guard-name');
    }

#### バリデーション／保管域のカスタマイズ

アプリケーションに新しいユーザーを登録する場合に入力してもらうフォームのフィールドを変更するか、データベースに新しいユーザーレコードを登録する方法をカスタマイズしたい場合は、`RegisterController`クラスを変更してください。このクラスはアプリケーションで新しいユーザーのバリデーションと作成に責任を持っています。

`RegisterController`の`validator`メソッドはアプリケーションの新しいユーザーに対するバリデーションルールで構成されています。このメソッドはお気に召すまま自由に変更してください。

`RegisterController`の`create`メソッドは新しい`App\User`レコードを[Eloquent ORM](/docs/{{version}}/eloquent)を使用し、データベースに作成することに責任を持っています。データベースの必要に合わせて自由にこのメソッドを変更してください。

<a name="retrieving-the-authenticated-user"></a>
### 認証済みユーザーの取得

`Auth`ファサードを使えば認証済みユーザーへ簡単にアクセスできます。

    use Illuminate\Support\Facades\Auth;

    // 現在認証されているユーザーの取得
    $user = Auth::user();

    // 現在認証されているユーザーのID取得
    $id = Auth::id();

もしくは、ユーザーが認証済みであれば、`Illuminate\Http\Request`インスタンス経由で、ユーザーへアクセスすることもできます。コントローラメソッドでタイプヒントしたクラスは、自動的にインスタンスが依存注入されることを思い出してください。

    <?php

    namespace App\Http\Controllers;

    use Illuminate\Http\Request;

    class ProfileController extends Controller
    {
        /**
         * ユーザープロフィールの更新
         *
         * @param  Request  $request
         * @return Response
         */
        public function update(Request $request)
        {
            // $request->user()は認証済みユーザーのインスタンスを返す
        }
    }

#### 現在のユーザーが認証されているか調べる

ユーザーが既にアプリケーションにログインしているかを調べるには、`Auth`ファサードの`check`メソッドが使えます。認証時に`true`を返します。

    use Illuminate\Support\Facades\Auth;

    if (Auth::check()) {
        // ユーザーはログインしている
    }

> {tip} `check`メソッドを使っても、あるユーザーが認証済みであるかを判定可能です。特定のルートやコントローラへユーザーをアクセスさせる前に、認証済みであるかをミドルウェアにより確認する場合、典型的に使用します。より詳細については[ルートの保護](/docs/{{version}}/authentication#protecting-routes)のドキュメントを参照してください。

<a name="protecting-routes"></a>
### ルートの保護

[ルートミドルウェア](/docs/{{version}}/middleware)は特定のルートに対し、認証済みユーザーのみアクセスを許すために使われます。Laravelには`Illuminate\Auth\Middleware\Authenticate`の中で定義されている`auth`ミドルウェアが最初から用意されています。このミドルウェアは、HTTPカーネルで登録済みのため、必要なのはルート定義でこのミドルウェアを指定するだけです。

    Route::get('profile', function() {
        // 認証済みのユーザーのみが入れる
    })->middleware('auth');

もちろん[コントローラ](/docs/{{version}}/controllers)を使っていれば、ルート定義に付加する代わりに、コントローラのコンストラクターで`middleware`メソッドを呼び出すことができます。

    public function __construct()
    {
        $this->middleware('auth');
    }

#### ガードの指定

`auth`ミドルウェアをルートに対し指定するときに、そのユーザーに対し認証を実行するガードを指定することもできます。指定されたガードは、`auth.php`設定ファイルの`guards`配列のキーを指定します。

    public function __construct()
    {
        $this->middleware('auth:api');
    }

<a name="login-throttling"></a>
### 認証回数制限

Laravelの組み込み`LoginController`クラスを使用している場合、`Illuminate\Foundation\Auth\ThrottlesLogins`トレイトが最初からコントローラで取り込まれています。デフォルトでは何度も正しくログインできなかった後、一分間ログインできなくなります。制限はユーザーの名前／メールアドレスとIPアドレスで限定されます。

<a name="authenticating-users"></a>
## 自前のユーザー認証

もちろん、Laravelに含まれる認証コントローラを使うことを強要しているわけでありません。これらのコントローラを削除する選択肢を選ぶのなら、Laravel認証クラスを直接使用しユーザーの認証を管理する必要があります。心配ありません。それでも簡単です！

Laravelの認証サービスには`Auth`[ファサード](/docs/{{version}}/facades)でアクセスできます。クラスの最初で`Auth`ファサードを確実にインポートしておきましょう。次に`attempt`メソッドを見てみましょう。

    <?php

    namespace App\Http\Controllers;

    use Illuminate\Support\Facades\Auth;

    class LoginController extends Controller
    {
        /**
         * 認証を処理する
         *
         * @return Response
         */
        public function authenticate()
        {
            if (Auth::attempt(['email' => $email, 'password' => $password])) {
                // 認証に成功した
                return redirect()->intended('dashboard');
            }
        }
    }

`attempt`メソッドは最初の引数として、キー／値ペアの配列を受け取ります。配列中の他の値は、データベーステーブルの中からそのユーザーを見つけるために使用されます。そのため、上の例では`email`カラム値によりユーザーが取得されます。データベース中のハッシュ済みパスワードと比較する前に、フレームワークは自動的に値をハッシュ化するため、`password`へ指定する値はハッシュ化しないでください。２つのハッシュ済みパスワードが一致したら、そのユーザーの新しい認証セッションが開始されます。

`attempt`メソッドは、認証が成功すれば`true`を返します。失敗時は`false`を返します。

リダイレクタ―の`intended`メソッドは、認証フィルターにかかる前にアクセスしようとしていたURLへ、ユーザーをリダイレクトしてくれます。そのリダイレクトが不可能な場合の移動先として、フォールバックURIをこのメソッドに指定してください。

#### 追加条件の指定

お望みであれば、ユーザーのメールアドレスとパスワードに付け加え、認証時のクエリに追加の条件を指定することも可能です。例として、ユーザーが「アクティブ」である条件を追加してみましょう。

    if (Auth::attempt(['email' => $email, 'password' => $password, 'active' => 1])) {
        // ユーザーは存在しており、かつアクティブで、資格停止されていない
    }

> {note} この例のように、`email`を必ず認証に使用しなくてならない訳ではありません。データーベース中にあるユーザー名(username)に該当する、一意にユーザーを特定できるカラムであれば何でも使用できます。

#### 特定のGuardインスタンスへのアクセス

`Auth`ファサードの`guard`メソッドにより、使用したいガードインスタンスを指定できます。これにより全く異なった認証用のモデルやユーザーテーブルを使い、アプリケーションの別々の部分に対する認証を管理することができます。

`guard`メソッドへ渡すガード名は、`auth.php`認証ファイルのguards設定の一つと対応している必要があります。

    if (Auth::guard('admin')->attempt($credentials)) {
        //
    }

#### ログアウト

アプリケーションからユーザーをログアウトさせるには、`Auth`ファサードの`logout`メソッドを使用してください。これはユーザーセッションの認証情報をクリアします。

    Auth::logout();

<a name="remembering-users"></a>
### 継続ログイン

アプリケーションでログイン維持(Remember me)機能を持たせたい場合は、`attempt`メソッドの第２引数に論理値を指定します。ユーザーが自分でログアウトしない限り、認証が無期限に持続するようになります。もちろん、"remember me"トークンを保存するために使用する文字列の`remember_token`カラムを`users`テーブルに持たせる必要があります。

    if (Auth::attempt(['email' => $email, 'password' => $password], $remember)) {
        // このメンバーは継続ログインされる
    }

> {tip} Laravelに用意されている、組み込み`LoginController`を使用する場合、このコントローラが使用しているトレイトにより、"remember"ユーザーを確実に処理するロジックが実装済みです。

この機能を使用している時に、ユーザーが"remember me"クッキーを使用して認証されているかを判定するには、`viaRemember`メソッドを使用します。

    if (Auth::viaRemember()) {
        //
    }

<a name="other-authentication-methods"></a>
### 他の認証方法

#### Userインスタンスによる認証

既に存在しているユーザーインスタンスでアプリケーションにログインさせる必要があれば、`login`メソッドにそのユーザーインスタンスを指定し呼び出してください。指定されたオブジェクトは`Illuminate\Contracts\Auth\Authenticatable`[契約](/docs/{{version}}/contracts)を実装している必要があります。もちろん、Laravelが用意している`App\User`モデルはこのインターフェイスを実装しています。

    Auth::login($user);

    // 指定したユーザーでログインし、"remember"にする
    Auth::login($user, true);

もちろん、使用したいガードインスタンスを指定することもできます。

    Auth::guard('admin')->login($user);

#### IDによるユーザー認証

ユーザーをアプリケーションへIDによりログインさせる場合は、`loginUsingId`メソッドを使います。このメソッドは認証させたいユーザーの主キーを引数に受け取ります。

    Auth::loginUsingId(1);

    // 指定したユーザーでログインし、"remember"にする
    Auth::loginUsingId(1, true);

#### ユーザーを一度だけ認証する

`once`メソッドを使用すると、アプリケーションにユーザーをそのリクエストの間だけログインさせることができます。セッションもクッキーも使用しないため、ステートレスなAPIを構築する場合に便利です。

    if (Auth::once($credentials)) {
        //
    }

<a name="http-basic-authentication"></a>
## HTTP基本認証

[HTTP基本認証](https://en.wikipedia.org/wiki/Basic_access_authentication)により、専用の「ログイン」ページを用意しなくても手っ取り早くアプリケーションにユーザーをログインさせられます。これを使用するには、ルートに`auth.basic`[ミドルウェア](/docs/{{version}}/middleware)を付けてください。`auth.basic`ミドルウェアはLaravelフレームワークに含まれているので、定義する必要はありません。

    Route::get('profile', function() {
        // 認証済みのユーザーのみが入れる
    })->middleware('auth.basic');

ミドルウェアをルートに指定すれば、ブラウザーからこのルートへアクセスされると自動的に認証が求められます。デフォルトでは、`auth.basic`ミドルウェアはユーザーを決める"username"としてユーザーの`email`カラムを使用します。

#### FastCGIの注意

PHP FastCGIを使用している場合、初期状態のままでHTTP基本認証は正しく動作しないでしょう。以下の行を`.htaccess`ファイルへ追加してください。

    RewriteCond %{HTTP:Authorization} ^(.+)$
    RewriteRule .* - [E=HTTP_AUTHORIZATION:%{HTTP:Authorization}]

<a name="stateless-http-basic-authentication"></a>
### ステートレスなHTTP基本認証

セッションの識別クッキーを用いずにHTTP基本認証を使用することもできます。これは特にAPI認証に便利です。実装するには、`onceBasic`メソッドを呼び出す[ミドルウェアを定義](/docs/{{version}}/middleware)してください。`onceBasic`メソッドが何もレスポンスを返さなかった場合、リクエストをアプリケーションの先の処理へ通します。

    <?php

    namespace App\Http\Middleware;

    use Illuminate\Support\Facades\Auth;

    class AuthenticateOnceWithBasicAuth
    {
        /**
         * 送信されたリクエストの処理
         *
         * @param  \Illuminate\Http\Request  $request
         * @param  \Closure  $next
         * @return mixed
         */
        public function handle($request, $next)
        {
            return Auth::onceBasic() ?: $next($request);
        }

    }

次に[ルートミドルウェアを登録](/docs/{{version}}/middleware#registering-middleware)し、ルートに付加します。

    Route::get('api/user', function() {
        // 認証済みのユーザーのみが入れる
    })->middleware('auth.basic.once');

<a name="adding-custom-guards"></a>
## カスタムガードの追加

独自の認証ガードは`Auth`ファサードの`extend`メソッドを使用し、定義します。[サービスプロバイダ](/docs/{{version}}/providers)の中で呼び出します。`AuthServiceProvider`をLaravelはあらかじめ用意しているので、この中にコードを設置できます。

    <?php

    namespace App\Providers;

    use App\Services\Auth\JwtGuard;
    use Illuminate\Support\Facades\Auth;
    use Illuminate\Foundation\Support\Providers\AuthServiceProvider as ServiceProvider;

    class AuthServiceProvider extends ServiceProvider
    {
        /**
         * サービスの初期起動後の登録実行
         *
         * @return void
         */
        public function boot()
        {
            $this->registerPolicies();

            Auth::extend('jwt', function($app, $name, array $config) {
                // Illuminate\Contracts\Auth\Guardのインスタンスを返す

                return new JwtGuard(Auth::createUserProvider($config['provider']));
            });
        }
    }

上記の例のように、コールバックを`extend`メソッドに渡し、`Illuminate\Contracts\Auth\Guard`の実装を返します。このインスタンスは、カスタムガードで定義が必要ないくつかのメソッドを持っています。カスタムガードを定義したら、`auth.php`設定ファイルの、`guards`設定で使用できます。

    'guards' => [
        'api' => [
            'driver' => 'jwt',
            'provider' => 'users',
        ],
    ],

<a name="adding-custom-user-providers"></a>
## カスタムユーザープロバイダの追加

ユーザー情報を保管するために伝統的なリレーショナルデータベースを使用したくなければ、Laravelに独自の認証ユーザープロバイダを拡張する必要があります。`Auth`ファサードの`provider`メソッドを使い、カスタムユーザープロバイダを定義します。

    <?php

    namespace App\Providers;

    use Illuminate\Support\Facades\Auth;
    use App\Extensions\RiakUserProvider;
    use Illuminate\Foundation\Support\Providers\AuthServiceProvider as ServiceProvider;

    class AuthServiceProvider extends ServiceProvider
    {
        /**
         * サービスの初期起動後の登録実行
         *
         * @return void
         */
        public function boot()
        {
            $this->registerPolicies();

            Auth::provider('riak', function($app, array $config) {
                // Illuminate\Contracts\Auth\UserProviderのインスタンスを返す

                return new RiakUserProvider($app->make('riak.connection'));
            });
        }
    }

`provider`メソッドでプロバイダを登録したら、`auth.php`設定ファイルで新しいユーザープロバイダへ切り替えます。最初に、新しいドライバを使用する`provider`を定義します。

    'providers' => [
        'users' => [
            'driver' => 'riak',
        ],
    ],

次に、このプロバイダを`guards`設定項目で利用します。

    'guards' => [
        'web' => [
            'driver' => 'session',
            'provider' => 'users',
        ],
    ],

<a name="the-user-provider-contract"></a>
### ユーザープロバイダ契約

`Illuminate\Contracts\Auth\UserProvider`は、MySQLやRiakなどのような持続性のストレージシステムに対する`Illuminate\Contracts\Auth\Authenticatable`の実装を取得することだけに責任を持っています。これらの２つのインターフェイスはユーザーデータがどのように保存されているか、それを表すのがどんなタイプのクラスなのかに関わらず、認証メカニズムを機能し続けるために役立っています。

`Illuminate\Contracts\Auth\UserProvider`契約を見てみましょう。

    <?php

    namespace Illuminate\Contracts\Auth;

    interface UserProvider {

        public function retrieveById($identifier);
        public function retrieveByToken($identifier, $token);
        public function updateRememberToken(Authenticatable $user, $token);
        public function retrieveByCredentials(array $credentials);
        public function validateCredentials(Authenticatable $user, array $credentials);

    }

`retrieveById`関数は通常MySQLデータベースの自動増分IDのようなユーザーを表すキーを受け付けます。IDにマッチする`Authenticatable`実装が取得され、返されます。

`retrieveByToken`関数は一意の`$identifier`と`remember_token`フィールドに保存されている"remember me" `$token`からユーザーを取得します。前のメソッドと同じく、`Authenticatable`実装が返されます。

`updateRememberToken`メソッドは`$user`の`remember_token`フィールドを新しい`$token`で更新します。新しいトークンは真新しいものでも、「Remember me」ログインに成功した時の値でも、ログアウト時の値でも受け付けます。

`retrieveByCredentials`メソッドはアプリケーションへログイン時に`Auth::attempt`メソッドに指定するのと同じく、ユーザー認証情報の配列を引数に取ります。メソッドは認証情報に一致するユーザーを裏の持続ストレージから「クエリ」する必要があります。典型的な使用方法の場合、このメソッドは`$credentials['username']`の"where"条件でクエリを実行するでしょう。メソッドは`Authenticatable`の実装を返します。**このメソッドはパスワードバリデーションや認証を行ってはいけません**

`validateCredentials`メソッドは指定された`$user`とユーザーを認証するための`$credentials`とを比較します。たとえばこのメソッドは`$user->getAuthPassword()`の値と`$credentials['password']`を`Hash::make`で値を比較します。このメソッドはパスワードが有効であるかを示す、`true`か`false`だけを返します。

<a name="the-authenticatable-contract"></a>
### Authenticatable契約

これで`UserProvider`の各メソッドが明らかになりました。続いて`Authenticatable`契約を見てみましょう。プロバイダは`retrieveById`と`retrieveByCredentials`メソッドでこのインターフェイスの実装を返していたことを思い出してください。

    <?php

    namespace Illuminate\Contracts\Auth;

    interface Authenticatable {

        public function getAuthIdentifierName();
        public function getAuthIdentifier();
        public function getAuthPassword();
        public function getRememberToken();
        public function setRememberToken($value);
        public function getRememberTokenName();

    }

このインターフェイスはシンプルです。`getAuthIdentifierName`メソッドは、ユーザーの「主キー」フィールドの名前を返します。`getAuthIdentifier`メソッドはユーザーの主キーを返します。MySQLを裏で使用している場合、これは自動増分される主キーでしょう。`getAuthPassword`はユーザーのハッシュ済みのパスワードを返します。このインターフェイスはどのORMや抽象ストレージ層を使用しているかに関わらず、どんなUserクラスに対しても認証システムが動作するようにしてくれています。デフォルトでLaravelは`app`ディレクトリ中に、このインターフェイスを実装してる`User`クラスを持っています。ですから実装例として、このクラスを調べてみてください。

<a name="events"></a>
## イベント

Laravelは認証処理の過程で、様々な[イベント](/docs/{{version}}/events)を発行します。`EventServiceProvider`の中で、こうしたイベントに対するリスナを設定できます。:

    /**
     * アプリケーションに指定されたイベントリスナ
     *
     * @var array
     */
    protected $listen = [
        'Illuminate\Auth\Events\Registered' => [
            'App\Listeners\LogRegisteredUser',
        ],

        'Illuminate\Auth\Events\Attempting' => [
            'App\Listeners\LogAuthenticationAttempt',
        ],

        'Illuminate\Auth\Events\Registered' => [
            'App\Listeners\LogRegisteredUser',
        ],

        'Illuminate\Auth\Events\Login' => [
            'App\Listeners\LogSuccessfulLogin',
        ],

        'Illuminate\Auth\Events\Failed' => [
            'App\Listeners\LogFailedLogin',
        ],

        'Illuminate\Auth\Events\Logout' => [
            'App\Listeners\LogSuccessfulLogout',
        ],

        'Illuminate\Auth\Events\Lockout' => [
            'App\Listeners\LogLockout',
        ],

        'Illuminate\Auth\Events\PasswordReset' => [
            'App\Listeners\LogPasswordReset',
        ],
    ];
