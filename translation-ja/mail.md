# メール

- [イントロダクション](#introduction)
    - [ドライバの動作要件](#driver-prerequisites)
- [Mailable概論](#generating-mailables)
- [Mailableプログラミング](#writing-mailables)
    - [Senderの設定](#configuring-the-sender)
    - [ビューの設定](#configuring-the-view)
    - [ビューデータ](#view-data)
    - [添付](#attachments)
    - [インライン添付](#inline-attachments)
    - [SwiftMailerメッセージのカスタマイズ](#customizing-the-swiftmailer-message)
- [Markdown Mailable](#markdown-mailables)
    - [Markdown Mailableの生成](#generating-markdown-mailables)
    - [Markdownメッセージの記述](#writing-markdown-messages)
    - [コンポーネントのカスタマイズ](#customizing-the-components)
- [ブラウザでのMailableプレビュー](#previewing-mailables-in-the-browser)
- [メール送信](#sending-mail)
    - [キュー使用メール](#queueing-mail)
- [メールとローカル開発](#mail-and-local-development)
- [イベント](#events)

<a name="introduction"></a>
## イントロダクション

Laravelは人気の高い[SwiftMailer](https://swiftmailer.symfony.com/)ライブラリーのクリーンでシンプルなAPIを提供しています。SMTP、Mailgun、SparkPost、Amazon SES、PHPの`mail`機能、`sendmail`ドライバーを提供しており、選択したローカルやクラウドベースのサービスを使い、素早くメール送信が開始できるようにしています。

<a name="driver-prerequisites"></a>
### ドライバの動作要件

MailgunとSparkPostなどAPIベースドライバはシンプルでSMTPサーバよりも高速です。可能であれば、こうしたドライバを使用しましょう。APIドライバはすべて、Guzzle HTTPライブラリを必要としますので、Composerパッケージマネージャでインストールしてください。

    composer require guzzlehttp/guzzle

#### Mailgunドライバ

Mailgunドライバを使用する場合、最初にGuzzleをインストールしてください。それから`config/mail.php`設定ファイル中の`driver`オプションを`mailgun`に設定してください。次に`config/services.php`設定ファイルが以下のオプションを含んでいるか確認してください。

    'mailgun' => [
        'domain' => 'your-mailgun-domain',
        'secret' => 'your-mailgun-key',
    ],

#### SparkPostドライバ

SparkPostドライバを使用するには、最初にGuzzleをインストールし、次に`config/mail.php`設定ファイル中の`driver`オプションを`sparkpost`に設定してください。`config/services.php`設定ファイルに以下のオプションが含まれているか確認してください。

    'sparkpost' => [
        'secret' => 'your-sparkpost-key',
    ],

#### SESドライバ

Amazon SESドライバを使う場合、Amazon AWS SDK for PHPをインストールしてください。`composer.json`ファイルの`require`セクションに以下の行を追加し、`composer update`コマンドを実行します。

    "aws/aws-sdk-php": "~3.0"

次に`config/mail.php`設定ファイルの`driver`オプションを`ses`に設定します。それから`config/services.php`設定ファイルが以下の内容になっているか確認してください。

    'ses' => [
        'key' => 'your-ses-key',
        'secret' => 'your-ses-secret',
        'region' => 'ses-region',  // e.g. us-east-1
    ],

<a name="generating-mailables"></a>
## Mailable概論

Laravelではアプリケーションが送信する、各種メールタイプを"mailable"クラスとして表します。これらのクラスは、`app/Mail`ディレクトリに保存します。アプリケーションにこのディレクトリが存在していなくても、心配ありません。`make:mail`コマンドを使用して、最初にmailableクラスを生成する時に、作成されます。

    php artisan make:mail OrderShipped

<a name="writing-mailables"></a>
## Mailableプログラミング

全mailableクラスの設定は、`build`メソッド中で行います。このメソッド中でメールのプレゼンテーションとデリバリーを設定する、`from`、`subject`、`view`、`attach`など様々なメソッドを呼び出します。

<a name="configuring-the-sender"></a>
### Senderの設定

#### `from`メソッドの使用

最初に、メールの送信者の設定を見てみましょう。言い換えれば、"from"により、メールを送信する人を指定します。送信者の設定には２つの方法があります。最初にmailableクラスの`build`メソッドの中で、`from`メソッドを使う方法です。

    /**
     * メッセージの生成
     *
     * @return $this
     */
    public function build()
    {
        return $this->from('example@example.com')
                    ->view('emails.orders.shipped');
    }

#### グローバル`from`アドレスの使用

もし、アプリケーションで同じ"from"アドレスを全メールで使用するのであれば、生成する全mailableクラスで`from`メソッドを呼び出すのは面倒です。代わりに、グローバルな"from"アドレスを`config/mail.php`設定ファイルで指定しましょう。このアドレスは、mailableクラスの中で、"from"アドレスが指定されなかった場合に使用されます。

    'from' => ['address' => 'example@example.com', 'name' => 'App Name'],

<a name="configuring-the-view"></a>
### ビューの設定

mailableクラスの`build`メソッドの中で、メールの中身をレンダーする時に使用するテンプレートを`view`メソッドにより指定できます。各メールでは内容をレンダーするのに[Blade テンプレート](/docs/{{version}}/blade)を通常使用しますので、メールのHTMLを構築する時にBladeテンプレートエンジンのパワーと利便性をフルに利用できます。

    /**
     * メッセージの生成
     *
     * @return $this
     */
    public function build()
    {
        return $this->view('emails.orders.shipped');
    }

> {tip} メール用テンプレートをすべて設置する、`resources/views/emails`ディレクトリを作成することができます。しかし、`resources/views`ディレクトリの中であれば、好きな場所へ自由に設置できます。

#### 平文テキストメール

平文テキスト版のメールを定義したいときは、`text`メソッドを使います。`view`メソッドと同様に、`text`メソッドは、メールの内容をレンダーするために使用する、テンプレート名を引数に取ります。メッセージのHTML版と平文テキスト版の両方を定義することも可能です。

    /**
     * メッセージの生成
     *
     * @return $this
     */
    public function build()
    {
        return $this->view('emails.orders.shipped')
                    ->text('emails.orders.shipped_plain');
    }

<a name="view-data"></a>
### ビューデータ

#### publicプロパティ使用

通常、メールのHTMLをレンダーする時には、ビューへ使用するデータを渡します。ビューでデータを使用できるようにするには、２つの方法があります。まず、mailableクラスで定義したpublicプロパティは、ビューで自動的に利用できます。そのため、たとえばmailableクラスのコンストラクタへデータを渡し、そのデータをクラス上のプロパティとして定義できます。

    <?php

    namespace App\Mail;

    use App\Order;
    use Illuminate\Bus\Queueable;
    use Illuminate\Mail\Mailable;
    use Illuminate\Queue\SerializesModels;

    class OrderShipped extends Mailable
    {
        use Queueable, SerializesModels;

        /**
         * 注文インスタンス
         *
         * @var Order
         */
        public $order;

        /**
         * 新しいメッセージインスタンスの生成
         *
         * @return void
         */
        public function __construct(Order $order)
        {
            $this->order = $order;
        }

        /**
         * メッセージの生成
         *
         * @return $this
         */
        public function build()
        {
            return $this->view('emails.orders.shipped');
        }
    }

データをpublicプロパティにセットしたら、自動的にビューで使用できるようになり、Bladeテンプレート中で、他のデータと同様にアクセスできます。

    <div>
        Price: {{ $order->price }}
    </div>

#### `with`メソッド使用

メールのデータフォーマットをテンプレートへ渡す前にカスタマイズしたい場合は、`with`メソッドを使いデータをビューへ渡すことができます。通常、この場合もデータをmailableクラスのコンストラクタで渡すことになるでしょう。しかし、自動的にテンプレートで使用可能にならないように、`protected`か`private`プロパティへデータをセットしてください。それから、テンプレートで使用したいデータの配列を引数として、`with`メソッドを呼び出してください。

    <?php

    namespace App\Mail;

    use App\Order;
    use Illuminate\Bus\Queueable;
    use Illuminate\Mail\Mailable;
    use Illuminate\Queue\SerializesModels;

    class OrderShipped extends Mailable
    {
        use Queueable, SerializesModels;

        /**
         * 注文インスタンス
         *
         * @var Order
         */
        protected $order;

        /**
         * 新しいメッセージインスタンスの生成
         *
         * @return void
         */
        public function __construct(Order $order)
        {
            $this->order = $order;
        }

        /**
         * メッセージの生成
         *
         * @return $this
         */
        public function build()
        {
            return $this->view('emails.orders.shipped')
                        ->with([
                            'orderName' => $this->order->name,
                            'orderPrice' => $this->order->price,
                        ]);
        }
    }

`with`メソッドへ渡したデータは、ビューで自動的に使用可能になり、Bladeテンプレートの他のデータと同様にアクセスできます。

    <div>
        Price: {{ $orderPrice }}
    </div>

<a name="attachments"></a>
### 添付

メールへ添付するには、`attach`メソッドをmailableクラスの`build`メソッド中で呼び出します。`attach`メソッドは最初の引数に、ファイルのフルパスを取ります。

        /**
         * メッセージの生成
         *
         * @return $this
         */
        public function build()
        {
            return $this->view('emails.orders.shipped')
                        ->attach('/path/to/file');
        }

ファイルをメッセージ添付する場合、`attach`メソッドの第２引数として配列を渡し、表示名やMIMEタイプを指定することもできます。

        /**
         * メッセージの生成
         *
         * @return $this
         */
        public function build()
        {
            return $this->view('emails.orders.shipped')
                        ->attach('/path/to/file', [
                            'as' => 'name.pdf',
                            'mime' => 'application/pdf',
                        ]);
        }

#### Rawデータ添付

`attachData`メソッドは添付内容のバイト文字列をそのまま添付する場合に使用します。たとえば、メモリ中でPDFを生成し、それをディスクに書き出さずに、メールへ添付したい場合にこのメソッドを使用できます。`attachData`メソッドはrawデータバイトを最初の引数に取り、ファイル名を第２引数に、オプションの配列を第３引数に取ります。

        /**
         * メッセージの生成
         *
         * @return $this
         */
        public function build()
        {
            return $this->view('emails.orders.shipped')
                        ->attachData($this->pdf, 'name.pdf', [
                            'mime' => 'application/pdf',
                        ]);
        }

<a name="inline-attachments"></a>
### インライン添付

インライン画像をメールに埋め込むのは、通常手間がかかります。しかし、Laravelは画像をメールに付け、最適なCIDを得る便利な方法を提供しています。インラインイメージを埋め込むには、メールビューの中で`$message`変数の`embed`メソッドを使ってください。Laravelでは全メールテンプレートで、`$message`変数が使用できるようになっていますので、この変数を渡すことについては心配する必要はありません。

    <body>
        ここに画像：

        <img src="{{ $message->embed($pathToFile) }}">
    </body>

> {note} Markdownメッセージでは、`$message`変数は使用できません。

#### 添付Rawデータの埋め込み

メールテンプレートへ埋め込むrawデータ文字列を既に用意してある場合は、`$message`変数の`embedData`メソッドを使ってください。

    <body>
        ここにrawデータからの画像：

        <img src="{{ $message->embedData($data, $name) }}">
    </body>

<a name="customizing-the-swiftmailer-message"></a>
### SwiftMailerメッセージのカスタマイズ

`Mailable`ベースクラスの`withSwiftMessage`クラスにより、メッセージ送信前にSwiftMailerメッセージインスタンスを直接呼び出すコールバクを登録できます。これにより配信する前に、メッセージを送信する機会を得られます。

        /**
         * メッセージの生成
         *
         * @return $this
         */
        public function build()
        {
            $this->view('emails.orders.shipped');

            $this->withSwiftMessage(function ($message) {
                $message->getHeaders()
                        ->addTextHeader('Custom-Header', 'HeaderValue');
            });
        }

<a name="markdown-mailables"></a>
## Markdown Mailable

Markdown mailableメッセージにより、事前に構築したテンプレートとメール通知のコンポーネントの利点をMailable中で利用できます。メッセージをMarkdownで記述すると、Laravelは美しいレスポンシブHTMLテンプレートをレンダーすると同時に、自動的に平文テキスト版も生成します。

<a name="generating-markdown-mailables"></a>
### Markdown Mailableの生成

対応するMarkdownテンプレートを指定し、Mailableを生成するには、`make:mail` Artisanコマンドを`--markdown`オプション付きで使用します。

    php artisan make:mail OrderShipped --markdown=emails.orders.shipped

次に、`build`メソッド内で、Mailableを設定します。`view`メソッドの代わりに、`markdown`メソッドを呼び出します。`markdown`メソッドはMarkdownテンプレートの名前とテンプレートで使用するデータの配列を引数に取ります。

    /**
     * メッセージの生成
     *
     * @return $this
     */
    public function build()
    {
        return $this->from('example@example.com')
                    ->markdown('emails.orders.shipped');
    }

<a name="writing-markdown-messages"></a>
### Markdown Messageの記述

Markdown MailableではBladeコンポーネントとMarkdown記法が利用でき、メールメッセージを簡単に構築できると同時に、Laravelが用意しているコンポーネントも活用できます。

    @component('mail::message')
    # 発送のお知らせ

    商品が発送されました！

    @component('mail::button', ['url' => $url])
    注文の確認
    @endcomponent

    ありがとうございました。<br>
    {{ config('app.name') }} 様
    @endcomponent

> {tip} Markdownメールを書く場合は、過剰なインデントを付けないでください。Markdownは段付をコードブロックとしてパースします。

#### Buttonコンポーネント

ボタンコンポーネントは中央寄せのボタンリンクをレンダーします。このコンポーネントは引数として、`url`とオプションの`color`を受け取ります。サポートしている色は`blue`、`green`、`red`です。メッセージに好きなだけのボタンコンポーネントを追加できます。

    @component('mail::button', ['url' => $url, 'color' => 'green'])
    注文の確認
    @endcomponent

#### Panelコンポーネント

パネルコンポーネントは、メッセージの他の部分とは少し異なった背景色のパネルの中に、指定されたテキストブロックをレンダーします。これにより、指定するテキストに注目を集められます。

    @component('mail::panel')
    ここはパネルの内容です。
    @endcomponent

#### Tableコンポーネント

テーブルコンポーネントは、MarkdownテーブルをHTMLテーブルへ変換します。このコンポーネントはMarkdownテーブルを内容として受け入れます。デフォルトのMarkdownテーブルの記法を使った、文字寄せをサポートしています。

    @component('mail::table')
    | Laravel       | テーブル      | 例       |
    | ------------- |:-------------:| --------:|
    | Col 2 is      | 中央寄せ      | $10      |
    | Col 3 is      | 右寄せ        | $20      |
    @endcomponent

<a name="customizing-the-components"></a>
### コンポーネントのカスタマイズ

自身のアプリケーション向きにカスタマイズできるように、Markdownメールコンポーネントは全てエクスポートできます。コンポーネントをエクスポートするには、`vendor:publish` Artisanコマンドを使い、`laravel-mail`アセットを公開します。

    php artisan vendor:publish --tag=laravel-mail

ディレクトリ下に、`html`と`markdown`ディレクトリがあります。各ディレクトリは名前が示す形式で、利用できるコンポーネント全てのレスポンシブなプレゼンテーションを持っています。`html`ディレクトリ下のコンポーネントはHTMLバージョンのメールを生成するために使用され、`markdown`ディレクトリ下のコンポーネントは平文テキストバージョンを生成するために使用されます。これらのコンポーネントはお好きなように自由にカスタマイズしてください。

#### CSSのカスタマイズ

コンポーネントをエクスポートすると、`resources/views/vendor/mail/html/themes`ディレクトリに、`default.css`ファイルが用意されます。このファイル中のCSSをカスタマイズすれば、Markdownメールメッセージ変換後のHTML形式の中に、インラインCSSとして自動的に取り込まれます。

> {tip} Markdownコンポーネント全体の新しいテーマを作成したい場合は、`html/themes`の中に新しいCSSファイルを記述し、`mail`設定ファイルの`theme`オプションを変更します。

<a name="previewing-mailables-in-the-browser"></a>
## ブラウザでのMailableプレビュー

mailableのテンプレートをデザインしているとき、Bladeテンプレートのようにブラウザでレンダし、簡単にレビューできると便利です。そのため、Laravelでは、ルートのクロージャやコントローラから直接mailableを返すことができます。mailableが返されるとレンダされ、ブラウザに表示されますので、実際のメールアドレスへ送る必要はなく、素早くレビューできます。

    Route::get('/mailable', function () {
        $invoice = App\Invoice::find(1);

        return new App\Mail\InvoicePaid($invoice);
    });

<a name="sending-mail"></a>
## メール送信

メッセージを送信するには、`Mail`[ファサード](/docs/{{version}}/facades)の`to`メソッドを使います。`to`メソッドはメールアドレス、ユーザーインスタンス、もしくはユーザーのコレクションを引数に取ります。一つのオブジェクト、もしくはオブジェクトのコレクションを渡すと、メーラは自動的にそれらの`email`と`name`プロパティを使用します。そのため、オブジェクトで、その属性を確実に使用可能にしてください。送信先を指定し終えたら、mailableクラスのインスタンスを`send`メソッドへ渡してください。

    <?php

    namespace App\Http\Controllers;

    use App\Order;
    use App\Mail\OrderShipped;
    use Illuminate\Http\Request;
    use Illuminate\Support\Facades\Mail;
    use App\Http\Controllers\Controller;

    class OrderController extends Controller
    {
        /**
         * 注文の配送
         *
         * @param  Request  $request
         * @param  int  $orderId
         * @return Response
         */
        public function ship(Request $request, $orderId)
        {
            $order = Order::findOrFail($orderId);

            // 配送処理…

            Mail::to($request->user())->send(new OrderShipped($order));
        }
    }

もちろん、メール送信時に"to"で受取人を指定するだけに限りません。"to"、"cc"、"bcc"による受取人をすべて一つのメソッドチェーンで呼び出せます。

    Mail::to($request->user())
        ->cc($moreUsers)
        ->bcc($evenMoreUsers)
        ->send(new OrderShipped($order));

<a name="queueing-mail"></a>
### キュー使用メール

#### メールメッセージのキューイング

メールメッセージを送ることにより、アプリケーションのレスポンス時間が極端に長くなり得るため、多くの開発者はメールメッセージをバックグランドで送信するためにキューイングすることを選びます。Laravelの[統一されたキューAPI](/docs/{{version}}/queues)を使うことで、簡単に実現できます。メールメッセージをキューへ送るには、`Mail`ファサードへ、受取人の指定の後に、`queue`メソッドを使います。

    Mail::to($request->user())
        ->cc($moreUsers)
        ->bcc($evenMoreUsers)
        ->queue(new OrderShipped($order));

このメソッドはバックグラウンドでメールを送信するため、自動的にジョブをキューに投入する面倒を見ます。もちろん、この機能を使用する前に[キューの設定](/docs/{{version}}/queues)を行う必要があります。

#### 遅延メッセージキュー

メッセージを投入するキューを指定したい場合、`laterOn`メソッドを使用します。最初の引数に、`later`メソッドは、メッセージを送信する時間を示す`DateTime`インスタンスを受け取ります。

    $when = now()->addMinutes(10);

    Mail::to($request->user())
        ->cc($moreUsers)
        ->bcc($evenMoreUsers)
        ->later($when, new OrderShipped($order));

#### 特定のキューに投入

`make:mail`コマンドにより生成されたmailableクラスにはすべて、`Illuminate\Bus\Queueable`トレイトが使用されています。接続とキュー名を指定する、`onQueue`と`onConnection`メソッドをすべてのmailableクラスインスタンスで呼び出せます。

    $message = (new OrderShipped($order))
                    ->onConnection('sqs')
                    ->onQueue('emails');

    Mail::to($request->user())
        ->cc($moreUsers)
        ->bcc($evenMoreUsers)
        ->queue($message);

#### デフォルトとしてキュー投入

いつもMailableクラスをキューへ投入したければ、クラスへ`ShouldQueue`契約を実装してください。それで、メール送信時に`send`メソッドを呼びだせば、そのMailableクラスは契約が実装されていますので、いつもキューイングされます。

    use Illuminate\Contracts\Queue\ShouldQueue;

    class OrderShipped extends Mailable implements ShouldQueue
    {
        //
    }

<a name="mail-and-local-development"></a>
## メールとローカル開発

メールを送信するアプリケーションを開発している間は、実際のメールアドレスにメールを送信したくはありません。Laravelはメールメッセージを実際に送信することをローカルでの開発期間の間、「無効」にする様々な方法を用意しています。

#### Logドライバ

メールを送信する代わりに、`log`メールドライバで、すべてのメールメッセージを確認のためにログファイルへ書き込こめます。アプリケーションの設定に関する詳細は、[設定のドキュメント](/docs/{{version}}/configuration#environment-configuration)を確認してください。

#### 全メールの送信先指定

Laravelが提供するもう一つの解決策は、フレームワークが送信する全メールの共通受け取り先を設定する方法です。この方法を使うと送信メッセージに指定した実際のアドレスの代わりに、アプリケーションが送る全メールを特定のアドレスに送信します。この方法を使用する場合、`config/mail.php`設定ファイルで`to`オプションを指定します。

    'to' => [
        'address' => 'example@example.com',
        'name' => 'Example'
    ],

#### Mailtrap

最後の方法は[Mailtrap](https://mailtrap.io)のようなサービスを使い、`smtp`ドライバで本当のメールクライアントにより内容を確認できる「ダミー」のメールボックスへメールメッセージを送る方法です。このアプローチの利点は最終的なメールをMailtrapのメッセージビュアーで実際に確認できることです。

<a name="events"></a>
## イベント

Laravelはメールメッセージ送信の過程で、イベントを２つ発行します。`MessageSending`イベントは、メッセージが送信される前に発行され、一方の`MessageSent`イベントは、メッセージを送った後に発行されます。２つのイベントは、キューした時点でなく、メールが**送信された**時に発行されることを覚えておいてください。これらに対するイベントリスナは、`EventServiceProvider`で定義できます。

    /**
     * アプリケーションへマッピングするイベントリスナ
     *
     * @var array
     */
    protected $listen = [
        'Illuminate\Mail\Events\MessageSending' => [
            'App\Listeners\LogSendingMessage',
        ],
        'Illuminate\Mail\Events\MessageSent' => [
            'App\Listeners\LogSentMessage',
        ],
    ];
