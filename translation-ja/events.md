# イベント

- [イントロダクション](#introduction)
- [イベント／リスナ登録](#registering-events-and-listeners)
    - [イベント／リスナ生成](#generating-events-and-listeners)
    - [任意のイベント登録](#manually-registering-events)
- [イベント定義](#defining-events)
- [リスナ定義](#defining-listeners)
- [イベントリスナのキュー投入](#queued-event-listeners)
    - [キューへの任意アクセス](#manually-accessing-the-queue)
    - [失敗したジョブの取り扱い](#handling-failed-jobs)
- [イベントの発行](#dispatching-events)
- [イベント購読](#event-subscribers)
    - [イベント購読プログラミング](#writing-event-subscribers)
    - [イベント購読登録](#registering-event-subscribers)

<a name="introduction"></a>
## イントロダクション

Laravelのイベントはシンプルなオブザーバの実装で、アプリケーションで発生する様々なイベントを購読し、リッスンするために使用します。イベントクラスは通常、`app/Events`ディレクトリに保存されます。一方、リスナは`app/Listeners`ディレクトリへ保存されます。アプリケーションに両ディレクトリが存在しなくても、心配ありません。Artisanコンソールコマンドを使い、イベントとリスナを生成するとき、ディレクトリも生成されます。

一つのイベントは、互いに依存していない複数のリスナに紐付けられますので、アプリケーションの様々な要素を独立させるための良い手段として活用できます。たとえば、注文を配送するごとにSlack通知をユーザーへ届けたいとします。注文の処理コードとSlackの通知コードを結合する代わりに、`OrderShipped`イベントを発行し、リスナがそれを受け取り、Slack通知へ変換するように実装できます。

<a name="registering-events-and-listeners"></a>
## イベント／リスナ登録

Laravelアプリケーションに含まれている`EventServiceProvider`は、イベントリスナを全て登録するために便利な場所を提供しています。`listen`プロパティは全イベント（キー）とリスナ（値）で構成されている配列です。もちろん、アプリケーションで必要とされているイベントをこの配列に好きなだけ追加できます。たとえば`OrderShipped`イベントを追加してみましょう。

    /**
     * アプリケーションのイベントリスナをマップ
     *
     * @var array
     */
    protected $listen = [
        'App\Events\OrderShipped' => [
            'App\Listeners\SendShipmentNotification',
        ],
    ];

<a name="generating-events-and-listeners"></a>
### イベント／リスナ生成

毎回ハンドラやリスナを作成するのは、当然のことながら手間がかかります。代わりにハンドラとリスナを`EventServiceProvider`に追加し、`event:generate`コマンドを使いましょう。このコマンドは`EventServiceProvider`にリストしてあるイベントやリスナを生成してくれます。既存のイベントとハンドラには、もちろん変更を加えません。

    php artisan event:generate

<a name="manually-registering-events"></a>
### イベントの手動登録

通常イベントは、`EventServiceProvider`の`$listen`配列により登録するべきです。しかし、`EventServiceProvider`の`boot`メソッドの中で、クロージャベースリスナを登録することができます。

    /**
     * アプリケーションの他のイベントを登録する
     *
     * @return void
     */
    public function boot()
    {
        parent::boot();

        Event::listen('event.name', function ($foo, $bar) {
            //
        });
    }

#### ワイルドカードリスナ

登録したリスナが、`*`をワイルドカードパラメータとして使用している場合、同じリスナで複数のイベントを捕捉できます。ワイルドカードリスナは、イベント全体のデータ配列を最初の引数として、イベントデータ全体を第２引数として受け取ります。

    Event::listen('event.*', function ($eventName, array $data) {
        //
    });

<a name="defining-events"></a>
## イベント定義

イベントクラスはデータコンテナとして、イベントに関する情報を保持します。たとえば生成した`OrderShipped`イベントが[Eloquent ORM](/docs/{{version}}/eloquent)オブジェクトを受け取るとしましょう。

    <?php

    namespace App\Events;

    use App\Order;
    use Illuminate\Queue\SerializesModels;

    class OrderShipped
    {
        use SerializesModels;

        public $order;

        /**
         * 新しいイベントインスタンスの生成
         *
         * @param  Order  $order
         * @return void
         */
        public function __construct(Order $order)
        {
            $this->order = $order;
        }
    }

ご覧の通り、このクラスはロジックを含みません。購入された`Order`オブジェクトのための、コンテナです。イベントオブジェクトがPHPの`serialize`関数でシリアライズされる場合でも、Eloquentモデルをイベントがuseしている`SerializesModels`トレイトが優雅にシリアライズします。

<a name="defining-listeners"></a>
## リスナの定義

次にサンプルイベントのリスナを取り上げましょう。イベントリスナはイベントインスタンスを`handle`メソッドで受け取ります。`event:generate`コマンドは自動的に適切なイベントクラスをインポートし、`handle`メソッドのイベントのタイプヒントを指定します。そのイベントに対応するために必要なロジックを`handle`メソッドで実行してください。

    <?php

    namespace App\Listeners;

    use App\Events\OrderShipped;

    class SendShipmentNotification
    {
        /**
         * イベントリスナ生成
         *
         * @return void
         */
        public function __construct()
        {
            //
        }

        /**
         * イベントの処理
         *
         * @param  OrderShipped  $event
         * @return void
         */
        public function handle(OrderShipped $event)
        {
            // $event->orderにより、注文へアクセス…
        }
    }

> {tip} イベントリスナでも、必要な依存をコンストラクターのタイプヒントで指定できます。イベントリスナは全てLaravelの[サービスコンテナで](/docs/{{version}}/container)依存解決されるので、依存は自動的に注入されます。

#### イベントの伝播の停止

場合によりイベントが他のリスナへ伝播されるのを止めたいこともあります。その場合は`handle`メソッドから`false`を返してください。

<a name="queued-event-listeners"></a>
## イベントリスナのキュー投入

メール送信やHTTPリクエストを作成するなど、遅い仕事を担当する場合、そのリスナをキューイングできると便利です。キューリスナに取り掛かる前に、[キューの設定](/docs/{{version}}/queues)を確実に行い、サーバかローカル開発環境でキューリスナを起動しておいてください。

リスナをキュー投入するように指定するには、`ShouldQueue`インターフェイスをリスナクラスに追加します。`event:generate` Artisanコマンドにより生成したリスナには、既にこのインターフェイスが現在の名前空間下にインポートされていますので、すぐに使用できます。

    <?php

    namespace App\Listeners;

    use App\Events\OrderShipped;
    use Illuminate\Contracts\Queue\ShouldQueue;

    class SendShipmentNotification implements ShouldQueue
    {
        //
    }

これだけです！これでこのリスナがイベントのために呼び出されると、Laravelの[キューシステム](/docs/{{version}}/queues)を使い、イベントデスパッチャーにより自動的にキューへ投入されます。キューにより実行されるリスナから例外が投げられなければ、そのキュージョブは処理が済み次第、自動的に削除されます。

#### キュー接続とキュー名のカスタマイズ

イベントリスナのキュー接続とキュー名をカスタマイズしたい場合は、`$connection`と`$queue`プロパティをリスナクラスで定義します。

    <?php

    namespace App\Listeners;

    use App\Events\OrderShipped;
    use Illuminate\Contracts\Queue\ShouldQueue;

    class SendShipmentNotification implements ShouldQueue
    {
        /**
         * ジョブを投入する接続名
         *
         * @var string|null
         */
        public $connection = 'sqs';

        /**
         * ジョブを投入するキュー名
         *
         * @var string|null
         */
        public $queue = 'listeners';
    }

<a name="manually-accessing-the-queue"></a>
### キューへの任意アクセス

リスナの裏で動作しているキュージョブの、`delete`や`release`メソッドを直接呼び出したければ、`Illuminate\Queue\InteractsWithQueue`トレイトを使えます。このトレイトは生成されたリスナにはデフォルトとしてインポートされており、これらのメソッドへアクセスできるようになっています。

    <?php

    namespace App\Listeners;

    use App\Events\OrderShipped;
    use Illuminate\Queue\InteractsWithQueue;
    use Illuminate\Contracts\Queue\ShouldQueue;

    class SendShipmentNotification implements ShouldQueue
    {
        use InteractsWithQueue;

        /**
         * イベントの処理
         *
         * @param  \App\Events\OrderShipped  $event
         * @return void
         */
        public function handle(OrderShipped $event)
        {
            if (true) {
                $this->release(30);
            }
        }
    }

<a name="handling-failed-jobs"></a>
### 失敗したジョブの取り扱い

時々、キュー投入したイベントリスナが落ちることがあります。キューワーカにより定義された最大試行回数を超え、キュー済みのリスナが実行されると、リスナの`failed`メソッドが実行されます。`failed`メソッドはイベントインスタンスと落ちた原因の例外を引数に受け取ります。

    <?php

    namespace App\Listeners;

    use App\Events\OrderShipped;
    use Illuminate\Queue\InteractsWithQueue;
    use Illuminate\Contracts\Queue\ShouldQueue;

    class SendShipmentNotification implements ShouldQueue
    {
        use InteractsWithQueue;

        /**
         * イベントの処理
         *
         * @param  \App\Events\OrderShipped  $event
         * @return void
         */
        public function handle(OrderShipped $event)
        {
            //
        }

        /**
         * 失敗したジョブの処理
         *
         * @param  \App\Events\OrderShipped  $event
         * @param  \Exception  $exception
         * @return void
         */
        public function failed(OrderShipped $event, $exception)
        {
            //
        }
    }

<a name="dispatching-events"></a>
## イベントの発行

イベントを発行するには、`event`ヘルパにイベントのインスタンスを渡してください。このヘルパは登録済みのリスナ全てに、イベントをディスパッチします。`event`ヘルパはグローバルに使用できますので、アプリケーションのどこからでも呼び出すことができます。

    <?php

    namespace App\Http\Controllers;

    use App\Order;
    use App\Events\OrderShipped;
    use App\Http\Controllers\Controller;

    class OrderController extends Controller
    {
        /**
         * 指定した注文を発送
         *
         * @param  int  $orderId
         * @return Response
         */
        public function ship($orderId)
        {
            $order = Order::findOrFail($orderId);

            // 注文発送ロジック…

            event(new OrderShipped($order));
        }
    }

> {tip} テスト時は実際にリスナを起動せずに、正しいイベントがディスパッチされたことをアサートできると便利です。Laravelに[組み込まれたテストヘルパ](/docs/{{version}}/mocking#event-fake)で簡単に行なえます。

<a name="event-subscribers"></a>
## イベント

<a name="writing-event-subscribers"></a>
### イベント購読プログラミング

イベント購読クラスは、その内部で複数のイベントを購読でき、一つのクラスで複数のイベントハンドラを定義できます。購読クラスは、イベントディスパッチャインスタンスを受け取る、`subscribe`メソッドを定義する必要があります。イベントリスナを登録するには、渡されたディスパッチャの`listen`メソッドを呼び出します。

    <?php

    namespace App\Listeners;

    class UserEventSubscriber
    {
        /**
         * ユーザーログインイベント処理
         */
        public function onUserLogin($event) {}

        /**
         * ユーザーログアウトイベント処理
         */
        public function onUserLogout($event) {}

        /**
         * 購読するリスナの登録
         *
         * @param  Illuminate\Events\Dispatcher  $events
         */
        public function subscribe($events)
        {
            $events->listen(
                'Illuminate\Auth\Events\Login',
                'App\Listeners\UserEventSubscriber@onUserLogin'
            );

            $events->listen(
                'Illuminate\Auth\Events\Logout',
                'App\Listeners\UserEventSubscriber@onUserLogout'
            );
        }

    }

<a name="registering-event-subscribers"></a>
### イベント購読登録

購読クラスを書いたら、イベントディスパッチャへ登録できる準備が整いました。`EventServiceProvider`の`$subscribe`プロパティを使用し、購読クラスを登録します。例として、`UserEventSubscriber`をリストに追加してみましょう。

    <?php

    namespace App\Providers;

    use Illuminate\Foundation\Support\Providers\EventServiceProvider as ServiceProvider;

    class EventServiceProvider extends ServiceProvider
    {
        /**
         * アプリケーションのイベントリスナをマップ
         *
         * @var array
         */
        protected $listen = [
            //
        ];

        /**
         * 登録する購読クラス
         *
         * @var array
         */
        protected $subscribe = [
            'App\Listeners\UserEventSubscriber',
        ];
    }
