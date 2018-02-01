# Redis

- [イントロダクション](#introduction)
    - [設定](#configuration)
    - [Predis](#predis)
    - [PhpRedis](#phpredis)
- [Redisの操作](#interacting-with-redis)
    - [パイプラインコマンド](#pipelining-commands)
- [publish／subscribe](#pubsub)

<a name="introduction"></a>
## イントロダクション

[Redis](https://redis.io)はオープンソースの進歩的なキー／値保存システムです。キーに[文字列](https://redis.io/topics/data-types#strings)、[ハッシュ](https://redis.io/topics/data-types#hashes)、[リスト](https://redis.io/topics/data-types#lists)、[セット](https://redis.io/topics/data-types#sets)、[ソート済みセット](https://redis.io/topics/data-types#sorted-sets)が使用できるため、データ構造サーバとしてよく名前が上がります。

LaravelでRedisを使用する前に、`predis/predis`パッケージをComposerでインストールする必要があります。

    composer require predis/predis

もしくは、[PhpRedis](https://github.com/phpredis/phpredis) PHP拡張をPECLでインストールすることもできます。この拡張のインストールはより複雑ですが、Redisを頻繁に使用するアプリケーションでは、よりパフォーマンスが良くなります。

<a name="configuration"></a>
### 設定

アプリケーションのRedis設定は`config/database.php`ファイルにあります。このファイルの中に`redis`配列があり、アプリケーションで使用されるRadisサーバの設定を含んでいます。

    'redis' => [

        'client' => 'predis',

        'default' => [
            'host' => env('REDIS_HOST', 'localhost'),
            'password' => env('REDIS_PASSWORD', null),
            'port' => env('REDIS_PORT', 6379),
            'database' => 0,
        ],

    ],

デフォルトのサーバ設定は、開発時には十分でしょう。しかしご自由に自分の環境に合わせてこの配列を変更してください。各Redisサーバは名前、ホスト、ポートの指定が必要です。

#### クラスタ設定

アプリケーションでRedisサーバのクラスタを使用している場合は、Redis設定の`clusters`キーで定義する必要があります。

    'redis' => [

        'client' => 'predis',

        'clusters' => [
            'default' => [
                [
                    'host' => env('REDIS_HOST', 'localhost'),
                    'password' => env('REDIS_PASSWORD', null),
                    'port' => env('REDIS_PORT', 6379),
                    'database' => 0,
                ],
            ],
        ],

    ],

デフォルトでクラスタはノード間のクライアントサイドシェアリングを実行し、ノードをプールし、利用可能な大きいRAMを作成できるようにします。しかしながら、クライアントサイドシェアリングはフェイルオーバーを処理しません。そのため、他のプライマリデータ保存域からのキャッシュデータを使用できるようにするのに適しています。ネイティブなRedisクラスタリングを使用したい場合は、Redis設置の`options`キーでこれを指定してください。

    'redis' => [

        'client' => 'predis',

        'options' => [
            'cluster' => 'redis',
        ],

        'clusters' => [
            // ...
        ],

    ],

<a name="predis"></a>
### Predis

デフォルトの`host`、`port`、`database`、`password`オプションに加え、Predisは各Redisサーバに対する[接続パラメータ](https://github.com/nrk/predis/wiki/Connection-Parameters)を定義する機能をサポートしています。これらの追加設定オプションを使うには、`config/database.php`設定ファイルのRedisサーバ設定へ追加してください。

    'default' => [
        'host' => env('REDIS_HOST', 'localhost'),
        'password' => env('REDIS_PASSWORD', null),
        'port' => env('REDIS_PORT', 6379),
        'database' => 0,
        'read_write_timeout' => 60,
    ],

<a name="phpredis"></a>
### PhpRedis

> {note} PhpRedis PHP拡張をPECL経由でインストールした場合は、`config/app.php`設定ファイル中の`Redis`エイリアスをリネームする必要があります。

PhpRedis拡張を使用するには、Redis設定の`client`オプションを`phpredis`へ変更してください。このオプションは`config/database.php`設定ファイルにあります。

    'redis' => [

        'client' => 'phpredis',

        // 残りのRedis設定…
    ],

デフォルトの`host`、`port`、`database`、`password`オプションに加え、PhpRedisは`persistent`、`prefix`、`read_timeout`、`timeout`追加オプションをサポートしています。`config/database.php`設定ファイル中のRedisサーバ設定に、これらのオプションを追加してください。

    'default' => [
        'host' => env('REDIS_HOST', 'localhost'),
        'password' => env('REDIS_PASSWORD', null),
        'port' => env('REDIS_PORT', 6379),
        'database' => 0,
        'read_timeout' => 60,
    ],

<a name="interacting-with-redis"></a>
## Redisの操作

`Redis`[ファサード](/docs/{{version}}/facades)のバラエティー豊かなメソッドを呼び出し、Redisを操作できます。`Redis`ファサードは動的メソッドをサポートしています。つまりファサードでどんな[Redisコマンド](https://redis.io/commands)でも呼び出すことができ、そのコマンドは直接Redisへ渡されます。以下の例ではRedisの`GET`コマンドを`Redis`ファサードの`get`メソッドで呼び出しています。

    <?php

    namespace App\Http\Controllers;

    use App\Http\Controllers\Controller;
    use Illuminate\Support\Facades\Redis;

    class UserController extends Controller
    {
        /**
         * 指定ユーザーのプロフィール表示
         *
         * @param  int  $id
         * @return Response
         */
        public function showProfile($id)
        {
            $user = Redis::get('user:profile:'.$id);

            return view('user.profile', ['user' => $user]);
        }
    }

もちろん前記の通り、`Redis`ファサードでどんなRedisコマンドでも呼び出すことができます。Laravelはmagicメソッドを使いコマンドをRedisサーバへ送りますので、Redisコマンドで期待されている引数を渡してください。

    Redis::set('name', 'Taylor');

    $values = Redis::lrange('names', 5, 10);

サーバにコマンドを送る別の方法は`command`メソッドを使う方法です。最初の引数にコマンド名、第２引数に値の配列を渡します。

    $values = Redis::command('lrange', ['name', 5, 10]);

#### 複数のRedis接続の使用

Redisインスタンスを`Redis::connection`メソッドの呼び出しで取得できます。

    $redis = Redis::connection();

これによりデフォルトのRedisサーバのインスタンスが取得できます。さらに、Redis設定で定義した、特定のサーバやクラスタを取得するために、`connection`メソッドへ接続名やクラスタ名を渡すこともできます。

    $redis = Redis::connection('my-connection');

<a name="pipelining-commands"></a>
### パイプラインコマンド

一回の操作でサーバに対し多くのコマンドを送る必要がある場合はパイプラインを使うべきでしょう。`pipeline`メソッドは引数をひとつだけ取り、Redisインスタンスを取る「クロージャ」です。このRedisインスタンスで全コマンドを発行し、一回の操作で全部実行できます。

    Redis::pipeline(function ($pipe) {
        for ($i = 0; $i < 1000; $i++) {
            $pipe->set("key:$i", $i);
        }
    });

<a name="pubsub"></a>
## publish／subscribe

さらにLaravelは、Redisの`publish`と`subscribe`コマンドの便利なインターフェイスも提供しています。これらのRedisコマンドは、指定した「チャンネル」のメッセージをリッスンできるようにしてくれます。他のアプリケーションからこのチャンネルにメッセージを公開するか、他の言語を使うこともでき、これによりアプリケーション／プロセス間で簡単に通信できます。

最初に`subscribe`メソッドでRedisを経由するチャンネルのリスナを準備します。`subscribe`メソッドは長時間動作するプロセスですので、このメソッドは[Artisanコマンド](/docs/{{version}}/artisan)の中で呼び出します。

    <?php

    namespace App\Console\Commands;

    use Illuminate\Console\Command;
    use Illuminate\Support\Facades\Redis;

    class RedisSubscribe extends Command
    {
        /**
         * コンソールコマンドの名前と使用法
         *
         * @var string
         */
        protected $signature = 'redis:subscribe';

        /**
         * コンソールコマンドの説明
         *
         * @var string
         */
        protected $description = 'Subscribe to a Redis channel';

        /**
         * コンソールコマンドの実行
         *
         * @return mixed
         */
        public function handle()
        {
            Redis::subscribe(['test-channel'], function ($message) {
                echo $message;
            });
        }
    }

これで`publish`メソッドを使いチャンネルへメッセージを公開できます。

    Route::get('publish', function () {
        // Route logic...

        Redis::publish('test-channel', json_encode(['foo' => 'bar']));
    });

#### ワイルドカード購入

`psubscribe`メソッドでワイルドカードチャネルに対し購入できます。全チャンネルの全メッセージを補足するために便利です。`$channel`名は指定するコールバック「クロージャ」の第２引数として渡されます。

    Redis::psubscribe(['*'], function ($message, $channel) {
        echo $message;
    });

    Redis::psubscribe(['users.*'], function ($message, $channel) {
        echo $message;
    });
