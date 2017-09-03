# Laravel Horizon

- [イントロダクション](#introduction)
- [インストール](#installation)
    - [設定](#configuration)
    - [ダッシュボードの認可](#dashboard-authentication)
- [Horizonの実行](#running-horizon)
    - [Horizonのデプロイ](#deploying-horizon)
- [タグ](#tags)
- [通知](#notifications)
- [メトリックス](#metrics)

<a name="introduction"></a>
## イントロダクション

Horizon（水平線、展望）は、Laravel上で動作させるRedisキューのための、美しいダッシュボードとコード駆動の設定を提供します。Horizonにより、ジョブのスループット、ランタイム、ジョブの失敗など、キューシステムのキーメトリックを簡単に監視できます。

一つのシンプルな設定ファイルにすべてのワーカ設定を保存するため、チーム全体がコラボレート可能なソースコントロール下に、設定を保持できます。

<a name="installation"></a>
## インストール

> {note} 非同期のプロセスシグナルを活用しているため、Horizonを動作させるにはPHP7.1以上が必要です。

Composerを使い、LaravelプロジェクトにHorizonをインストールします。

    composer require laravel/horizon

Horizonをインストールしたら、`vendor:publish` Artisanコマンドを使用し、アセットを公開します。

    php artisan vendor:publish --provider="Laravel\Horizon\HorizonServiceProvider"

<a name="configuration"></a>
### 設定

Horizonのアセットを公開すると、`config/horizon.php`に一番重要な設定ファイルが設置されます。この設定ファイルにより、ワーカのオプションを設置できます。各オプションにはその目的が説明されていますので、このファイル全体をしっかりと確認してください。

#### バランスオプション

Horizonでは３つのバランシング戦略が選択できます。`simple`と`auto`、`false`です。`simple`戦略がデフォルトで、投入されたジョブをプロセス間に均等に割り当てます。

    'balance' => 'simple',

`auto`戦略は、現在のキュー負荷に基づき、それぞれのキューへ割り当てるワーカプロセス数を調整します。たとえば、`notifications`キューに１，０００ジョブが溜まっており、一方で`render`キューが空の場合、Horizonは空になるまで`notifications`キューにより多くのワーカを割り当てます。`balance`オプションへ`false`を設定すると、設定にリストした順番でキューが処理される、Laravelデフォルトの振る舞いが使われます。

<a name="dashboard-authentication"></a>
### ダッシュボードの認可

Horizonは、`/horizon`でダッシュボードを表示します。デフォルトでは、`local`環境でのみ、このダッシュボードへアクセスできます。ダッシュボードへ更に多くのアクセスポリシーを割り当てるには、`Horizon::auth`メソッドを使用する必要があります。`auto`メソッドは、`true`か`false`を返すコールバックを引数に取り、そのユーザーがHorizonダッシュボードへアクセスできるかどうかを指示します。

    Horizon::auth(function ($request) {
        // trueかfalseを返す
    });

<a name="running-horizon"></a>
## Horizonの実行

`config/horizon.php`設定ファイルでワーカの設定を済ませたら、`horizon` Artisanコマンドを使用し、Horizonを使用開始します。このコマンド一つで、設定済みのワーカ全部を起動できます。

    php artisan horizon

Horizonプロセスを`horizon:pause` Artisanコマンドで一時停止したり、`horizon:continue`コマンドで処理を続行したりできます。

    php artisan horizon:pause

    php artisan horizon:continue

マシン上のマスタHorizonプロセスを穏やかに終了させたい場合は、`horizon:terminate` Artisanコマンドを使用します。現在処理中のジョブが完了した後に、Horizonは停止します。

    php artisan horizon:terminate

<a name="deploying-horizon"></a>
### Horizonのデプロイ

Horizonを実働サーバにデプロイしている場合、`php artisan horizon`コマンドをプロセスモニタで監視し、予期せず終了した場合には再起動をかけるように設定する必要があります。サーバに新しいコードをデプロイしたときには、Horizonプロセスを停止指示する必要があります。その結果、プロセスモニタにより再起動され、コードの変更が反映されます。

マシン上のマスターHorizonプロセスは、`horizon:terminate` Artisanコマンドで穏やかに終了できます。現在処理中のジョブを完了させた後に、Horizonは停止します。

    php artisan horizon:terminate

#### Supervisor設定

`horizon`プロセスを管理するため、Supervisorプロセスモニタを使用する場合は、以下の設定ファイルが利用できるでしょう。

    [program:horizon]
    process_name=%(program_name)
    command=php /home/forge/app.com/artisan horizon
    autostart=true
    autorestart=true
    user=forge
    redirect_stderr=true
    stdout_logfile=/home/forge/app.com/horizon.log

> {tip} 自分のサーバ管理に自身がない場合は、[Laravel Forge](https://forge.laravel.com)の利用を考慮してください。ForgeはHorizonを利用するモダンで堅牢なLaravelアプリケーションに必要なすべてをPHP7以上のサーバに供給します。

<a name="tags"></a>
## タグ

Horizonではmailableやイベントブロードキャスト、通知、キューイベントリスナなどを含むジョブに「タグ」を割り付けられます。実際、ジョブに割り付けられたEloquentモデルに基づいて、ほとんどのジョブでは賢く自動的にHorizonがタグ付けします。例として、以下のジョブをご覧ください。

    <?php

    namespace App\Jobs;

    use App\Video;
    use Illuminate\Bus\Queueable;
    use Illuminate\Queue\SerializesModels;
    use Illuminate\Queue\InteractsWithQueue;
    use Illuminate\Contracts\Queue\ShouldQueue;
    use Illuminate\Foundation\Bus\Dispatchable;

    class RenderVideo implements ShouldQueue
    {
        use Dispatchable, InteractsWithQueue, Queueable, SerializesModels;

        /**
         * ビデオインスタンス
         *
         * @var \App\Video
         */
        public $video;

        /**
         * 新しいジョブインスタンスの生成
         *
         * @param  \App\Video  $video
         * @return void
         */
        public function __construct(Video $video)
        {
            $this->video = $video;
        }

        /**
         * ジョブの実行
         *
         * @return void
         */
        public function handle()
        {
            //
        }
    }

`id`が`1`の`App\Video`インスタンスを持つジョブがキューされると、自動的に`App\Video:1`タグが付けられます。HorizonはジョブのプロパティがEloquentモデルであるかを確認するからです。Eloquentモデルが見つかると、Horizonはモデルのクラス名と主キーを使用し、賢くタグ付けします。

    $video = App\Video::find(1);

    App\Jobs\RenderVideo::dispatch($video);

#### 手動のタグ付け

queueableオブジェクトのタグを任意に定義したい場合は、そのクラスで`tags`メソッドを定義してください。

    class RenderVideo implements ShouldQueue
    {
        /**
         * ジョブに割り付けるタグの取得
         *
         * @return array
         */
        public function tags()
        {
            return ['render', 'video:'.$this->video->id];
        }
    }

<a name="notifications"></a>
## 通知

> **注意：** 通知を利用する前に、プロジェクトへ`guzzlehttp/guzzle` Composerパッケージを追加してください。HorizonでSMSを通知する設定の場合は、[Nexmo通知ドライバーの動作要件](https://laravel.com/docs/5.4/notifications#sms-notifications)についても、確認する必要があります。

あるキューが長時間waitしている時に、通知を受け取りたい場合は、`Horizon::routeSlackNotificationsTo`や、`Horizon::routeSmsNotificationsTo`メソッドを利用してください。

    Horizon::routeSlackNotificationsTo('slack-webhook-url');

    Horizon::routeSmsNotificationsTo('15556667777');

#### 通知wait時間のシュレッドホールド設定

何秒を「長時間」と考えるかは、`config/horizon.php`設定ファイルで指定できます。このファイルの`wait`設定オプションで、接続／キューの組み合わせごとに、長時間と判定するシュレッドホールドをコントロールできます。

    'waits' => [
        'redis:default' => 60,
    ],

<a name="metrics"></a>
## メトリックス

Horizonはジョブとキューの待ち時間とスループットの情報をダッシュボードに表示します。このダッシュボードを表示するために、アプリケーションの[スケジューラ](/docs/{{version}}/scheduling)で、５分毎に`snapshot` Artisanコマンドを実行する設定を行う必要があります。

    /**
     * アプリケーションのコマンドスケジュールの定義
     *
     * @param  \Illuminate\Console\Scheduling\Schedule  $schedule
     * @return void
     */
    protected function schedule(Schedule $schedule)
    {
        $schedule->command('horizon:snapshot')->everyFiveMinutes();
    }
