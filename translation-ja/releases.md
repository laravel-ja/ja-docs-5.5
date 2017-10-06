# リリースノート

- [バージョニング規約](#versioning-scheme)
- [サポートポリシー](#support-policy)
- [Laravel 5.5](#laravel-5.5)

<a name="versioning-scheme"></a>
## バージョニング規約

Laravelのバージョニングは、「パラダイム.メジャー・マイナー」の規約を維持しています。メジャーフレームワークリリースは、１月と６月の半年ごとにリリースします。一方、マイナーリリースは毎週のように、頻繁にリリースされます。マイナーリリースは、ブレーキングチェンジを**絶対に**含めません。

アプリケーションやパッケージで、Laravelフレームワークやコンポーネントを利用する場合、常に`5.5.*`のようにバージョンを指定してください。理由は上記の通り、Laravelのメジャーリリースは、ブレーキングチェンジを含んでいるからです。新しいメジャーリリースへの更新は、一日かからない程度になるように努力しています。

パラダイムシフトリリースは数年空けています。これはフレームワークの構造と規約に重要な変更が起きたことを表します。現在、パラダイムシフトリリースは開発されていません。

#### Laravelはなぜセマンティックバージョニングを使わないのか

Laravelのコンポーネント（Cashier、Dusk、Valet、Socialiteなど）では、セマンティックバージョニングが**使われて**います。しかし、Laravelフレームワーク自体には、使用していません。なぜなら、セマンティックバージョニングは、２つのコード部分がコンパチブルであるかを「還元論」的に決める方法だからです。たとえ、セマンティックバージョニングを採用していても、皆さんはアップグレードパッケージをインストール後、自動化したテストスーツを実行することにより、自分のコードベースでもコンパチブルであることを**実際に**確認する必要があります。

そのため、Laravelフレームワークでは、実際のリリース期間をより良く表すバージョニング方法を採用しています。マイナーリリースは**決して**内部のブレーキングチェンジを含まないわけですから、「パラダイム.メジャー.*」記法でバージョンを指定している限り、ブレーキングチェンジは発生しません。

<a name="support-policy"></a>
## サポートポリシー

Laravel5.5のようなLTSリリースでは、バグフィックスは２年間、セキュリティフィックスは３年間提供します。これらのリリースは長期間に渡るサポートとメンテナンスを提供します。 一般的なリリースでは、バグフィックスは６ヶ月、セキュリティフィックスは１年です。

<a name="laravel-5.5"></a>
## Laravel 5.5 (LTS)

Laravel5.5は、パッケージの自動検出やAPIリソース／変換、コンソールコマンドの自動登録、キュージョブのチェーン、キュージョブのレート制限、時間ベースのジョブ再試行、renderableなmailable、renderableかつreportableな例外、より一貫性のある例外処理、データベーステストの向上、簡単になったカスタムバリデーションルール、Reactのフロントエンドのプリセット、`Route::view`と`Route::redirect`メソッド、MemcachedとRedisキャッシュドライバの「ロック」、オンデマンド通知、DuskでのヘッドレスChromeのサポート、便利なBladeのショートカット記法、信用するプロキシサポートの向上などを付け加え、持続的な進化を遂げています。

同時に、美しいキューダッシュボードと設定システムを提供する、RedisベースのLaravelキューのための[Laravel Horizon](http://horizon.laravel.com)も新たにリリースしました。

> {tip} このドキュメントはフレームワークで注目してもらいたい機能向上についてまとめたものです。より全体的な変更ログは、いつでも[GitHub](https://github.com/laravel/framework/blob/5.5/CHANGELOG-5.5.md)で確認できます。

### Laravel Horizon

Horizonは、Laravelで動作するRedisキューのための、美しいダッシュボードとコード駆動による設定を提供します。Horizonにより、ジョブのスループット、実行時間、失敗したジョブのような、キューのメトリックを簡単に監視できます。

ワーカ設定はすべて一つのシンプルな設定ファイルにまとめられ、チーム全体でコラボ―レートできるソースコントロール下に置くことができます。

Horizonの詳細は、[完全なHorizonドキュメント](/docs/{{version}}/horizon)をご覧ください。

### パッケージディスカバリー

> {video} この機能の無料[動画チュートリアル](https://laracasts.com/series/whats-new-in-laravel-5-5/episodes/5)（英語）がLaracastsに用意されています。

以前のバージョンのLaravelではパッケージのインストールで、`app`設定ファイルへサービスプロバイダを追加したり、関連のあるファサードを登録したりと、多くの追加のステップが要求されるのが通常でした。しかし、Laravel5.5から、自動的に検出し、サービスプロバイダとファサードを登録します。

たとえば、この機能を試して見るために、人気の`barryvdh/laravel-debugbar`パッケージをLaravelアプリケーションにインストールしてみることができます。Composerによりパッケージがインストールされると、余計な設定をせずとも、アプリケーションにデバッグバーが利用できるようになっています。

    composer require barryvdh/laravel-debugbar

パッケージ開発者は、提供するパッケージの`composer.json`ファイルへ、サービスプロバイダとファサードを追加する必要があります。

    "extra": {
        "laravel": {
            "providers": [
                "Laravel\\Tinker\\TinkerServiceProvider"
            ]
        }
    },

サービスプロバイダとファサードをディスカバリーで使用するための変更の詳細は、[パッケージ開発](/docs/{{version}}/packages)のドキュメントで確認してください。

### APIリソース

APIを構築する場合、Eloquentモデルとアプリケーションのユーザーへ実際に返送するJSONレスポンスの間に、変換レイヤーが必要となります。Laravelのリソースクラスで、モデルとモデルコレクションをJSONへ、記述的かつ簡単に変換できます。

リソースクラスはJSON構造へ変換する必要のある、単一モデルを表します。例として、簡単な`User`リソースクラスをご覧ください。

    <?php

    namespace App\Http\Resources;

    use Illuminate\Http\Resources\Json\Resource;

    class User extends Resource
    {
        /**
         * リソースを配列へ変換
         *
         * @param  \Illuminate\Http\Request
         * @return array
         */
        public function toArray($request)
        {
            return [
                'id' => $this->id,
                'name' => $this->name,
                'email' => $this->email,
                'created_at' => $this->created_at,
                'updated_at' => $this->updated_at,
            ];
        }
    }

もちろん、これは一番基本的なAPIリソースです。Laravelは、リソースとリソースコレクションを構築する時に役立つ、バラエティ豊かなメソッドを提供しています。詳細は、APIリソースの[完全なドキュメント](/docs/{{version}}/eloquent-resources)をご覧ください。

### コンソールコマンドの自動登録

> {video} この機能の無料[動画チュートリアル](https://laracasts.com/series/whats-new-in-laravel-5-5/episodes/12)（英語）がLaracastsに用意されています。

新しいコンソールコマンドを作成する場合、コンソールカーネルの`$commands`プロパティへリストする必要はなくなりました。代わりに、新しい`load`メソッドがカーネルの`commands`メソッドから呼び出されています。これは、指定したディレクトリでコンソールコマンドを探し、自動的に登録します。

    /**
     * アプリケーションのコマンド登録
     *
     * @return void
     */
    protected function commands()
    {
        $this->load(__DIR__.'/Commands');

        // ...
    }

### 新しいフロントエンドプリセット

> {video} この機能の無料[動画チュートリアル](https://laracasts.com/series/whats-new-in-laravel-5-5/episodes/4)（英語）がLaracastsに用意されています。

基本的なVueスカフォールドもLaravel5.5に含まれたままですが、新しいフロントエンドプリセットの選択肢が利用できるようになりました。真新しいLaravelアプリケーションで`preset`コマンドを使用し、VueスカフォールドをReactスカフォールドへ交換できます。

    php artisan preset react

もしくは、`none`プリセットを使用すれば、JavaScriptとCSSフレームワークのスカフォールドを取り除くことができます。このプリセットでは、アプリケーションに空のSassファイルと、シンプルなJavaScriptユーティリティがいくつか残るだけです。

    php artisan preset none

> {note} これらのコマンドは、インストールしたてのLaravelアプリケーションで実行する目的で用意されています。既存のアプリケーションでは使用してはいけません。

### キュージョブのチェーン

ジョブのチェーンにより、順番に実行すべきキュージョブのリストを指定できるようになりました。リスト中のあるジョブが失敗すると、それ以降のジョブは実行されません。ジョブチェーンを実行するには、dispatchableなジョブの`withChain`メソッドを使用します。

    ProvisionServer::withChain([
        new InstallNginx,
        new InstallPhp
    ])->dispatch();

### キュージョブのレート制限

アプリケーションがRedisを扱っている場合、回数や時間によりキュージョブの制限ができるようになりました。この機能は、キュージョブがレート制限のあるAPIを取り扱う場合に便利です。例として、特定のタイプのジョブで、６０秒毎に、１０回の制限を指定してみましょう。

    Redis::throttle('key')->allow(10)->every(60)->then(function () {
        // ジョブのロジック処理…
    }, function () {
        // ロックできなかった場合の処理…

        return $this->release(10);
    });

> {tip} 上記の例で`key`は、レート制限したいジョブのタイプを表す、一意の認識文字列です。たとえば、ジョブのクラス名と、（そのジョブに含まれているならば）EloquentモデルのIDを元に、制限できます。

もしくは、ジョブを同時に処理するワーカの最大数を指定することができます。これは、一度に一つのジョブが更新すべきリソースを変更するキュージョブを使用する場合に、役立ちます。`funnel`メソッドの使用例として、一度に１ワーカのみにより処理される、特定のタイプのジョブを制限してみましょう。

    Redis::funnel('key')->limit(1)->then(function () {
        // ジョブのロジック処理…
    }, function () {
        // ロックできなかった場合の処理…

        return $this->release(10);
    });

### 時間ベースの試行

失敗するまでジョブの試行を何度認めるかを定義する代わりに、ジョブのタイムアウト時間を定義することもできます。これにより、指定した時間内で複数回ジョブを試行します。タイムアウト時間を定義するには、ジョブクラスに`retryUntil`メソッドを追加します。

    /**
     * タイムアウトになる時間を決定
     *
     * @return \DateTime
     */
    public function retryUntil()
    {
        return now()->addSeconds(5);
    }

> {tip} キューイベントリスナでも、`retryUntil`メソッドを定義できます。

### バリデーションルールオブジェクト

> {video} この機能の無料[動画チュートリアル](https://laracasts.com/series/whats-new-in-laravel-5-5/episodes/7)（英語）がLaracastsに用意されています。

バリデーションルールオブジェクトは、アプリケーションにカスタムルールを追加する、新しい簡単な方法を提供します。以前のバージョンのLaravelでは、クロージャを使用してカスタムバリデーションルールを追加する、`Validator::extend`が使われていました。しかし、これは改善されました。Laravel5.5では、新しいバリデーションルールを`app/Rules`ディレクトリへ生成するために、`make:rule` Artisan新コマンドを使用します。

    php artisan make:rule ValidName

ルールオブジェクトは２つのメソッドを含みます。`passes`と`message`です。`passes`メソッドは属性の値と名前を受け取り、その属性値が有効であれば`true`、無効であれば`false`を返します。`message`メソッドは、バリデーション失敗時に使用する、バリデーションエラーメッセージを返します。

    <?php

    namespace App\Rules;

    use Illuminate\Contracts\Validation\Rule;

    class ValidName implements Rule
    {
        /**
         * バリデーションの成功を判定
         *
         * @param  string  $attribute
         * @param  mixed  $value
         * @return bool
         */
        public function passes($attribute, $value)
        {
            return strlen($value) === 6;
        }

        /**
         * バリデーションエラーメッセージの取得
         *
         * @return string
         */
        public function message()
        {
            return 'The name must be six characters long.';
        }
    }

ルールが定義できたら、他のバリデーションルールと一緒に、ルールオブジェクトのインスタンスをバリデータへ渡し、指定します。

    use App\Rules\ValidName;

    $request->validate([
        'name' => ['required', new ValidName],
    ]);

### 信用するプロキシの統合

TLS／SSL証明を行うロードバランサの裏でアプリケーションが実行されている場合、アプリケーションが時々HTTPSリンクを生成しないことに、気づくでしょう。典型的な理由は、トラフィックがロードバランサにより８０番ポートへフォワーディングされるため、セキュアなリンクを生成すべきだと判断できないからです。

これを解決するために多くのLaravelユーザーはChris Fidaoさんが作成した、[Trusted Proxies](https://github.com/fideloper/TrustedProxy)パッケージをインストールしています。これはコモンケースですから、Laravel5.5ではデフォルトとしてChrisのパッケージを利用しています。

Laravel5.5では、新しい`App\Http\Middleware\TrustProxies`ミドルウェアがデフォルトとして含まれています。このミドルウェアにより、アプリケーションが信用するプロキシを素早くカスタマイズできます。

    <?php

    namespace App\Http\Middleware;

    use Illuminate\Http\Request;
    use Fideloper\Proxy\TrustProxies as Middleware;

    class TrustProxies extends Middleware
    {
        /**
         * このアプリケーションで信用するプロキシ
         *
         * @var array
         */
        protected $proxies;

        /**
         * 現在のプロキシヘッダーのマップ
         *
         * @var array
         */
        protected $headers = [
            Request::HEADER_FORWARDED => 'FORWARDED',
            Request::HEADER_X_FORWARDED_FOR => 'X_FORWARDED_FOR',
            Request::HEADER_X_FORWARDED_HOST => 'X_FORWARDED_HOST',
            Request::HEADER_X_FORWARDED_PORT => 'X_FORWARDED_PORT',
            Request::HEADER_X_FORWARDED_PROTO => 'X_FORWARDED_PROTO',
        ];
    }

### オンデマンド通知

場合により、アプリケーションの「ユーザー」として保存されていない誰かに対し、通知を送る必要が起きることがあります。`Notification::route`メソッドを使い、通知を送る前にアドホックな通知ルーティング情報を指定できます。

    Notification::route('mail', 'taylor@laravel.com')
                ->route('nexmo', '5555555555')
                ->send(new InvoicePaid($invoice));

### Renderableなmailable

> {video} この機能の無料[動画チュートリアル](https://laracasts.com/series/whats-new-in-laravel-5-5/episodes/6)（英語）がLaracastsに用意されています。

mailableは、ルートから直接返せるようになり、ブラウザでmailableのデザインを素早くレビューできるようになりました。

    Route::get('/mailable', function () {
        $invoice = App\Invoice::find(1);

        return new App\Mail\InvoicePaid($invoice);
    });

### Reportable／Renderable例外

> {video} この機能の無料[動画チュートリアル](https://laracasts.com/series/whats-new-in-laravel-5-5/episodes/18)（英語）がLaracastsに用意されています。

以前のバージョンのLaravelでは、特定の例外に対するカスタムレスポンスをレンダするために、例外ハンドラで「タイプチェック」を使っていました。たとえば、以下のようなコードを例外ハンドラの`render`メソッドに書いていたと思います。

    /**
     * 例外をHTTPレスポンスへレンダ
     *
     * @param  \Illuminate\Http\Request  $request
     * @param  \Exception  $exception
     * @return \Illuminate\Http\Response
     */
    public function render($request, Exception $exception)
    {
        if ($exception instanceof SpecialException) {
            return response(...);
        }

        return parent::render($request, $exception);
    }

Laravel5.5では、`render`メソッドを直接自分の例外で定義できます。これにより、例外に直接レスポンスのレンダロジックを置けるようになり、例外ハンドラに条件判定のロジックが積み重なるのを防ぐことができます。例外のレポートもカスタマイズしたい場合は、クラスに`report`メソッドを定義してください。

    <?php

    namespace App\Exceptions;

    use Exception;

    class SpecialException extends Exception
    {
        /**
         * 例外のレポート
         *
         * @return void
         */
        public function report()
        {
            //
        }

        /**
         * 例外のレンダ
         *
         * @param  \Illuminate\Http\Request
         * @return void
         */
        public function render($request)
        {
            return response(...);
        }
    }

### リクエストのバリデーション

> {video} この機能の無料[動画チュートリアル](https://laracasts.com/series/whats-new-in-laravel-5-5/episodes/2)（英語）がLaracastsに用意されています。

`Illuminate\Http\Request`オブジェクトが、`validate`メソッドを提供するようになり、ルートクロージャやコントローラで送信されてきたリクエストを素早くバリデートできます。

    use Illuminate\Http\Request;

    Route::get('/comment', function (Request $request) {
        $request->validate([
            'title' => 'required|string',
            'body' => 'required|string',
        ]);

        // ...
    });

### 一貫性のある例外処理

フレームワーク全体を通し、バリデーション例外の処理が統一されました。以前は、JSONバリデーションエラーレスポンスのデフォルトフォーマットを変更するには、フレームワークの複数の箇所をカスタマイズする必要がありました。Laravel5.5では、JSONバリデーションレスポンスのデフォルト形式は、以下の規約にしたがっています。

    {
        "message": "The given data was invalid.",
        "errors": {
            "field-1": [
                "Error 1",
                "Error 2"
            ],
            "field-2": [
                "Error 1",
                "Error 2"
            ],
        }
    }

すべてのJSONバリデーションエラーフォーマットは、`App\Exceptions\Handler`クラスの一つのメソッドで定義されています。以下のカスタマイズ例は、Laravel5.4の規約を使用する、JSONバリデーションレスポンスをフォーマットしています。

    use Illuminate\Validation\ValidationException;

    /**
     * バリデーション例外をJSONレスポンスへ変換
     *
     * @param  \Illuminate\Http\Request  $request
     * @param  \Illuminate\Validation\ValidationException  $exception
     * @return \Illuminate\Http\JsonResponse
     */
    protected function invalidJson($request, ValidationException $exception)
    {
        return response()->json($exception->errors(), $exception->status);
    }

### キャッシュロック

キャッシュドライバは、アトミックな「ロック」の取得と開放をサポートするようになりました。これにより、競合状態を心配せずに、自由にロックを取得できるシンプルな手段が提供されました。たとえば、既に処理中の同じタスクを実行するのを防ぐため、実行を試みる前にロックを取得できます。

    if (Cache::lock('lock-name', 60)->get()) {
        // ６０秒ロックを取得し、処理を継続…

        Cache::lock('lock-name')->release();
    } else {
        // ロックを取得できなかった
    }

もしくは、`get`メソッドでクロージャを渡せます。このクロージャはロックが取得された場合のみ実行され、クロージャが終了すると自動的に開放します。

    Cache::lock('lock-name', 60)->get(function () {
        // ６０秒ロックを取得
    });

さらに、ロックできるようになるまで、「ブロック」することもできます。

    if (Cache::lock('lock-name', 60)->block(10)) {
        // ロックが利用できるようになるまで、最長で１０秒待つ
    }

### Bladeの向上

> {video} この機能の無料[動画チュートリアル](https://laracasts.com/series/whats-new-in-laravel-5-5/episodes/10)（英語）がLaracastsに用意されています。

シンプルなカスタム条件文を定義する時、必要以上にカスタムディレクティブのプログラミングが複雑になってしまうことが、時々起きます。そのため、Bladeはクロージャを使用し、カスタム条件ディレクティブを素早く定義できるように、`Blade::if`メソッドを提供しています。例として、現在のアプリケーション環境をチェックするカスタム条件を定義してみましょう。`AppServiceProvider`の`boot`メソッドで行います。

    use Illuminate\Support\Facades\Blade;

    /**
     * サービスの初期処理後に実行
     *
     * @return void
     */
    public function boot()
    {
        Blade::if('env', function ($environment) {
            return app()->environment($environment);
        });
    }

カスタム条件を定義したら、テンプレートの中で簡単に利用できます。

    @env('local')
        // アプリケーションはlocal環境
    @else
        // アプリケーションはlocal環境ではない
    @endenv

Bladeのカスタム条件ディレクティブを簡単に定義できる機能に付け加え、現在のユーザーの認証状態を素早くチェックできる、新しいショートカット記法も追加されました。

    @auth
        // ユーザーは認証済み
    @endauth

    @guest
        // ユーザーは未認証
    @endguest

### 新しいルーティングメソッド

> {video} この機能の無料[動画チュートリアル](https://laracasts.com/series/whats-new-in-laravel-5-5/episodes/16)（英語）がLaracastsに用意されています。

他のURIへリダイレクトするルートを定義する場合は、`Route::redirect`メソッドを使用します。このメソッドは便利な短縮形を提供しているので、単純なリダイレクトを実行するために、完全なルートやコントローラを定義する必要はありません。

    Route::redirect('/here', '/there', 301);

ルートからビューを返すだけの場合は、`Route::view`メソッドを使用します。`redirect`メソッドと同様に、このメソッドはシンプルな短縮形を提供しており、完全なルートやコントローラを定義する必要はありません。`view`メソッドは、最初の引数にURIを取り、ビュー名は第２引数です。更に、オプションの第３引数として、ビューへ渡すデータの配列を指定することもできます。

    Route::view('/welcome', 'welcome');

    Route::view('/welcome', 'welcome', ['name' => 'Taylor']);

### "Sticky"データベース接続

#### `sticky`オプション

read/writeデータベース接続を設定する場合に、新しい`sticky`設定オプションが使えるようになりました。

    'mysql' => [
        'read' => [
            'host' => '192.168.1.1',
        ],
        'write' => [
            'host' => '196.168.1.2'
        ],
        'sticky'    => true,
        'driver'    => 'mysql',
        'database'  => 'database',
        'username'  => 'root',
        'password'  => '',
        'charset' => 'utf8mb4',
        'collation' => 'utf8mb4_unicode_ci',
        'prefix'    => '',
    ],

`sticky`オプションは**オプショナル**値で、現在のリクエストサイクル中にデータベースへ書き込まれたレコードを即時に読み込めるようにします。`sticky`オプションが有効で、現在のリクエストサイクル中にデータベースに対して「書き込み(write)」処理が実行されると、すべての「読み込み(read)」操作で"write"接続が使われるようになります。これによりリクエストサイクル中に書き込まれたデータが、同じリクエスト中にデータベースから即時に読み込まれることが確実になります。この振る舞いが皆さんのアプリケーションで好ましいかは、皆さんが判断してください。
