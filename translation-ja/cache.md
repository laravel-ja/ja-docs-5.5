# キャッシュ

- [設定](#configuration)
    - [ドライバ事前要件](#driver-prerequisites)
- [キャッシュの使用法](#cache-usage)
    - [キャッシュインスタンスの取得](#obtaining-a-cache-instance)
    - [キャッシュからアイテム取得](#retrieving-items-from-the-cache)
    - [キャッシュへアイテム保存](#storing-items-in-the-cache)
    - [キャッシュからのアイテム削除](#removing-items-from-the-cache)
    - [cacheヘルパ](#the-cache-helper)
- [キャッシュタグ](#cache-tags)
    - [タグ付けしたキャッシュアイテムの保存](#storing-tagged-cache-items)
    - [タグ付けしたキャッシュアイテムへのアクセス](#accessing-tagged-cache-items)
    - [タグ付けしたキャッシュアイテムの削除](#removing-tagged-cache-items)
- [カスタムキャッシュドライバの追加](#adding-custom-cache-drivers)
    - [ドライバープログラミング](#writing-the-driver)
    - [ドライバ登録](#registering-the-driver)
- [イベント](#events)

<a name="configuration"></a>
## 設定

Laravelは読み書きしやすい、多くのキャッシュシステムに対する統一したAPIを提供します。キャッシュの設定は、`config/cache.php`で指定します。アプリケーション全体のデフォルトとして使用するキャッシュドライバをこのファイルの中で指定します。[Memcached](https://memcached.org)や[Redis](https://redis.io)など、人気のあるキャッシュシステムをLaravelは最初からサポートしています。

キャッシュ設定ファイルは、様々な他のオプションも含んでいます。コメントで説明してありますので、よく読んで確認してください。Laravelのデフォルトとして、`file`キャッシュドライバが設定されています。ファイルシステムへオブジェクトをシリアライズして保存します。大きなアプリケーションではMemecachedやAPCのような、より堅牢なドライバを使うことをおすすめします。複数のドライバを使用するキャッシュ設定も可能です。

<a name="driver-prerequisites"></a>
### ドライバ事前要件

#### データベース

データベースをキャッシュドライバに使用する場合、キャッシュアイテムを構成するテーブルを用意する必要があります。このテーブルの「スキーマ」を定義するサンプルを見てください。

    Schema::create('cache', function ($table) {
        $table->string('key')->unique();
        $table->text('value');
        $table->integer('expiration');
    });

> {tip} 正確なスキーマのマイグレーションを生成するために、`php artisan cache:table` Artisanコマンドを使用することもできます。

#### Memcached

Memcachedキャッシュを使用する場合は、[Memcached PECLパッケージ](https://pecl.php.net/package/memcached)をインストールする必要があります。全Memcachedサーバは、`config/cache.php`設定ファイルにリストしてください。

    'memcached' => [
        [
            'host' => '127.0.0.1',
            'port' => 11211,
            'weight' => 100
        ],
    ],

さらに、UNIXソケットパスへ、`host`オプションを設定することもできます。これを行うには`port`オプションに`0`を指定してください。

    'memcached' => [
        [
            'host' => '/var/run/memcached/memcached.sock',
            'port' => 0,
            'weight' => 100
        ],
    ],

#### Redis

LaravelでRedisを使う前に、Composerで`predis/predis`パッケージ（~1.0）、もしくはPECLでPhpRedis PHP拡張のどちらかをインストールしておく必要があります。

Redisの設定についての詳細は、[Laravelドキュメントページ](/docs/{{version}}/redis#configuration)を読んでください。

<a name="cache-usage"></a>
## キャッシュの使用法

<a name="obtaining-a-cache-instance"></a>
### キャッシュインスタンスの取得

`Illuminate\Contracts\Cache\Factory`と`Illuminate\Contracts\Cache\Repository`[契約](/docs/{{version}}/contracts)は、Laravelのキャッシュサービスへのアクセスを提供します。`Factory`契約は、アプリケーションで定義している全キャッシュドライバへのアクセスを提供します。`Repository`契約は通常、`cache`設定ファイルで指定している、アプリケーションのデフォルトキャッシュドライバの実装です。

しかし、このドキュメント全体で使用している、`Cache`ファサードも利用できます。`Cache`ファサードは裏で動作している、Laravelキャッシュ契約の実装への便利で簡潔なアクセスを提供しています。

    <?php

    namespace App\Http\Controllers;

    use Illuminate\Support\Facades\Cache;

    class UserController extends Controller
    {
        /**
         * アプリケーションの全ユーザーリストの表示
         *
         * @return Response
         */
        public function index()
        {
            $value = Cache::get('key');

            //
        }
    }

#### 複数のキャッシュ保存先へのアクセス

`Cache`ファサードの`store`メソッドを使い、様々なキャッシュ保存域へアクセスできます。`store`メソッドに渡すキーは、`cache`設定ファイルの`stores`設定配列にリストしている保存域の一つです。

    $value = Cache::store('file')->get('foo');

    Cache::store('redis')->put('bar', 'baz', 10);

<a name="retrieving-items-from-the-cache"></a>
### キャッシュからアイテム取得

`Cache`ファサードの`get`メソッドは、キャッシュからアイテムを取得するために使用します。アイテムがキャッシュに存在していない場合は、`null`が返されます。アイテムが存在していない時に返したい、カスタムデフォルト値を`get`メソッドの第２引数として渡すこともできます。

    $value = Cache::get('key');

    $value = Cache::get('key', 'default');

デフォルト値として「クロージャ」を渡すこともできます。キャッシュに指定したアイテムが存在していない場合、「クロージャ」の結果が返されます。クロージャを渡すことで、データベースや外部サービスからデフォルト値を取得するのを遅らせることができます。

    $value = Cache::get('key', function () {
        return DB::table(...)->get();
    });

#### アイテムの存在確認

`has`メソッドで、キャッシュにアイテムが存在しているかを調べることができます。このメソッドは、値が`null`か`false`の場合、`false`を返します。

    if (Cache::has('key')) {
        //
    }

#### 値の増減

`increment`と`decrement`メソッドはキャッシュの整数アイテムの値を調整するために使用します。両方のメソッドともそのアイテムの値をどのくらい増減させるかの増分をオプションの第２引数に指定できます。

    Cache::increment('key');
    Cache::increment('key', $amount);
    Cache::decrement('key');
    Cache::decrement('key', $amount);

#### 取得不可時更新

キャッシュからアイテムを取得しようとして、指定したアイテムが存在しない場合は、デフォルト値を保存したい場合もあるでしょう。たとえば、全ユーザーをキャッシュから取得しようとし、存在していない場合はデータベースから取得しキャッシュへ追加したい場合です。`Cache::remember`メソッドを使用します。

    $value = Cache::remember('users', $minutes, function () {
        return DB::table('users')->get();
    });

キャッシュに存在しない場合、`remember`メソッドに渡された「クロージャ」が実行され、結果がキャッシュに保存されます。

`rememberForever`メソッドでアイテムをキャッシュから取得するか、できない場合は永久に保存できます。

    $value = Cache::rememberForever('users', function() {
        return DB::table('users')->get();
    });

#### 取得後削除

キャッシュからアイテムを取得した後に削除したい場合は、`pull`メソッドを使用します。`get`メソッドと同様にキャッシュにアイテムが存在していない場合は、`null`が返ります。

    $value = Cache::pull('key');

<a name="storing-items-in-the-cache"></a>
### キャッシュへアイテム保存

`Cache`ファサードの`put`メソッドにより、キャッシュにアイテムを保存できます。キャッシュにアイテムを保存するときには、何分保存するかを指定する必要があります。

    Cache::put('key', 'value', $minutes);

どのくらいでアイテムが無効になるかを分数で指定する代わりに、キャッシュされたアイテムの有効期限を示す`DateTime`インスタンスを渡すこともできます。

    $expiresAt = now()->addMinutes(10);

    Cache::put('key', 'value', $expiresAt);

#### 非存在時保存

`add`メソッドはキャッシュに保存されていない場合のみ、そのアイテムを保存します。キャッシュに実際にアイテムが追加された場合は`true`が返ってきます。そうでなければ`false`が返されます。

    Cache::add('key', 'value', $minutes);

#### アイテムを永遠に保存

`forever`メソッドはそのアイテムをキャッシュへ永遠に保存します。こうした値は有効期限が切れないため、`forget`メソッドを使用し、削除する必要があります。

    Cache::forever('key', 'value');

> {tip} Memcachedドライバーを使用する場合、キャッシュが最大値に達すると、"forever"を指定したアイテムも削除されます。

<a name="removing-items-from-the-cache"></a>
### キャッシュからのアイテム削除

`forget`メソッドでキャッシュからアイテムを削除します。

    Cache::forget('key');

キャッシュ全体をクリアしたい場合は`flush`メソッドを使います。

    Cache::flush();

> {note} `flush`メソッドは、キャッシュのプレフィックスを考慮せずに、キャッシュから全アイテムを削除します。他のアプリケーションと共有するキャッシュを削除するときは、利用を熟考してください。

<a name="the-cache-helper"></a>
### cacheヘルパ

`Cache`ファサードと[Cache契約](/docs/{{version}}/contracts)に付け加え、キャッシュからのデータ取得／保存を行う、グローバル`cache`関数も使用できます。`cache`関数を文字列引数一つで呼び出す場合、指定したキーの値を返します。

    $value = cache('key');

キー／値ペアの配列と、有効期間を関数へ渡す場合、指定期間の間、値をキャッシュへ保存します。

    cache(['key' => 'value'], $minutes);

    cache(['key' => 'value'], now()->addSeconds(10));

> グローバル`cache`関数への呼び出しをテストする場合、[ファサードのテスト](/docs/{{version}}/mocking#mocking-facades)と同様に、`Cache::shouldReceive`メソッドを使います。

<a name="cache-tags"></a>
## キャッシュタグ

> {note} `file`と`database`キャッシュドライバ使用時、キャッシュタグはサポートされません。また、"forever"を使い、複数のタグをつけたキャッシュを使用する場合、古いレコードを自動的にパージする`memcached`のようなドライバがパフォーマンス的に最適です。

<a name="storing-tagged-cache-items"></a>
### タグ付けキャッシュアイテムの保存

キャッシュタグにより、キャッシュ中の関連するアイテムへタグ付けできます。その後、指定したタグがつけられたキャッシュの値を全部削除できます。タグを順番に指定する配列を渡すことで、タグ付けしたキャッシュへアクセスできます。例としてタグ付けしたキャッシュにアクセスし、キャッシュへ値を`put`してみましょう。

    Cache::tags(['people', 'artists'])->put('John', $john, $minutes);

    Cache::tags(['people', 'authors'])->put('Anne', $anne, $minutes);

<a name="accessing-tagged-cache-items"></a>
### タグ付けキャッシュアイテムへのアクセス

タグ付けしたキャッシュアイテムを取得するには、`tags`メソッドに同じ順序でタグのリストを渡し、続けて`get`メソッドで取得したいキーを指定します。

    $john = Cache::tags(['people', 'artists'])->get('John');

    $anne = Cache::tags(['people', 'authors'])->get('Anne');

<a name="removing-tagged-cache-items"></a>
### タグ付けキャッシュアイテムの削除

タグ一つ、もしくはタグのリストに結びついた全アイテムを一度に消去することができます。たとえば、次の実行文は`people`か`authors`のどちらか、または両方にタグ付けされたキャッシュを全部削除します。ですから、`Anne`と`John`は両方共キャッシュから削除されます。

    Cache::tags(['people', 'authors'])->flush();

対照的に、次の実行分では`authors`にタグ付けしたキャッシュのみ削除されますので、`Anne`が削除され、`John`は残ります。

    Cache::tags('authors')->flush();

<a name="adding-custom-cache-drivers"></a>
## カスタムキャッシュドライバの追加

<a name="writing-the-driver"></a>
### ドライバープログラミング

カスタムキャッシュドライバを作成するには、`Illuminate\Contracts\Cache\Store`[契約](/docs/{{version}}/contracts)を最初に実装する必要があります。そのため、MongoDBキャッシュドライバは、以下のような実装になるでしょう。

    <?php

    namespace App\Extensions;

    use Illuminate\Contracts\Cache\Store;

    class MongoStore implements Store
    {
        public function get($key) {}
        public function many(array $keys);
        public function put($key, $value, $minutes) {}
        public function putMany(array $values, $minutes);
        public function increment($key, $value = 1) {}
        public function decrement($key, $value = 1) {}
        public function forever($key, $value) {}
        public function forget($key) {}
        public function flush() {}
        public function getPrefix() {}
    }

これらのメソッドをMongoDB接続を用い、実装するだけです。各メソッドをどのように実装するかの例は、フレームワークの`Illuminate\Cache\MemcachedStore`のソースコードを参照してください。実装を完了したら、ドライバを登録します。

    Cache::extend('mongo', function ($app) {
        return Cache::repository(new MongoStore);
    });

> {tip} カスタムキャッシュドライバーをどこに設置するか迷っているなら、`app`ディレクトリ下に`Extensions`の名前空間で作成できます。しかし、Laravelはアプリケーション構造を強制していませんので、自分の好みに合わせてアプリケーションを自由に構築できることを忘れないでください。

<a name="registering-the-driver"></a>
### ドライバ登録

Laravelにカスタムキャッシュドライバを登録するには、`Cache`ファサードの`extend`メソッドを使います。新しくインストールしたLaravelに含まれている、デフォルトの`App\Providers\AppServiceProvider`の`boot`メソッドで、`Cache::extend`を呼び出せます。もしくは、拡張を設置するために自身のサービスプロバイダを作成することもできます。`config/app.php`プロバイダ配列に、そのプロバイダを登録し忘れないようにしてください。

    <?php

    namespace App\Providers;

    use App\Extensions\MongoStore;
    use Illuminate\Support\Facades\Cache;
    use Illuminate\Support\ServiceProvider;

    class CacheServiceProvider extends ServiceProvider
    {
        /**
         * サービス起動後の登録処理
         *
         * @return void
         */
        public function boot()
        {
            Cache::extend('mongo', function ($app) {
                return Cache::repository(new MongoStore);
            });
        }

        /**
         * コンテナへ結合を登録
         *
         * @return void
         */
        public function register()
        {
            //
        }
    }

`extend`メソッドの最初の引数はドライバ名です。これは`config/cache.php`設定ファイルの、`driver`オプションと対応します。第２引数は、`Illuminate\Cache\Repository`インスタンスを返すクロージャです。クロージャには、[サービスコンテナ](/docs/{{version}}/container)インスタンスの`$app`インスタンスが渡されます。

拡張を登録したら、`config/cache.php`設定ファイルの`driver`オプションへ、拡張の名前を登録してください。

<a name="events"></a>
## イベント

全キャッシュ操作に対してコードを実行するには、キャッシュが発行する[イベント](/docs/{{version}}/events)を購読する必要があります。通常、イベントリスナは`EventServiceProvider`の中へ設置します。

    /**
     * アプリケーションのイベントリスナ
     *
     * @var array
     */
    protected $listen = [
        'Illuminate\Cache\Events\CacheHit' => [
            'App\Listeners\LogCacheHit',
        ],

        'Illuminate\Cache\Events\CacheMissed' => [
            'App\Listeners\LogCacheMissed',
        ],

        'Illuminate\Cache\Events\KeyForgotten' => [
            'App\Listeners\LogKeyForgotten',
        ],

        'Illuminate\Cache\Events\KeyWritten' => [
            'App\Listeners\LogKeyWritten',
        ],
    ];
