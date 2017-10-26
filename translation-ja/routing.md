# ルーティング

- [基本的なルーティング](#basic-routing)
    - [リダイレクトルート](#redirect-routes)
    - [ビュールート](#view-routes)
- [ルートパラメーター](#route-parameters)
    - [必須パラメータ](#required-parameters)
    - [任意パラメータ](#parameters-optional-parameters)
    - [正規表現制約](#parameters-regular-expression-constraints)
- [名前付きルート](#named-routes)
- [ルートグループ](#route-groups)
    - [ミドルウェア](#route-group-middleware)
    - [名前区間](#route-group-namespaces)
    - [サブドメインルーティング](#route-group-sub-domain-routing)
    - [ルートプレフィックス](#route-group-prefixes)
- [モデル結合ルート](#route-model-binding)
    - [暗黙の結合](#implicit-binding)
    - [明示的な結合](#explicit-binding)
- [擬似フォームメソッド](#form-method-spoofing)
- [現在のルートへのアクセス](#accessing-the-current-route)

<a name="basic-routing"></a>
## 基本的なルーティング

一番基本のLaravelルートはシンプルにURIと「クロージャ」により定義され、単純で記述しやすいルートの定義方法を提供しています。

    Route::get('foo', function () {
        return 'Hello World';
    });

#### デフォルトルート定義ファイル

Laravelの全ルートは、`routes`ディレクトリ下に設置されている、ルートファイルで定義されます。これらのファイルはフレームワークにより、自動的に読み込まれます。`routes/web.php`ファイルで、Webインターフェイスのルートを定義します。定義されたルートは`web`ミドルウェアグループにアサインされ、セッション状態やCSRF保護などの機能が提供されます。`routes/api.php`中のルートはステートレスで、`api`ミドルウェアグループにアサインされます。

ほとんどのアプリケーションでは、`routes/web.php`ファイルからルート定義を始めます。`routes/web.php`中で定義されたルートは、ブラウザで定義したルートのURLを入力することでアクセスします。たとえば、次のルートはブラウザから`http://your-app.dev/user`でアクセスします。

    Route::get('/user', 'UsersController@index');

`routes/api.php`ファイル中で定義したルートは`RouteServiceProvider`により、ルートグループの中にネストされます。このグループには、`/api`のURIが自動的にプレフィックスされ、それによりこのファイル中の全ルートにわざわざ指定する必要はありません。プレフィックスや他のルートグループオプションに変更する場合は、`RouteServiceProvider`を変更してください。

#### 使用可能なルート定義メソッド

ルータはHTTP動詞に対応してルートを定義できるようにしています。

    Route::get($uri, $callback);
    Route::post($uri, $callback);
    Route::put($uri, $callback);
    Route::patch($uri, $callback);
    Route::delete($uri, $callback);
    Route::options($uri, $callback);

複数のHTTP動詞に対応したルートを登録する必要が起きることもあります。`match`メソッドが利用できます。もしくは全HTTP動詞に対応する`any`メソッドを使い、ルート登録することもできます。

    Route::match(['get', 'post'], '/', function () {
        //
    });

    Route::any('foo', function () {
        //
    });

#### CSRF保護

`web`ルートファイル中で定義され、`POST`、`PUT`、`DELETE`ルートへ送信されるHTMLフォームはすべて、CSRFトークンフィールドを含んでいる必要があります。含めていないと、そのリクエストは拒否されます。CSRF保護についての詳細は、[CSRFのドキュメント](/docs/{{version}}/csrf)をご覧ください。

    <form method="POST" action="/profile">
        {{ csrf_field() }}
        ...
    </form>

<a name="redirect-routes"></a>
### リダイレクトルート

他のURIへリダイレクトするルートを定義する場合は、`Route::redirect`メソッドを使用します。このメソッドは便利な短縮形を提供しているので、単純なリダイレクトを実行するために、完全なルートやコントローラを定義する必要はありません。

    Route::redirect('/here', '/there', 301);

<a name="view-routes"></a>
### ビュールート

ルートからビューを返すだけの場合は、`Route::view`メソッドを使用します。`redirect`メソッドと同様に、このメソッドはシンプルな短縮形を提供しており、完全なルートやコントローラを定義する必要はありません。`view`メソッドは、最初の引数にURIを取り、ビュー名は第２引数です。更に、オプションの第３引数として、ビューへ渡すデータの配列を指定することもできます。

    Route::view('/welcome', 'welcome');

    Route::view('/welcome', 'welcome', ['name' => 'Taylor']);

<a name="route-parameters"></a>
## ルートパラメーター

<a name="required-parameters"></a>
### 必須パラメータ

もちろん、ルートの中のURIセグメントを取り出す必要が起きることもあります。たとえば、URLからユーザーIDを取り出したい場合です。ルートパラメーターを定義してください。

    Route::get('user/{id}', function ($id) {
        return 'User '.$id;
    });

ルートで必要なだけのルートパラメーターを定義することができます。

    Route::get('posts/{post}/comments/{comment}', function ($postId, $commentId) {
        //
    });

ルートパラメータは、いつも`{}`括弧で囲み、アルファベット文字で構成してください。ルートパラメータには、ハイフン（`-`）を使えません。下線（`_`）を代わりに使用してください。ルートパラメータは、ルートコールバック／コントローラへ順番通りに注入されます。コールバック／コントローラ引数の名前は考慮されません。

<a name="parameters-optional-parameters"></a>
### 任意パラメータ

ルートパラメータを指定してもらう必要があるが、指定は任意にしたいこともよく起こります。パラメータ名の後に`?`を付けると、任意指定のパラメータになります。対応するルートの引数に、デフォルト値を必ず付けてください。

    Route::get('user/{name?}', function ($name = null) {
        return $name;
    });

    Route::get('user/{name?}', function ($name = 'John') {
        return $name;
    });

<a name="parameters-regular-expression-constraints"></a>
### 正規表現制約

ルートインスタンスの`where`メソッドを使用し、ルートパラメータのフォーマットを制約できます。`where`メソッドはパラメータ名と、そのパラメータがどのように制約を受けるのかを定義する正規表現を引数に取ります。

    Route::get('user/{name}', function ($name) {
        //
    })->where('name', '[A-Za-z]+');

    Route::get('user/{id}', function ($id) {
        //
    })->where('id', '[0-9]+');

    Route::get('user/{id}/{name}', function ($id, $name) {
        //
    })->where(['id' => '[0-9]+', 'name' => '[a-z]+']);

<a name="parameters-global-constraints"></a>
#### グローバル制約

指定した正規表現でいつもルートパラメータを制約したい場合は、`pattern`メソッドを使ってください。`RouteServiceProvider`の`boot`メソッドの中で、このようなパターンを定義します。

    /**
     * ルートモデル結合、パターンフィルタなどの定義
     *
     * @return void
     */
    public function boot()
    {
        Route::pattern('id', '[0-9]+');

        parent::boot();
    }

パターンを定義すると、パラメータ名を使用している全ルートで、自動的に提供されます。

    Route::get('user/{id}', function ($id) {
        // {id}が数値の場合のみ実行される
    });

<a name="named-routes"></a>
## 名前付きルート

名前付きルートは特定のルートへのURLを生成したり、リダイレクトしたりする場合に便利です。ルート定義に`name`メソッドをチェーンすることで、そのルートに名前がつけられます。

    Route::get('user/profile', function () {
        //
    })->name('profile');

コントローラアクションに対しても名前を付けることができます。

    Route::get('user/profile', 'UserController@showProfile')->name('profile');

#### 名前付きルートへのURLを生成する

ルートに一度名前を付ければ、その名前をグローバルな`route`関数で使用することで、URLを生成したり、リダイレクトしたりできます。

    // URLの生成
    $url = route('profile');

    // リダイレクトの生成
    return redirect()->route('profile');

そのルートでパラメーターを定義してある場合は、`route`関数の第２引数としてパラメーターを渡してください。指定されたパラメーターは自動的にURLの正しい場所へ埋め込まれます。

    Route::get('user/{id}/profile', function ($id) {
        //
    })->name('profile');

    $url = route('profile', ['id' => 1]);

#### 現在ルートの検査

現在のリクエストが指定した名前付きルートのものであるかを判定したい場合は、Routeインスタンスの`named`メソッドを使います。たとえば、ルートミドルウェアから、現在のルート名を判定できます。

    /**
     * 送信されたリクエストの処理
     *
     * @param  \Illuminate\Http\Request  $request
     * @param  \Closure  $next
     * @return mixed
     */
    public function handle($request, Closure $next)
    {
        if ($request->route()->named('profile')) {
            //
        }

        return $next($request);
    }

<a name="route-groups"></a>
## ルートグループ

ルートグループは多くのルートで共通なミドルウェアや名前空間のようなルート属性をルートごとに定義するのではなく、一括して適用するための手法です。`Route::group`メソッドの最初の引数には、共通の属性を配列で指定します。

<a name="route-group-middleware"></a>
### ミドルウェア

グループ中の全ルートにミドルウェアを指定するには、そのグループを定義する前に`middleware`メソッドを使用します。ミドルウェアは、配列に定義された順番で実行されます。

    Route::middleware(['first', 'second'])->group(function () {
        Route::get('/', function () {
            // firstとsecondミドルウェアを使用
        });

        Route::get('user/profile', function () {
            // firstとsecondミドルウェアを使用
        });
    });

<a name="route-group-namespaces"></a>
### 名前空間

ルートグループのもう一つのよくあるユースケースで、グループ内のコントローラに同じPHP名前空間を指定する場合は、`namespace`メソッドを使用します。

    Route::namespace('Admin')->group(function () {
        // "App\Http\Controllers\Admin"名前空間下のコントローラ
    });

`App\Http\Controllers`名前空間をコントローラルート登録時に毎回指定しなくても済むように、デフォルトで`RouteServiceProvider`が名前空間グループの中で`routes.php`ファイルを読み込み、指定していることを覚えておいてください。これにより、先頭の`App\Http\Controllers`名前空間を省略でき、続きの部分を指定するだけで済みます。

<a name="route-group-sub-domain-routing"></a>
### サブドメインルーティング

ルートグループはワイルドカードサブドメインをルート定義するためにも使えます。サブドメインの部分を取り出しルートやコントローラで使用するために、ルートURIにおけるルートパラメーターのように指定できます。サブドメインはグループを定義する前に、`domain`メソッドを呼び出し指定します。

    Route::domain('{account}.myapp.com')->group(function () {
        Route::get('user/{id}', function ($account, $id) {
            //
        });
    });

<a name="route-group-prefixes"></a>
### ルートプレフィックス

`prefix`メソッドはグループ内の各ルートに対して、指定されたURIのプレフィックスを指定するために使用します。たとえばグループ内の全ルートのURIに`admin`を付けたければ、次のように指定します。

    Route::prefix('admin')->group(function () {
        Route::get('users', function () {
            // Matches The "/admin/users" URL
        });
    });

<a name="route-model-binding"></a>
## モデル結合ルート

ルートかコントローラアクションへモデルIDが指定される場合、IDに対応するそのモデルを取得するため、大抵の場合クエリします。Laravelのモデル結合はルートへ直接、そのモデルインスタンスを自動的に注入する便利な手法を提供しています。つまり、ユーザーのIDが渡される代わりに、指定されたIDに一致する`User`モデルインスタンスが渡されます。

<a name="implicit-binding"></a>
### 暗黙の結合

Laravelはタイプヒントされた変数名とルートセグメント名が一致する場合、Laravelはルートかコントローラアクション中にEloquentモデルが定義されていると、自動的に依存解決します。

    Route::get('api/users/{user}', function (App\User $user) {
        return $user->email;
    });

`$user`変数が`App\User` Eloquentモデルとしてタイプヒントされており、変数名が`{user}` URIセグメントと一致しているため、Laravelは、リクエストされたURIの対応する値に一致するIDを持つ、モデルインスタンスを自動的に注入します。一致するモデルインスタンスがデータベースへ存在しない場合、404 HTTPレスポンスが自動的に生成されます。

#### キー名のカスタマイズ

指定されたモデルクラス取得時に、`id`以外のデータベースカラムをモデル結合で使用したい場合、Eloquentモデルの`getRouteKeyName`メソッドをオーバーライドしてください。

    /**
     * モデルのルートキーの取得
     *
     * @return string
     */
    public function getRouteKeyName()
    {
        return 'slug';
    }

<a name="explicit-binding"></a>
### 明示的な結合

明示的に結合を登録するには、ルータの`model`メソッドで、渡されるパラメータに対するクラスを指定します。`RouteServiceProvider`クラスの`boot`メソッドの中で明示的なモデル結合を定義してください。

    public function boot()
    {
        parent::boot();

        Route::model('user', App\User::class);
    }

次に`{user}`パラメーターを含むルートを定義します。

    Route::get('profile/{user}', function (App\User $user) {
        //
    });

`{user}`パラメーターを`App\User`モデルへ結合しているため、`User`インスタンスはルートへ注入されます。ですからたとえば、`profile/1`のリクエストでは、IDが`1`の`User`インスタンスが注入されます。

一致するモデルインスタンスがデータベース上に見つからない場合、404 HTTPレスポンスが自動的に生成されます。

#### 依存解決ロジックのカスタマイズ

独自の依存解決ロジックを使いたい場合は、`Route::bind`メソッドを使います。`bind`メソッドに渡す「クロージャ」は、URIセグメントの値を受け取るので、ルートへ注入したいクラスのインスタンスを返してください。

    public function boot()
    {
        parent::boot();

        Route::bind('user', function ($value) {
            return App\User::where('name', $value)->first() ?? abort(404);
        });
    }

<a name="form-method-spoofing"></a>
## 擬似フォームメソッド

HTLMフォームは`PUT`、`PATCH`、`DELETE`アクションをサポートしていません。ですから、HTMLフォームから呼ばれる`PUT`、`PATCH`、`DELETE`ルートを定義する時、フォームに`_method`隠しフィールドを追加する必要があります。`_method`フィールドとして送られた値は、HTTPリクエストメソッドとして使用されます。

    <form action="/foo/bar" method="POST">
        <input type="hidden" name="_method" value="PUT">
        <input type="hidden" name="_token" value="{{ csrf_token() }}">
    </form>

`_method`入力フィールドを生成するために、`method_field`ヘルパ関数を使用することもできます。

    {{ method_field('PUT') }}

<a name="accessing-the-current-route"></a>
## 現在のルートへのアクセス

送信されたリクエストを処理しているルートに関する情報へアクセスするには、`Route`ファサードへ`current`、`currentRouteName`、`currentRouteAction`メソッドを使用します。

    $route = Route::current();

    $name = Route::currentRouteName();

    $action = Route::currentRouteAction();

組み込まれている全メソッドを確認するには、[Routeファサードの裏で動作しているクラス](https://laravel.com/api/{{version}}/Illuminate/Routing/Router.html)と、[Routeインスタンス](https://laravel.com/api/{{version}}/Illuminate/Routing/Route.html)の２つについてのAPIドキュメントを参照してください。
