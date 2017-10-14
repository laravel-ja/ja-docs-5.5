# HTTPセッション

- [イントロダクション](#introduction)
    - [設定](#configuration)
    - [ドライバの事前要件](#driver-prerequisites)
- [セッションの使用](#using-the-session)
    - [データ取得](#retrieving-data)
    - [データ保存](#storing-data)
    - [フラッシュデータ](#flash-data)
    - [データ削除](#deleting-data)
    - [セッションIDの再生性](#regenerating-the-session-id)
- [カスタムセッションドライバの追加](#adding-custom-session-drivers)
    - [ドライバの実装](#implementing-the-driver)
    - [ドライバの登録](#registering-the-driver)

<a name="introduction"></a>
## イントロダクション

HTTP駆動のアプリケーションはステートレスのため、リクエスト間に渡りユーザーに関する情報を保存するセッションが提供されています。Laravelは記述的で統一されたAPIを使える様々なバックエンドのセッションを用意しています。人気のある[Memcached](https://memcached.org)や[Redis](https://redis.io)、データベースも始めからサポートしています。

<a name="configuration"></a>
### 設定

セッションの設定は`config/session.php`にあります。このファイルのオプションには詳しくコメントがついていますので確認して下さい。ほとんどのアプリケーションでうまく動作できるように、Laravelは`file`セッションドライバをデフォルトとして設定しています。実働環境のアプリケーションではセッションの動作をより早くするために、`memcached`や`redis`ドライバの使用を考慮しましょう。

セッションドライバ(`driver`)はリクエスト毎のセッションデータをどこに保存するかを決めます。Laravelには最初から素晴らしいドライバが用意されています。

<div class="content-list" markdown="1">
- `file` - セッションは`storage/framework/sessions`に保存されます。
- `cookie` - セションは暗号化され安全なクッキーに保存されます。
- `database` - セッションはリレーショナルデータベースへ保存されます。
- `memcached`／`redis` - セッションはスピードの早いキャッシュベースの保存域に保存されます。
- `array` - セッションはPHPの配列として保存されるだけで、リクエスト間で継続しません。
</div>

> {tip} セッションデータを持続させないため、arrayドライバは通常[テスト](/docs/{{version}}/testing)時に使用します。

<a name="driver-prerequisites"></a>
### ドライバの事前要件

#### データベース

`database`セッションドライバを使う場合、セッションアイテムを含むテーブルを作成する必要があります。以下にこのテーブル宣言のサンプル「スキーマ」を示します。

    Schema::create('sessions', function ($table) {
        $table->string('id')->unique();
        $table->unsignedInteger('user_id')->nullable();
        $table->string('ip_address', 45)->nullable();
        $table->text('user_agent')->nullable();
        $table->text('payload');
        $table->integer('last_activity');
    });

`session:table` Artisanコマンドを使えば、このマイグレーションが生成できます。

    php artisan session:table

    php artisan migrate

#### Redis

ReidsセッションをLaravelで使用する前に、Composerで`predis/predis`パッケージ(~1.0)をインストールする必要があります。Redis接続は`database`設定ファイルで設定します。`session`設定ファイルでは、`connection`オプションで、どのRedis接続をセッションで使用するか指定します。

<a name="using-the-session"></a>
## セッションの使用

<a name="retrieving-data"></a>
### データ取得

Laravelでセッションを操作するには、主に２つの方法があります。グローバルな`session`ヘルパを使用する方法と、コントローラメソッドにタイプヒントで指定できる`Request`インスタンスを経由する方法です。最初は`Request`インスタンスを経由する方法を見てみましょう。コントローラのメソッドに指定した依存インスタンスは、Laravelの[サービスコンテナにより](/docs/{{version}}/container)、自動的に注入されることを覚えておきましょう。

    <?php

    namespace App\Http\Controllers;

    use Illuminate\Http\Request;
    use App\Http\Controllers\Controller;

    class UserController extends Controller
    {
        /**
         * 指定されたユーザーのプロフィールを表示
         *
         * @param  Request  $request
         * @param  int  $id
         * @return Response
         */
        public function show(Request $request, $id)
        {
            $value = $request->session()->get('key');

            //
        }
    }

`get`メソッドでセッションから値を取り出すとき、第２引数にデフォルト値も指定できます。このデフォルト値は、セッションに指定したキーが存在していなかった場合に、返されます。`get`メソッドのデフォルト値に「クロージャ」を渡した場合に、要求したキーが存在しなければ、その「クロージャ」が実行され、結果が返されます。

    $value = $request->session()->get('key', 'default');

    $value = $request->session()->get('key', function () {
        return 'default';
    });

#### sessionグローバルヘルパ

グローバルな`session` PHP関数で、セッションからデータを出し入れすることもできます。`session`ヘルパが文字列ひとつだけで呼び出されると、そのセッションキーに対する値を返します。ヘルパがキー／値ペアの配列で呼び出されると、それらの値はセッションへ保存されます。

    Route::get('home', function () {
        // セッションから一つのデータを取得する
        $value = session('key');

        // デフォルト値を指定する場合
        $value = session('key', 'default');

        // セッションへ一つのデータを保存する
        session(['key' => 'value']);
    });

> {tip} セッションをHTTPリクエストインスタンスを経由する場合と、グローバルな`session`ヘルパを使用する場合では、実践上の違いがあります。どんなテストケースであろうとも使用可能な、`assertSessionHas`メソッドを利用して、どちらの手法も[テスト可能](/docs/{{version}}/testing)です。

#### 全セッションデータの取得

セッション中の全データを取得する場合は、`all`メソッドを使います。

    $data = $request->session()->all();

#### セッション中のアイテム存在を確認

セッションへ値が存在するか調べたい場合は、`has`メソッドを使います。その値が存在し、`null`でない場合は`true`が返ります。

    if ($request->session()->has('users')) {
        //
    }

セッション中に、たとえ値が`null`であろうとも存在していることを確認したい場合は、`exists`メソッドを使います。`exists`メソッドは、値が存在していれば`true`を返します。

    if ($request->session()->exists('users')) {
        //
    }

<a name="storing-data"></a>
### データ保存

セッションへデータを保存する場合、通常`put`メソッドか、`session`ヘルパを使用します。

    // リクエストインスタンス経由
    $request->session()->put('key', 'value');

    // グローバルヘルパ使用
    session(['key' => 'value']);

#### 配列セッション値の追加

`push`メソッドは新しい値を配列のセッション値へ追加します。たとえば`user.teams`キーにチーム名の配列が含まれているなら、新しい値を次のように追加できます。

    $request->session()->push('user.teams', 'developers');

#### 取得後アイテムを削除

`pull`メソッド一つで、セッションからアイテムを取得後、削除できます。

    $value = $request->session()->pull('key', 'default');

<a name="flash-data"></a>
### フラッシュデータ

次のリクエスト間だけセッションにアイテムを保存したいことは良くあります。`flash`メソッドを使ってください。`flash`メソッドは直後のHTTPリクエストの間だけセッションにデータを保存します。それ以降は削除されます。フラッシュデータは主にステータスメッセージなど継続しない情報に便利です。

    $request->session()->flash('status', 'Task was successful!');

フラッシュデータをその先のリクエストまで持続させたい場合は、`reflash`メソッドを使い、全フラッシュデータを次のリクエストまで持続させられます。特定のフラッシュデータのみ持続させたい場合は、`keep`メソッドを使います。

    $request->session()->reflash();

    $request->session()->keep(['username', 'email']);

<a name="deleting-data"></a>
### データ削除

`forget`メソッドでセッションからデータを削除できます。セッションから全データを削除したければ、`flush`メソッドが使用できます。

    $request->session()->forget('key');

    $request->session()->flush();

<a name="regenerating-the-session-id"></a>
### セッションIDの再生性

セッションIDの再生成は多くの場合、悪意のあるユーザーからの、アプリケーションに対する[session fixation](https://en.wikipedia.org/wiki/Session_fixation)攻撃を防ぐために行います。

Laravelに組み込まれている`LoginController`を使用していれば、認証中にセッションIDは自動的に再生性されます。しかし、セッションIDを任意に再生成する必要があるのでしたら、`regenerate`メソッドを使ってください。

    $request->session()->regenerate();

<a name="adding-custom-session-drivers"></a>
## カスタムセッションドライバの追加

<a name="implementing-the-driver"></a>
#### ドライバの実装

カスタムセッションドライバでは、`SessionHandlerInterface`を実装してください。このインターフェイスには実装する必要のある、シンプルなメソッドが数個含まれています。MongoDBの実装をスタブしてみると、次のようになります。

    <?php

    namespace App\Extensions;

    class MongoHandler implements SessionHandlerInterface
    {
        public function open($savePath, $sessionName) {}
        public function close() {}
        public function read($sessionId) {}
        public function write($sessionId, $data) {}
        public function destroy($sessionId) {}
        public function gc($lifetime) {}
    }

> {tip} こうした拡張を含むディレクトリをLaravelでは用意していません。お好きな場所に設置してください。上記の例では、`Extension`ディレクトリを作成し、`MongoHandler`ファイルを設置しています。

これらのメソッドの目的を読んだだけでは理解しづらいため、それぞれのメソッドを簡単に見てみましょう。

<div class="content-list" markdown="1">
- `open`メソッドは通常ファイルベースのセッション保存システムで使われます。Laravelは`file`セッションドライバを用意していますが、皆さんはこのメソッドに何も入れる必要はないでしょう。空のスタブのままで良いでしょう。実際、PHPが実装するように要求しているこのメソッドは、下手なインターフェイスデザインなのです。
- `close`メソッドも`open`と同様に通常は無視できます。ほどんどのドライバでは必要ありません。
- `read`メソッドは指定された`$sessionId`と紐付いたセッションデータの文字列バージョンを返します。取得や保存時にドライバ中でデータをシリアライズしたり、他のエンコード作業を行ったりする必要はありません。Laravelがシリアライズを行います。
- `write`メソッドはMongoDBやDynamoなどの持続可能なストレージに、`$sessionId`に紐付け指定した`$data`文字列を書き出します。  Again, you should not perform any serialization - Laravel will have already handled that for you.
- `destroy`メソッドは持続可能なストレージから`$sessionId`に紐付いたデータを取り除きます。
- `gc`メソッドは指定したUNIXタイムスタンプの`$lifetime`よりも古い前セッションデータを削除します。自前で破棄するMemcachedやRedisのようなシステムでは、このメソッドは空のままにしておきます。
</div>

<a name="registering-the-driver"></a>
#### ドライバの登録

ドライバを実装したら、フレームワークへ登録する準備が整いました。Laravelのセッションバックエンドへドライバを追加するには、`Session`[ファサード](/docs/{{version}}/facades)の`extend`メソッドを呼び出します。[サービスプロバイダ](/docs/{{version}}/providers)の`boot`メソッドから、`extend`メソッドを呼び出してください。既存の`AppServiceProvider`か真新しく作成し、呼び出してください。

    <?php

    namespace App\Providers;

    use App\Extensions\MongoSessionStore;
    use Illuminate\Support\Facades\Session;
    use Illuminate\Support\ServiceProvider;

    class SessionServiceProvider extends ServiceProvider
    {
        /**
         * サービス起動処理の事前登録
         *
         * @return void
         */
        public function boot()
        {
            Session::extend('mongo', function ($app) {
                // SessionHandlerInterfaceの実装を返す…
                return new MongoSessionStore;
            });
        }

        /**
         * コンテナへ結合を登録する
         *
         * @return void
         */
        public function register()
        {
            //
        }
    }

セッションドライバを登録したら、`config/session.php`設定ファイルで`mongo`ドライバが使用できます。
