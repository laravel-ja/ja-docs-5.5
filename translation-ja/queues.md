# キュー

- [イントロダクション](#introduction)
    - [接続 Vs. キュー](#connections-vs-queues)
    - [ドライバ事前要件](#driver-prerequisites)
- [ジョブの作成](#creating-jobs)
    - [ジョブクラスの生成](#generating-job-classes)
    - [クラス構成](#class-structure)
- [ジョブのディスパッチ](#dispatching-jobs)
    - [遅延ディスパッチ](#delayed-dispatching)
    - [ジョブのチェーン](#job-chaining)
    - [キューと接続のカスタマイズ](#customizing-the-queue-and-connection)
    - [最大試行回数／タイムアウト値の指定](#max-job-attempts-and-timeout)
    - [レート制限](#rate-limiting)
    - [エラー処理](#error-handling)
- [キューワーカの実行](#running-the-queue-worker)
    - [キュープライオリティ](#queue-priorities)
    - [キューワーカとデプロイ](#queue-workers-and-deployment)
    - [ジョブの期限切れとタイムアウト](#job-expirations-and-timeouts)
- [Supervisor設定](#supervisor-configuration)
- [失敗したジョブの処理](#dealing-with-failed-jobs)
    - [ジョブ失敗後のクリーンアップ](#cleaning-up-after-failed-jobs)
    - [ジョブ失敗イベント](#failed-job-events)
    - [失敗したジョブの再試行](#retrying-failed-jobs)
- [ジョブイベント](#job-events)

<a name="introduction"></a>
## イントロダクション

> {tip} 現在、LaravelはRedisで動作するキューのための美しいダッシュボードと設定システムを備えたHorizonを提供しています。詳細は、[Horizonのドキュメント](/docs/{{version}}/horizon)で確認してください。

Laravelのキューサービスは、Beanstalk、Amazon SQS、Redis、さらにはリレーショナル・データベースなど様々なキューバックエンドに対し共通のAPIを提供しています。キューによりメール送信のような時間を費やす処理を遅らせることが可能です。時間のかかるタスクを遅らせることで、よりアプリケーションのリクエストをドラマチックにスピードアップできます。

キューの設定ファイルは`config/queue.php`です。このファイルにはフレームワークに含まれているそれぞれのドライバーへの接続設定が含まれています。それにはデータベース、[Beanstalkd](https://kr.github.com/beanstalkd)、[Amazon SQS](https://aws.amazon.com/sqs)、[Redis](https://redis.io)、ジョブが即時に実行される同期（ローカル用途）ドライバーが含まれています。 `null`キュードライバはキューされたジョブが実行されないように、破棄します。

<a name="connections-vs-queues"></a>
### 接続 Vs. キュー

Laravelのキューに取り掛かる前に、「接続」と「キュー」の区別を理解しておくことが重要です。`config/queue.php`設定ファイルの中には、`connections`設定オプションがあります。このオプションはAmazon SQS、Beanstalk、Redisなどのバックエンドサービスへの個々の接続を定義します。しかし、どんな指定されたキュー接続も、複数の「キュー」を持つことができます。「キュー」とはキュー済みのジョブのスタック、もしくは積み重ねのことです。

`queue`接続ファイルの`queue`属性を含んでいる、各接続設定例に注目してください。ジョブがディスパッチされ、指定された接続へ送られた時にのデフォルトキューです。言い換えれば、どのキューへディスパッチするのか明確に定義していないジョブをディスパッチすると、そのジョブは接続設定の`queue`属性で定義したキューへ送られます。

    // このジョブはデフォルトキューへ送られる
    Job::dispatch();

    // このジョブは"emails"キューへ送られる
    Job::dispatch()->onQueue('emails');

あるアプリケーションでは複数のキューへジョブを送る必要はなく、代わりに１つのシンプルなキューが適しているでしょう。しかし、複数のキューへジョブを送ることは、優先順位づけしたい、もしくはジョブの処理を分割したいアプリケーションでは、特に便利です。Laravelのキューワーカはプライオリティによりどのキューで処理するかを指定できるからです。たとえば、ジョブを`high`キューへ送れば、より高い処理プライオリティのワーカを実行できます。

    php artisan queue:work --queue=high,default

<a name="driver-prerequisites"></a>
### ドライバ毎の必要要件

#### データベース

`database`キュードライバを使用するには、ジョブを記録するためのデータベーステーブルが必要です。このテーブルを作成するマイグレーションは`queue:table` Artisanコマンドにより生成できます。マイグレーションが生成されたら、`migrate`コマンドでデータベースをマイグレートしてください。

    php artisan queue:table

    php artisan migrate

#### Redis

`redis`キュードライバーを使用するには、`config/database.php`設定ファイルでRedisのデータベースを設定する必要があります。

Redisキュー接続でRedisクラスタを使用している場合は、キュー名に[キーハッシュタグ](https://redis.io/topics/cluster-spec#keys-hash-tags)を含める必要があります。これはキューに指定した全Redisキーが同じハッシュスロットに確実に置かれるようにするためです。

    'redis' => [
        'driver' => 'redis',
        'connection' => 'default',
        'queue' => '{default}',
        'retry_after' => 90,
    ],

#### 他のドライバの要件

以下の依存パッケージがリストしたキュードライバを使用するために必要です。

<div class="content-list" markdown="1">
- Amazon SQS: `aws/aws-sdk-php ~3.0`
- Beanstalkd: `pda/pheanstalk ~3.0`
- Redis: `predis/predis ~1.0`
</div>

<a name="creating-jobs"></a>
## ジョブの作成

<a name="generating-job-classes"></a>
### ジョブクラスの生成

キュー投入可能なアプリケーションの全ジョブは、デフォルトで`app/Jobs`ディレクトリへ保存されます。`app/Jobs`ディレクトリが存在しなくても、`make:job` Artisanコマンドの実行時に生成されます。新しいキュージョブをArtisan CLIで生成できます。

    php artisan make:job ProcessPodcast

非同期で実行するため、ジョブをキューへ投入することをLaravelへ知らせる、`Illuminate\Contracts\Queue\ShouldQueue`インターフェイスが生成されたクラスには実装されます。

<a name="class-structure"></a>
### クラス構成

ジョブクラスは通常とてもシンプルで、キューによりジョブが処理される時に呼び出される、`handle`メソッドのみで構成されています。手始めに、ジョブクラスのサンプルを見てみましょう。この例は、ポッドキャストの公開サービスを管理し、公開前にアップロードしたポッドキャストファイルを処理する必要があるという仮定です。

    <?php

    namespace App\Jobs;

    use App\Podcast;
    use App\AudioProcessor;
    use Illuminate\Bus\Queueable;
    use Illuminate\Queue\SerializesModels;
    use Illuminate\Queue\InteractsWithQueue;
    use Illuminate\Contracts\Queue\ShouldQueue;
    use Illuminate\Foundation\Bus\Dispatchable;

    class ProcessPodcast implements ShouldQueue
    {
        use Dispatchable, InteractsWithQueue, Queueable, SerializesModels;

        protected $podcast;

        /**
         * 新しいジョブインスタンスの生成
         *
         * @param  Podcast  $podcast
         * @return void
         */
        public function __construct(Podcast $podcast)
        {
            $this->podcast = $podcast;
        }

        /**
         * ジョブの実行
         *
         * @param  AudioProcessor  $processor
         * @return void
         */
        public function handle(AudioProcessor $processor)
        {
            // アップロード済みポッドキャストの処理…
        }
    }

この例中、キュージョブのコンテナーに直接[Eloquentモデル](/docs/{{version}}/eloquent)が渡せることに注目してください。ジョブが使用している`SerializesModels`トレイトによりEloquentモデルは優雅にシリアライズされ、ジョブが処理される時にアンシリアライズされます。キュー投入されたジョブがコンテナでEloquentモデルを受け取ると、モデルの識別子のみシリアライズされています。ジョブが実際に処理される時、キューシステムは自動的にデータベースから完全なモデルインスタンスを再取得します。これらは全てアプリケーションの完全な透過性のためであり、Eloquentモデルインスタンスをシリアライズするときに発生する問題を防ぐことができます。

`handle`メソッドはキューによりジョブが処理されるときに呼びだされます。ジョブの`handle`メソッドにタイプヒントにより依存を指定できることに注目してください。Laravelの[サービスコンテナ](/docs/{{version}}/container)が自動的に依存を注入します。

> {note} Rawイメージコンテンツのようなバイナリデータは、キュージョブへ渡す前に、`base64_encode`関数を通してください。そうしないと、そのジョブはキューへ設置する前にJSONへ正しくシリアライズされません。

<a name="dispatching-jobs"></a>
## ジョブのディスパッチ

ジョブクラスを書き上げたら、ジョブクラス自身の`dispatch`メソッドを使い、ディスパッチできます。`dispatch`メソッドへ渡す引数は、ジョブのコンストラクタへ渡されます。

    <?php

    namespace App\Http\Controllers;

    use App\Jobs\ProcessPodcast;
    use Illuminate\Http\Request;
    use App\Http\Controllers\Controller;

    class PodcastController extends Controller
    {
        /**
         * 新ポッドキャストの保存
         *
         * @param  Request  $request
         * @return Response
         */
        public function store(Request $request)
        {
            // ポッドキャスト作成…

            ProcessPodcast::dispatch($podcast);
        }
    }

<a name="delayed-dispatching"></a>
### 遅延ディスパッチ

キュー投入されたジョブの実行を遅らせたい場合は、ジョブのディスパッチ時に`delay`メソッドを使います。例として、ディスパッチ後１０分経つまでは、処理が行われないジョブを指定してみましょう。

    <?php

    namespace App\Http\Controllers;

    use App\Jobs\ProcessPodcast;
    use Illuminate\Http\Request;
    use App\Http\Controllers\Controller;

    class PodcastController extends Controller
    {
        /**
         * 新ポッドキャストの保存
         *
         * @param  Request  $request
         * @return Response
         */
        public function store(Request $request)
        {
            // ポッドキャスト作成…

            ProcessPodcast::dispatch($podcast)
                    ->delay(now()->addMinutes(10));
        }
    }

> {note} Amazon SQSキューサービスは、最大１５分の遅延時間です。

<a name="job-chaining"></a>
### ジョブチェーン

連続して実行する必要がある、キュー投入ジョブのリストをジョブチェーンで指定できます。一連のジョブの内、あるジョブが失敗すると、残りのジョブは実行されません。キュー投入ジョブチェーンを実行するには、dispatchableジョブどれかに対し、`withChain`メソッドを使用します。

    ProcessPodcast::withChain([
        new OptimizePodcast,
        new ReleasePodcast
    ])->dispatch();

<a name="customizing-the-queue-and-connection"></a>
### キューと接続のカスタマイズ

#### 特定キューへのディスパッチ

ジョブを異なるキューへ投入することで「カテゴライズ」できますし、様々なキューにいくつのワーカを割り当てるかと言うプライオリティ付けもできます。これはキー設定ファイルで定義した、別々のキュー「接続」へのジョブ投入を意味してはいないことに気をつけてください。一つの接続内の複数のキューを指定する方法です。

    <?php

    namespace App\Http\Controllers;

    use App\Jobs\ProcessPodcast;
    use Illuminate\Http\Request;
    use App\Http\Controllers\Controller;

    class PodcastController extends Controller
    {
        /**
         * 新ポッドキャストの保存
         *
         * @param  Request  $request
         * @return Response
         */
        public function store(Request $request)
        {
            // ポッドキャスト作成…

            ProcessPodcast::dispatch($podcast)->onQueue('processing');
        }
    }

#### 特定の接続へのディスパッチ

複数のキュー接続を利用するなら、ジョブを投入するキューを指定できます。ジョブをディスパッチする時に、`onConnection`メソッドで接続を指定します。

    <?php

    namespace App\Http\Controllers;

    use App\Jobs\ProcessPodcast;
    use Illuminate\Http\Request;
    use App\Http\Controllers\Controller;

    class PodcastController extends Controller
    {
        /**
         * 新ポッドキャストの保存
         *
         * @param  Request  $request
         * @return Response
         */
        public function store(Request $request)
        {
            // ポッドキャスト作成…

            ProcessPodcast::dispatch($podcast)->onConnection('sqs');
        }
    }

もちろん、ジョブを投入する接続とキューを指定するために、`onConnection`と`onQueue`メソッドをチェーンすることもできます。

    ProcessPodcast::dispatch($podcast)
                  ->onConnection('sqs')
                  ->onQueue('processing');

<a name="max-job-attempts-and-timeout"></a>
### 最大試行回数／タイムアウト値の指定

#### 最大試行回数

ジョブが試行する最大回数を指定するアプローチの一つは、Artisanコマンドラインへ`--tries`スイッチ使う方法です。

    php artisan queue:work --tries=3

しかし、より粒度の高いアプローチは、ジョブクラス自身に最大試行回数を定義する方法です。これはコマンドラインで指定された値より、優先度が高くなっています。

    <?php

    namespace App\Jobs;

    class ProcessPodcast implements ShouldQueue
    {
        /**
         * 最大試行回数
         *
         * @var int
         */
        public $tries = 5;
    }

<a name="time-based-attempts"></a>
#### 時間ベースの試行

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

#### タイムアウト

> {note} タイムアウト機能はバージョン7.1以上のPHPと、`pcntl` PHP拡張に対し、オプティマイズされています。

同様に、ジョブの最大実行秒数を指定するために、Artisanコマンドラインに`--timeout`スイッチを指定することができます。

    php artisan queue:work --timeout=30

しかしながら、最大実行秒数をジョブクラス自身に定義することもできます。ジョブにタイムアウト時間を指定すると、コマンドラインに指定されたタイムアウトよりも優先されます。

    <?php

    namespace App\Jobs;

    class ProcessPodcast implements ShouldQueue
    {
        /**
         * ジョブがタイムアウトになるまでの秒数
         *
         * @var int
         */
        public $timeout = 120;
    }

<a name="rate-limiting"></a>
### レート制限

> {note} この機能が動作するには、アプリケーションで[Redisサーバ](/docs/{{version}}/redis)が利用できる必要があります。

アプリケーションでRedisを利用しているなら、時間と回数により、キュージョブを制限できます。この機能は、キュージョブがレート制限のあるAPIに関連している場合に役立ちます。`throttle`メソッドの使用例として、指定したジョブタイプを６０秒毎に１０回だけ実行できるように制限しましょう。ロックできなかった場合、後で再試行できるように、通常はジョブをキューへ戻す必要があります。

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

> {tip} レート制限を使用する場合、実行を成功するまでに必要な試行回数を決めるのは、難しくなります。そのため、レート制限は[時間ベースの試行](#time-based-attempts)と組み合わせるのが便利です。

<a name="error-handling"></a>
### エラー処理

ジョブの処理中に例外が投げられると、ジョブは自動的にキューへ戻され、再試行されます。ジョブはアプリケーションが許している最大試行回数に達するまで、連続して実行されます。最大試行回数は`queue:work` Artisanコマンドへ`--tries`スイッチを使い定義されます。もしくは、ジョブクラス自身に最大試行回数を定義することもできます。キューワーカの実行についての情報は、[以降](#running-the-queue-worker)で説明します。

<a name="running-the-queue-worker"></a>
## キューワーカの実行

Laravelには、キューに投入された新しいジョブを処理する、キューワーカも含まれています。`queue:work` Artisanコマンドを使いワーカを実行できます。`queue:work`コマンドが起動したら、皆さんが停止するか、ターミナルを閉じるまで実行し続けることに注意してください。

    php artisan queue:work

> {tip} バックグランドで`queue:work`プロセスを永続的に実行し続けるには、キューワーカが止まらずに実行し続けていることを確実にするため、[Supervisor](#supervisor-configuration)のようなプロセスモニタを利用する必要があります。

キューワーカは長時間起動するプロセスで、起動した状態のままメモリに保存されることを覚えておいてください。その結果、一度起動したら、コードベースの変更は反映されません。そのため、開発期間中は[キューワーカを再起動](#queue-workers-and-deployment)することを忘れないでください。

#### ジョブを一つ処理する

`--once`オプションは、ワーカにキュー中のジョブをひとつだけ処理するように指示します。

    php artisan queue:work --once

#### 接続とキューの指定

どのキュー接続をワーカが使用するのかを指定できます。`work`コマンドで指定する接続名は、`config/queue.php`設定ファイルで定義されている接続と対応します。

    php artisan queue:work redis

指定した接続の特定のキューだけを処理するように、さらにキューワーカをカスタマイズすることもできます。たとえば、メールの処理をすべて、`redis`キュー接続の`emails`キューで処理する場合、以下のコマンドで単一のキューの処理だけを行うワーカを起動できます。

    php artisan queue:work redis --queue=emails

#### リソースの考察

デーモンキューワーカは各ジョブを処理する前に、フレームワークを「再起動」しません。そのため、各ジョブが終了したら、大きなリソースを開放してください。たとえば、GDライブラリでイメージ処理を行ったら、終了前に`imagedestroy`により、メモリを開放してください。

<a name="queue-priorities"></a>
### キュープライオリティ

時々、キューをどのように処理するかをプライオリティ付けしたいことも起きます。たとえば、`config/queue.php`で`redis`接続のデフォルト`queue`を`low`に設定したとしましょう。しかし、あるジョブを`high`プライオリティでキューへ投入したい場合です。

    dispatch((new Job)->onQueue('high'));

`low`キュー上のジョブの処理が継続される前に、全`high`キュージョブが処理されることを確実にするには、`work`コマンドのキュー名にコンマ区切りのリストで指定してください。

    php artisan queue:work --queue=high,low

<a name="queue-workers-and-deployment"></a>
### キューワーカとデプロイ

キューワーカは長時間起動プロセスであるため、リスタートしない限りコードの変更を反映しません。ですから、キューワーカを使用しているアプリケーションをデプロイする一番シンプルな方法は、デプロイ処理の間、ワーカをリスタートすることです。`queue:restart`コマンドを実行することで、全ワーカを穏やかに再起動できます。

    php artisan queue:restart

このコマンドは存在しているジョブが失われないように、現在のジョブの処理が終了した後に、全キューワーカーへ穏やかに「終了する(die)」よう指示します。キューワーカは`queue:restart`コマンドが実行されると、終了するわけですから、キュージョブを自動的に再起動する、Supervisorのようなプロセスマネージャーを実行すべきでしょう。

> {tip} このコマンドはリスタートシグナルを保存するために、[キャッシュ](/docs/{{version}}/cache)を使用します。そのため、この機能を使用する前に、アプリケーションのキャッシュドライバーが、正しく設定されていることを確認してください。

<a name="job-expirations-and-timeouts"></a>
### ジョブの期限切れとタイムアウト

#### ジョブの有効期限

`config/queue.php`設定ファイルの中で、各キュー接続は`retry_after`オプションを定義しています。このオプションは処理中のジョブを再試行するまで、キュー接続を何秒待つかを指定します。たとえば、`retry_after`の値が`90`であれば、そのジョブは９０秒の間に削除されることなく処理され続ければ、キューへ再投入されます。通常、`retry_after`値はジョブが処理を妥当に完了するまでかかるであろう秒数の最大値を指定します。

> {note} `retry_after`を含まない唯一の接続は、Amazon SQSです。SQSはAWSコンソールで管理する、[Default Visibility Timeout](https://docs.aws.amazon.com/AWSSimpleQueueService/latest/SQSDeveloperGuide/AboutVT.html)を元にリトライを行います。

#### ワーカタイムアウト

`queue:work` Artisanコマンドは`--timeout`オプションも提供しています。`--timeout`オプションはLaravelキューマスタプロセスが、ジョブを処理する子のキューワーカをKillするまでどのくらい待つかを指定します。外部のHTTP呼び出しの反応が無いなど様々な理由で、時より子のキュープロセスは「フリーズ」します。`--timeout`オプションは指定した実行時間を過ぎた、フリーズプロセスを取り除きます。

    php artisan queue:work --timeout=60

`retry_after`設定オプションと`--timeout` CLIオプションは異なります。しかし、確実にジョブを失わずに、一度だけ処理を完了できるよう共に働きます。

> {note} `--timeout`値は、最低でも数秒`retry_after`設定値よりも短くしてください。これにより、与えられたジョブを処理するワーカが、ジョブのリトライ前に確実にkillされます。`--timeout`オプションを`retry_after`設定値よりも長くすると、ジョブが２度実行されるでしょう。

#### ワーカスリープ時間

ジョブがキュー上に存在しているとき、ワーカは各ジョブ間にディレイを取らずに実行し続けます。`sleep`オプションは新しく処理するジョブが存在しない時に、どの程度「スリープ」するかを決めます。スリープ中、ワーカは新しいジョブを処理しません。ジョブはワーカが目を様した後に処理されます。

    php artisan queue:work --sleep=3

<a name="supervisor-configuration"></a>
## Supervisor設定

#### Supervisorのインストール

SupervisorはLinuxオペレーティングシステムのプロセスモニタで、`queue:work`プロセスが落ちると自動的に起動します。UbuntuにSupervisorをインストールするには、次のコマンドを使ってください。

    sudo apt-get install supervisor

> {tip} Supervisoの設定に圧倒されそうならば、Laravelプロジェクトのために、Supervisorを自動的にインストールし、設定する[Laravel Forge](https://forge.laravel.com)の使用を考慮してください。

#### Supervisorの設定

Supervisorの設定ファイルは、通常`/etc/supervisor/conf.d`ディレクトリに保存します。このディレクトリの中には、Supervisorにどのようにプロセスを監視するのか指示する設定ファイルを好きなだけ設置できます。たとえば、`laravel-worker.conf`ファイルを作成し、`queue:work`プロセスを起動、監視させてみましょう。

    [program:laravel-worker]
    process_name=%(program_name)s_%(process_num)02d
    command=php /home/forge/app.com/artisan queue:work sqs --sleep=3 --tries=3
    autostart=true
    autorestart=true
    user=forge
    numprocs=8
    redirect_stderr=true
    stdout_logfile=/home/forge/app.com/worker.log

この例の`numprocs`ディレクティブは、Supervisorに全部で８つのqueue:workプロセスを実行・監視し、落ちている時は自動的に再起動するように指示しています。もちろん`command`ディレクティブの`queue:work sqs`の部分を変更し、希望のキュー接続に合わせてください。

#### Supervisorの起動

設定ファイルができたら、Supervisorの設定を更新し起動するために以下のコマンドを実行してください。

    sudo supervisorctl reread

    sudo supervisorctl update

    sudo supervisorctl start laravel-worker:*

Supervisorの詳細情報は、[Supervisorドキュメント](http://supervisord.org/index.html)で確認してください。

<a name="dealing-with-failed-jobs"></a>
## 失敗したジョブの処理

時より、キューされたジョブは失敗します。心配ありません。物事は計画通りに進まないものです。Laravelではジョブを再試行する最大回数を指定できます。この回数試行すると、そのジョブは`failed_jobs`データベーステーブルに挿入されます。`failed_jobs`テーブルのマイグレーションを生成するには`queue:failed-table`コマンドを実行して下さい。

    php artisan queue:failed-table

    php artisan migrate

次に[キューワーカ](#running-the-queue-worker)の実行時、`queue:work`コマンドに`--tries`スイッチを付け、最大試行回数を指定します。`--tries`オプションに値を指定しないと、ジョブは無限に試行します。

    php artisan queue:work redis --tries=3

<a name="cleaning-up-after-failed-jobs"></a>
### ジョブ失敗後のクリーンアップ

失敗時にジョブ特定のクリーンアップを実行するため、ジョブクラスで`failed`メソッドを直接定義できます。これはユーザーに警告を送ったり、ジョブの実行アクションを巻き戻すために最適な場所です。`failed`メソッドには、そのジョブを落とすことになった例外（`Exception`）が渡されます。

    <?php

    namespace App\Jobs;

    use Exception;
    use App\Podcast;
    use App\AudioProcessor;
    use Illuminate\Bus\Queueable;
    use Illuminate\Queue\SerializesModels;
    use Illuminate\Queue\InteractsWithQueue;
    use Illuminate\Contracts\Queue\ShouldQueue;

    class ProcessPodcast implements ShouldQueue
    {
        use InteractsWithQueue, Queueable, SerializesModels;

        protected $podcast;

        /**
         * 新しいジョブインスタンスの生成
         *
         * @param  Podcast  $podcast
         * @return void
         */
        public function __construct(Podcast $podcast)
        {
            $this->podcast = $podcast;
        }

        /**
         * ジョブの実行
         *
         * @param  AudioProcessor  $processor
         * @return void
         */
        public function handle(AudioProcessor $processor)
        {
            // アップロード済みポッドキャストの処理…
        }

        /**
         * 失敗したジョブの処理
         *
         * @param  Exception  $exception
         * @return void
         */
        public function failed(Exception $exception)
        {
            // 失敗の通知をユーザーへ送るなど…
        }
    }

<a name="failed-job-events"></a>
### ジョブ失敗イベント

ジョブが失敗した時に呼び出されるイベントを登録したい場合、`Queue::failing`メソッドが使えます。このイベントはメールや[HipChat](https://www.hipchat.com)により、チームへ通知する良い機会になります。例として、Laravelに含まれている`AppServiceProvider`で、このイベントのコールバックを付け加えてみましょう。

    <?php

    namespace App\Providers;

    use Illuminate\Support\Facades\Queue;
    use Illuminate\Queue\Events\JobFailed;
    use Illuminate\Support\ServiceProvider;

    class AppServiceProvider extends ServiceProvider
    {
        /**
         * 全アプリケーションサービスの初期処理
         *
         * @return void
         */
        public function boot()
        {
            Queue::failing(function (JobFailed $event) {
                // $event->connectionName
                // $event->job
                // $event->exception
            });
        }

        /**
         * サービスプロバイダの登録
         *
         * @return void
         */
        public function register()
        {
            //
        }
    }

<a name="retrying-failed-jobs"></a>
### 失敗したジョブの再試行

`failed_jobs`データベーステーブルに挿入された、失敗したジョブを全部確認したい場合は`queue:failed` Artisanコマンドを利用します。

    php artisan queue:failed

`queue:failed`コマンドはジョブID、接続、キュー、失敗した時間をリスト表示します。失敗したジョブをジョブIDで指定することでリトライできます。たとえば、IDが`5`の失敗したジョブを再試行するため、以下のコマンドを実行します。

    php artisan queue:retry 5

失敗したジョブをすべて再試行するには、IDとして`all`を`queue:retry`コマンドへ指定し、実行してください。

    php artisan queue:retry all

失敗したジョブを削除する場合は、`queue:forget`コマンドを使います。

    php artisan queue:forget 5

失敗したジョブを全部削除するには、`queue:flush`コマンドを使います。

    php artisan queue:flush

<a name="job-events"></a>
## ジョブイベント

`Queue`[ファサード](/docs/{{version}}/facades)に`before`と`after`メソッドを使い、キューされたジョブの実行前後に実行する、コールバックを指定できます。これらのコールバックはログを追加したり、ダッシュボードの状態を増加させたりするための機会を与えます。通常、これらのメソッドは[サービスプロバイダ](/docs/{{version}}/providers)から呼び出します。たとえば、Laravelに含まれる`AppServiceProvider`を使っていましょう。

    <?php

    namespace App\Providers;

    use Illuminate\Support\Facades\Queue;
    use Illuminate\Support\ServiceProvider;
    use Illuminate\Queue\Events\JobProcessed;
    use Illuminate\Queue\Events\JobProcessing;

    class AppServiceProvider extends ServiceProvider
    {
        /**
         * 全アプリケーションサービスの初期処理
         *
         * @return void
         */
        public function boot()
        {
            Queue::before(function (JobProcessing $event) {
                // $event->connectionName
                // $event->job
                // $event->job->payload()
            });

            Queue::after(function (JobProcessed $event) {
                // $event->connectionName
                // $event->job
                // $event->job->payload()
            });
        }

        /**
         * サービスプロバイダの登録
         *
         * @return void
         */
        public function register()
        {
            //
        }
    }

`Queue` [ファサード](/docs/{{version}}/facades)の`looping`メソッドを使用し、ワーカがキューからジョブをフェッチする前に、指定したコールバックを実行できます。たとえば、直前の失敗したジョブの未処理のままのトランザクションをロールバックするクロージャを登録できます。

    Queue::looping(function () {
        while (DB::transactionLevel() > 0) {
            DB::rollBack();
        }
    });
