# ファサード

- [イントロダクション](#introduction)
- [いつファサードを使うか](#when-to-use-facades)
    - [ファサード vs. 依存注入](#facades-vs-dependency-injection)
    - [ファサード vs. ヘルパ関数](#facades-vs-helper-functions)
- [ファサードの仕組み](#how-facades-work)
- [ファサードクラス一覧](#facade-class-reference)

<a name="introduction"></a>
## イントロダクション

ファサード（facade、「入り口」）はアプリケーションの[サービスコンテナ](/docs/{{version}}/container)に登録したクラスへ、「静的」なインターフェイスを提供します。Laravelのほとんどの機能に対して、ファサードが用意されています。Laravelの「ファサード」は、サービスコンテナ下で動作しているクラスに対し、"static proxy"として動作しています。これにより伝統的な静的メソッドよりもテストの行いやすさと柔軟性を保ちながらも、簡潔で記述的であるという利点があります。

Laravelのファサードはすべて、`Illuminate\Support\Facades`名前空間下で定義されています。ですから、簡単にファサードへアクセスできます。

    use Illuminate\Support\Facades\Cache;

    Route::get('/cache', function () {
        return Cache::get('key');
    });

フレームワークの様々な機能をデモンストレートするために、Laravelのドキュメント全般でたくさんの例がファサードを使用しています。

<a name="when-to-use-facades"></a>
## いつファサードを使うか

ファサードにはたくさんの利点があります。自分で取り込んだり、設定したりする必要があり、長くて覚えにくいクラス名を使わずに、Laravelの機能を簡素で覚えやすい文法で使ってもらえます。その上に、PHPの動的メソッドのユニークな使用方法のおかげで、簡単にテストができます。

しかしながら、ファサードの使用にはいくつか気をつけるべき点も存在します。ファサードの一番の危険性は、クラスの責任範囲の暴走です。ファサードはとても簡単に使用でき、依存注入も必要ないため、簡単にクラスが成長し続ける結果、一つのクラスで多くのファサードが使われます。依存注入を使用すれば、クラスが大きくなりすぎることに伴う、大きなコンストラクタの視覚的なフィードバックにより、この危険性は抑制されます。ですから、ファサードを使用するときは、クラスの責任範囲を小さくとどめるため、クラスサイズに特に注意をはらいましょう。

> {tip} Laravelに関連した、サードパーティパッケージを構築する場合は、ファサードの代わりに[Laravelの契約](/docs/{{version}}/contracts)を使うほうが好ましいでしょう。Laravel自身の外でパッケージを構築するわけですから、Laravelのテストヘルパへアクセスする必要はありません。

<a name="facades-vs-dependency-injection"></a>
### ファサード vs. 依存注入

依存注入の最大の利便性は、注入するクラスの実装を入れ替えられるという機能です。モックやスタブを注入し、そうした代替オブジェクトの様々なメソッドのアサートが行えるため、テスト中に便利です。

本当の静的クラスメソッドをモックしたり、スタブにすることは、通常は不可能です。しかしファサードは、サービスコンテナが依存解決したオブジェクトの代替メソッドを呼び出すために、動的メソッドが使えますので、注入したクラスインスタンスをテストするのと同様に、ファサードを実際にテスト可能です。

    use Illuminate\Support\Facades\Cache;

    Route::get('/cache', function () {
        return Cache::get('key');
    });

`Cache::get`メソッドが、予想した引数で呼び出されることを確認するために、以下のようなテストを書けます。

    use Illuminate\Support\Facades\Cache;

    /**
     * 基本的なテスト機能の例
     *
     * @return void
     */
    public function testBasicExample()
    {
        Cache::shouldReceive('get')
             ->with('key')
             ->andReturn('value');

        $this->visit('/cache')
             ->see('value');
    }

<a name="facades-vs-helper-functions"></a>
### ファサード vs. ヘルパ関数

ファサードに加え、Laravelは様々な「ヘルパ」関数を用意しており、ビューの生成、イベントの発行、ジョブの起動、HTTPレスポンスの送信など、一般的なタスクを実行できます。こうしたヘルパ関数の多くは、対応するファサードと同じ機能を実行します。たとえば、以下のファサードとヘルパの呼び出しは、同じ働きをします。

    return View::make('profile');

    return view('profile');

ここではファサードとヘルパ関数との間に、全く違いはありません。ヘルパ関数を使う場合も、対応するファサードと全く同様にテストできます。たとえば、以下のルートが存在するとしましょう。

    Route::get('/cache', function () {
        return cache('key');
    });

内部で`cache`ヘルパは、`Cache`ファサードの裏で動作しているクラスの`get`メソッドを呼び出します。ですから、ヘルパ関数を使用していても、期待する引数でメソッドが呼びだされていることを確認する、以下のテストを書けます。

    use Illuminate\Support\Facades\Cache;

    /**
     * 基本的なテスト機能の例
     *
     * @return void
     */
    public function testBasicExample()
    {
        Cache::shouldReceive('get')
             ->with('key')
             ->andReturn('value');

        $this->visit('/cache')
             ->see('value');
    }

<a name="how-facades-work"></a>
## ファサードの仕組み

Laravelアプリケーション中で、ファサードとは、コンテナを通じてオブジェクトにアクセス方法を提供するクラスのことです。`Facade`クラス中の仕組みでこれを行なっています。Laravelのファサードと皆さんが作成するカスタムファサードは、`Illuminate\Support\Facades\Facade`クラスを拡張します。

`Facade`基本クラスは、ファサードへの関数呼び出しをコンテナにより依存解決されたオブジェクトへ送るため、`__callStatic()`マジックメソッドを使用します。下の例では、Laravelのキャッシュシステムを呼び出しています。これを読むと一見、`Cache`クラスのstaticな`get`メソッドが呼び出されているのだと考えてしまうことでしょう。

    <?php

    namespace App\Http\Controllers;

    use App\Http\Controllers\Controller;
    use Illuminate\Support\Facades\Cache;

    class UserController extends Controller
    {
        /**
         * 指定したユーザーのプロフィール表示
         *
         * @param  int  $id
         * @return Response
         */
        public function showProfile($id)
        {
            $user = Cache::get('user:'.$id);

            return view('profile', ['user' => $user]);
        }
    }

ファイルの先頭で、`Cache`ファサードを取り込んでいることに注目です。このファサードサービスは、`Illuminate\Contracts\Cache\Factory`インターフェイスの裏にある実装へアクセスするプロキシとして動作します。ファサードを使ったメソッド呼び出しは、裏のLaravelのキャッシュサービスの実装へ渡されます。

そのため、`Illuminate\Support\Facades\Cache`クラスを見てもらえば、staticの`get`メソッドは存在していないことが分かります。

    class Cache extends Facade
    {
        /**
         * コンポーネントの登録名を取得
         *
         * @return string
         */
        protected static function getFacadeAccessor() { return 'cache'; }
    }

かわりに`Cache`ファサードは、`Facade`ベースクラスを拡張し、`getFacadeAccessor()`メソッドを定義しています。このメソッドの仕事は、サービスコンテナの結合名を返すことです。ユーザーが`Cache`ファサードのどのstaticメソッドを利用しようと、Laravelは[サービスコンテナ](/docs/{{version}}/container)から`cache`に結び付けられたインスタンスを依存解決し、要求されたメソッドを（この場合は`get`）そのオブジェクトに対し実行します。

<a name="facade-class-reference"></a>
## ファサードクラス一覧

以下は全ファサードと実際のクラスの一覧です。これは特定のファサードを元にし、APIドキュメントを素早く探したい場合に便利な道具になります。対応する[サービスコンテナ結合キー](/docs/{{version}}/container)も記載しています。

ファサード  |  クラス  |  サービスコンテナ結合
------------- | ------------- | -------------
App  |  [Illuminate\Foundation\Application](https://laravel.com/api/{{version}}/Illuminate/Foundation/Application.html)  | `app`
Artisan  |  [Illuminate\Contracts\Console\Kernel](https://laravel.com/api/{{version}}/Illuminate/Contracts/Console/Kernel.html)  |  `artisan`
Auth  |  [Illuminate\Auth\AuthManager](https://laravel.com/api/{{version}}/Illuminate/Auth/AuthManager.html)  |  `auth`
Blade  |  [Illuminate\View\Compilers\BladeCompiler](https://laravel.com/api/{{version}}/Illuminate/View/Compilers/BladeCompiler.html)  |  `blade.compiler`
Bus  |  [Illuminate\Contracts\Bus\Dispatcher](https://laravel.com/api/{{version}}/Illuminate/Contracts/Bus/Dispatcher.html)  |  &nbsp;
Cache  |  [Illuminate\Cache\Repository](https://laravel.com/api/{{version}}/Illuminate/Cache/Repository.html)  |  `cache`
Config  |  [Illuminate\Config\Repository](https://laravel.com/api/{{version}}/Illuminate/Config/Repository.html)  |  `config`
Cookie  |  [Illuminate\Cookie\CookieJar](https://laravel.com/api/{{version}}/Illuminate/Cookie/CookieJar.html)  |  `cookie`
Crypt  |  [Illuminate\Encryption\Encrypter](https://laravel.com/api/{{version}}/Illuminate/Encryption/Encrypter.html)  |  `encrypter`
DB  |  [Illuminate\Database\DatabaseManager](https://laravel.com/api/{{version}}/Illuminate/Database/DatabaseManager.html)  |  `db`
DB (Instance)  |  [Illuminate\Database\Connection](https://laravel.com/api/{{version}}/Illuminate/Database/Connection.html)  |  &nbsp;
Event  |  [Illuminate\Events\Dispatcher](https://laravel.com/api/{{version}}/Illuminate/Events/Dispatcher.html)  |  `events`
File  |  [Illuminate\Filesystem\Filesystem](https://laravel.com/api/{{version}}/Illuminate/Filesystem/Filesystem.html)  |  `files`
Gate  |  [Illuminate\Contracts\Auth\Access\Gate](https://laravel.com/api/{{version}}/Illuminate/Contracts/Auth/Access/Gate.html)  |  &nbsp;
Hash  |  [Illuminate\Contracts\Hashing\Hasher](https://laravel.com/api/{{version}}/Illuminate/Contracts/Hashing/Hasher.html)  |  `hash`
Lang  |  [Illuminate\Translation\Translator](https://laravel.com/api/{{version}}/Illuminate/Translation/Translator.html)  |  `translator`
Log  |  [Illuminate\Log\Writer](https://laravel.com/api/{{version}}/Illuminate/Log/Writer.html)  |  `log`
Mail  |  [Illuminate\Mail\Mailer](https://laravel.com/api/{{version}}/Illuminate/Mail/Mailer.html)  |  `mailer`
Notification  |  [Illuminate\Notifications\ChannelManager](https://laravel.com/api/{{version}}/Illuminate/Notifications/ChannelManager.html)  |  &nbsp;
Password  |  [Illuminate\Auth\Passwords\PasswordBrokerManager](https://laravel.com/api/{{version}}/Illuminate/Auth/Passwords/PasswordBrokerManager.html)  |  `auth.password`
Queue  |  [Illuminate\Queue\QueueManager](https://laravel.com/api/{{version}}/Illuminate/Queue/QueueManager.html)  |  `queue`
Queue (Instance)  |  [Illuminate\Contracts\Queue\Queue](https://laravel.com/api/{{version}}/Illuminate/Contracts/Queue/Queue.html)  |  `queue`
Queue (Base Class) |  [Illuminate\Queue\Queue](https://laravel.com/api/{{version}}/Illuminate/Queue/Queue.html)  |  &nbsp;
Redirect  |  [Illuminate\Routing\Redirector](https://laravel.com/api/{{version}}/Illuminate/Routing/Redirector.html)  |  `redirect`
Redis  |  [Illuminate\Redis\Database](https://laravel.com/api/{{version}}/Illuminate/Redis/Database.html)  |  `redis`
Request  |  [Illuminate\Http\Request](https://laravel.com/api/{{version}}/Illuminate/Http/Request.html)  |  `request`
Response  |  [Illuminate\Contracts\Routing\ResponseFactory](https://laravel.com/api/{{version}}/Illuminate/Contracts/Routing/ResponseFactory.html)  |  &nbsp;
Route  |  [Illuminate\Routing\Router](https://laravel.com/api/{{version}}/Illuminate/Routing/Router.html)  |  `router`
Schema  |  [Illuminate\Database\Schema\Blueprint](https://laravel.com/api/{{version}}/Illuminate/Database/Schema/Blueprint.html)  |  &nbsp;
Session  |  [Illuminate\Session\SessionManager](https://laravel.com/api/{{version}}/Illuminate/Session/SessionManager.html)  |  `session`
Session (Instance)  |  [Illuminate\Session\Store](https://laravel.com/api/{{version}}/Illuminate/Session/Store.html)  |  &nbsp;
Storage  |  [Illuminate\Contracts\Filesystem\Factory](https://laravel.com/api/{{version}}/Illuminate/Contracts/Filesystem/Factory.html)  |  `filesystem`
URL  |  [Illuminate\Routing\UrlGenerator](https://laravel.com/api/{{version}}/Illuminate/Routing/UrlGenerator.html)  |  `url`
Validator  |  [Illuminate\Validation\Factory](https://laravel.com/api/{{version}}/Illuminate/Validation/Factory.html)  |  `validator`
Validator (Instance)  |  [Illuminate\Validation\Validator](https://laravel.com/api/{{version}}/Illuminate/Validation/Validator.html) |  &nbsp;
View  |  [Illuminate\View\Factory](https://laravel.com/api/{{version}}/Illuminate/View/Factory.html)  |  `view`
View (Instance)  |  [Illuminate\View\View](https://laravel.com/api/{{version}}/Illuminate/View/View.html)  |  &nbsp;
