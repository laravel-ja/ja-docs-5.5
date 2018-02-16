# Artisanコンソール

- [イントロダクション](#introduction)
- [コマンド記述](#writing-commands)
    - [コマンド生成](#generating-commands)
    - [コマンド構造](#command-structure)
    - [クロージャコマンド](#closure-commands)
- [コマンドライン指定の定義](#defining-input-expectations)
    - [引数](#arguments)
    - [オプション](#options)
    - [入力配列](#input-arrays)
    - [入力の説明](#input-descriptions)
- [コマンドI/O](#command-io)
    - [入力の取得](#retrieving-input)
    - [入力のプロンプト](#prompting-for-input)
    - [出力の書き出し](#writing-output)
- [コマンド登録](#registering-commands)
- [プログラムによるコマンド実行](#programmatically-executing-commands)
    - [他のコマンドからの呼び出し](#calling-commands-from-other-commands)

<a name="introduction"></a>
## イントロダクション

ArtisanはLaravelに含まれているコマンドラインインターフェイスです。アプリケーション開発全体で役に立つ、数多くのコマンドを提供しています。使用可能な全Artisanコマンドを確認するには、`list`コマンドを使います。

    php artisan list

全てのコマンドは「ヘルプ」が用意されており、説明と使用できる引数、オプションを表示します。ヘルプを表示するには`help`に続いてコマンド名を入力してください。

    php artisan help migrate

#### Laravel REPL

全てのLaravelアプリケーションには、[PsySH](https://github.com/bobthecow/psysh)パッケージによるREPLである、Tinkerが含まれています。Tinkerにより、Laravel全体のEloquent ORM、ジョブ、イベントなどをコマンドラインから操作できます。Tinker環境に入るには、`tinker` Artisanコマンドを実行します。

    php artisan tinker

<a name="writing-commands"></a>
## コマンド記述

Artisanに用意されているコマンドに加え、独自のカスタムコマンドも構築できます。コマンドは通常、`app/Console/Commands`ディレクトリへ設置します。しかし、Composerによりコマンドがロードできる場所であるならば、自由に設置場所を選べます。

<a name="generating-commands"></a>
### コマンド生成

新しいコマンドを作成するには、`make:command` Artisanコマンドを使います。このコマンドは、`app/Console/Commands`ディレクトリへ新しいコマンドクラスを設置します。アプリケーションにこのディレクトリがなくても心配ありません。最初に、`make:command` Artisanコマンドを実行するときに作成されます。生成されたコマンドには、すべてのコマンドで必要な、プロパティとメソッドがデフォルトで一揃い用意されています。

    php artisan make:command SendEmails

<a name="command-structure"></a>
### コマンド構造

コマンドが生成できたら、`list`スクリーンでそのコマンドが表示できるように、クラスの`signature`と`description`プロパティを指定してください。`handle`メソッドは、コマンド実行時に呼び出されます。コマンドのロジックは、このメソッドの中へ記述します。

> {tip} コンソールコマンドを軽いままにし、実行内容をアプリケーションサービスとして遅らせるのは、コードの再利用性のためのグッドプラクティスです。以下の例で、メール送信の「重荷を軽く」するために、サービスクラスを注入しているところに注目してください。

コマンドのサンプルを見てみましょう。コマンドのコンストラクタで、必要な依存を注入できるところに注意を払ってください。Laravelの[サービスコンテナ](/docs/{{version}}/container)は、コンストラクタでタイプヒントされた依存をすべて自動的に注入します。

    <?php

    namespace App\Console\Commands;

    use App\User;
    use App\DripEmailer;
    use Illuminate\Console\Command;

    class SendEmails extends Command
    {
        /**
         * コンソールコマンドの名前と引数、オプション
         *
         * @var string
         */
        protected $signature = 'email:send {user}';

        /**
         * コンソールコマンドの説明
         *
         * @var string
         */
        protected $description = 'Send drip e-mails to a user';

        /**
         * drip Eメールサービス
         *
         * @var DripEmailer
         */
        protected $drip;

        /**
         * 新しいコマンドインスタンスの生成
         *
         * @param  DripEmailer  $drip
         * @return void
         */
        public function __construct(DripEmailer $drip)
        {
            parent::__construct();

            $this->drip = $drip;
        }

        /**
         * コンソールコマンドの実行
         *
         * @return mixed
         */
        public function handle()
        {
            $this->drip->send(User::find($this->argument('user')));
        }
    }

<a name="closure-commands"></a>
### クロージャコマンド

クロージャベースコマンドは、クラスによりコンソールコマンドを定義する方法の代替手法を提供します。ルートクロージャがコントローラの代替であるのと同じように、コマンドクロージャはコマンドクラスの代替だと考えてください。`app/Console/Kernel.php`ファイルの`commands`メソッドの中で、Laravelは`routes/console.php`ファイルをロードしています。

    /**
     * アプリケーションのクロージャベースコマンドの登録
     *
     * @return void
     */
    protected function commands()
    {
        require base_path('routes/console.php');
    }

HTTPルートは定義していませんが、このファイルはアプリケーションに対する、コンソールベースのエントリポイント（ルート）を定義しているのです。`Artisan::command`メソッドを使い、全クロージャベースルートをこのファイル中で定義します。`command`メソッドは[コマンドの使い方](#defining-input-expectations)と、コマンドの引数とオプションを受け取るクロージャを引数として受け取ります。

    Artisan::command('build {project}', function ($project) {
        $this->info("Building {$project}!");
    });

クロージャは裏で動作するコマンドインスタンスと結合します。そのため、完全なコマンドクラス上でアクセスできる、通常のヘルパメソッドにすべてアクセスできます。

#### 依存のタイプヒント

コマンドの引数とオプションに付け加え、コマンドクロージャはタイプヒントによる追加の依存を受け取り、それらは[サービスコンテナ](/docs/{{version}}/container)により依存解決されます。

    use App\User;
    use App\DripEmailer;

    Artisan::command('email:send {user}', function (DripEmailer $drip, $user) {
        $drip->send(User::find($user));
    });

#### クロージャコマンドの説明

クロージャベースコマンドの定義時には、コマンドの説明を追加するために`describe`メソッドを使います。この説明は`php artisan list`や`php artisan help`コマンド実行時に表示されます。

    Artisan::command('build {project}', function ($project) {
        $this->info("Building {$project}!");
    })->describe('Build the project');

<a name="defining-input-expectations"></a>
## コマンドライン指定の定義

コンソールコマンドを書く場合、引数やオプションによりユーザーから情報を入力してもらうのが一般的です。コマンドの`signature`プロパティにユーザーに期待する入力を記述することにより、Laravelではとても便利に定義できます。`signature`プロパティ１行でわかりやすいルート定義のような記法により、コマンドの名前と引数、オプションを定義できます。

<a name="arguments"></a>
### 引数

ユーザーから入力してもらう引数とオプションは全て波括弧で囲みます。以下の例の場合、**必須の**`user`コマンド引数を定義しています。

    /**
     * コンソールコマンドの名前と引数、オプション
     *
     * @var string
     */
    protected $signature = 'email:send {user}';

任意の引数やデフォルト値を指定することも可能です。

    // 任意指定な引数
    email:send {user?}

    // デフォルト値を持つ、任意指定な引数
    email:send {user=foo}

<a name="options"></a>
### オプション

オプションも引数と同様にユーザーからの入力です。コマンドラインで指定する場合、２つのハイフン(`--`)を先頭に付けます。値を取るものと、取らないもの、２つのタイプのオプションがあります。値を取らないオプションは、論理的な「スイッチ」として動作します。このタイプのオプションを確認しましょう。

    /**
     * コンソールコマンドの名前と引数、オプション
     *
     * @var string
     */
    protected $signature = 'email:send {user} {--queue}';

この例の場合、Artisanコマンド起動時に、`--queue`スイッチを指定できるようになります。`--queue`スイッチが指定されると、オプションの値は`true`になります。そうでなければ、値は`false`になります。

    php artisan email:send 1 --queue

<a name="options-with-values"></a>
#### 値を取るオプション

次に、ユーザーによる値の入力を基体するオプションを確認しましょう。ユーザーによりオプションの値の指定が必要である場合、オプション名は`=`記号ではじめます。

    /**
     * コンソールコマンドの名前と引数、オプション
     *
     * @var string
     */
    protected $signature = 'email:send {user} {--queue=}';

この例では、次のようにオプションに値を指定します。

    php artisan email:send 1 --queue=default

オプション名の後に値を指定することにより、オプションのデフォルト値を指定できます。ユーザーにより値が指定されない場合、デフォルト値が指定されます。

    email:send {user} {--queue=default}

<a name="option-shortcuts"></a>
#### オプションショートカット

オプション定義時にショートカットを割りつけるには、完全なオプション名の前に|で区切り、ショートカットを指定してください。

    email:send {user} {--Q|queue}

<a name="input-arrays"></a>
### 入力配列

引数やオプションで、配列による入力を定義したい場合は、`*`文字を使います。最初に引数の配列指定の例を、見てみましょう。

    email:send {user*}

このメソッド呼び出し時に、`user`引数はコマンドラインの順番に渡されます。以下のコマンドは、`user`に`['foo', 'bar']`をセットします。

    php artisan email:send foo bar

オプションの配列入力を定義する場合、各値はオプション名を付けて指定する必要があります。

    email:send {user} {--id=*}

    php artisan email:send --id=1 --id=2

<a name="input-descriptions"></a>
### 入力の説明

入力の引数とオプションの説明をパラメータと説明をコロンで分けることで指定できます。コマンドを定義するため、もう少し余裕が欲しければ、定義を複数行へ自由に分割してください。

    /**
     * コンソールコマンドの名前と引数、オプション
     *
     * @var string
     */
    protected $signature = 'email:send
                            {user : The ID of the user}
                            {--queue= : Whether the job should be queued}';

<a name="command-io"></a>
## コマンドI/O

<a name="retrieving-input"></a>
### 入力の取得

コマンド実行時に指定された引数やオプションの値にアクセスする必要は明確にあります。そのために、`argument`と`option`メソッドを使用して下さい。

    /**
     * コンソールコマンドの実行
     *
     * @return mixed
     */
    public function handle()
    {
        $userId = $this->argument('user');

        //
    }

全引数を「配列」で受け取りたければ、`argument`を呼んでください。

    $arguments = $this->arguments();

引数と同様、とても簡単に`option`メソッドでオプションを取得できます。`argument`メソッドと同じように呼びだせば、全オプションを「配列」で取得できます。

    // 特定オプションの取得
    $queueName = $this->option('queue');

    // 全オプションの取得
    $options = $this->options();

引数やオプションが存在していない場合、`null`が返されます。

<a name="prompting-for-input"></a>
### 入力のプロンプト

コマンドラインに付け加え、コマンド実行時にユーザーに入力を尋ねることもできます。`ask`メソッドにユーザーへ表示する質問を指定すると、ユーザーに入力してもらい、その後値がコマンドに返ってきます。

    /**
     * コンソールコマンドの実行
     *
     * @return mixed
     */
    public function handle()
    {
        $name = $this->ask('What is your name?');
    }

`secret`メソッドも`ask`と似ていますが、コンソールでユーザーがタイプした値を表示しません。このメソッドはパスワードのような機密情報を尋ねるときに便利です。

    $password = $this->secret('What is the password?');

#### 確認

単純にユーザーから確認を取りたい場合は、`confirm`メソッドを使ってください。このメソッドはデフォルトで`false`を返します。プロンプトに対して`y`か`yes`が入力されると、`true`を返します。

    if ($this->confirm('Do you wish to continue?')) {
        //
    }

#### 自動補完

`anticipate`メソッドは可能性のある選択肢の、自動補完機能を提供するために使用します。ユーザーは表示される自動補完の候補にかかわらず、どんな答えも返答できます。

    $name = $this->anticipate('What is your name?', ['Taylor', 'Dayle']);

#### 複数選択の質問

あらかじめ決められた選択肢をユーザーから選んでもらいたい場合は、`choice`メソッドを使用します。何も選ばれなかった場合に返ってくるデフォルト値の配列インデックスを指定することも可能です。

    $name = $this->choice('What is your name?', ['Taylor', 'Dayle'], $defaultIndex);

<a name="writing-output"></a>
### 出力の書き出し

コンソールに出力するには、`line`、`info`、`comment`、`question`、`error`メソッドを使います。その名前が表す目的で使用し、それぞれ適当なANSIカラーが表示に使われます。たとえば、全般的な情報をユーザーへ表示しましょう。通常、`info`メソッドはコンソールに緑の文字で表示します。

    /**
     * コンソールコマンドの実行
     *
     * @return mixed
     */
    public function handle()
    {
        $this->info('Display this on the screen');
    }

エラーメッセージを表示する場合は、`error`メソッドです。エラーメッセージは通常赤で表示されます。

    $this->error('Something went wrong!');

プレーンな、色を使わずにコンソール出力する場合は、`line`メソッドを使います。

    $this->line('Display this on the screen');

#### テーブルレイアウト

`table`メソッドにより簡単に正しくデータの複数行／カラムをフォーマットできます。メソッドにヘッダと行を渡してください。幅と高さは与えたデータから動的に計算されます。

    $headers = ['Name', 'Email'];

    $users = App\User::all(['name', 'email'])->toArray();

    $this->table($headers, $users);

#### プログレスバー

時間がかかるタスクでは、進捗状況のインディケータを表示できると便利です。出力のオブジェクトを使用し、プログレスバーを開始、進行、停止することができます。最初に、処理全体を繰り返す総ステップ数を定義します。それから、各アイテムの処理の後に、プログレスバーを進めます。

    $users = App\User::all();

    $bar = $this->output->createProgressBar(count($users));

    foreach ($users as $user) {
        $this->performTask($user);

        $bar->advance();
    }

    $bar->finish();

より詳細なオプションに関しては、[Symfonyのプログレスバーコンポーネントのドキュメント](https://symfony.com/doc/2.7/components/console/helpers/progressbar.html)で確認してください。

<a name="registering-commands"></a>
## コマンド登録

コンソールカーネルの`command`メソッドの中で、`load`メソッドが呼び出されているため、`app/Console/Commands`ディレクトリ中の全コマンドは、自動的にArtisanに登録されます。そのため、他のディレクトリに存在するArtisanコマンドをスキャンするために、`load`メソッドを自由に呼び出すことができます。

    /**
     * アプリケーションのコマンドを登録
     *
     * @return void
     */
    protected function commands()
    {
        $this->load(__DIR__.'/Commands');
        $this->load(__DIR__.'/MoreCommands');

        // ...
    }

また、`app/Console/Kernel.php`ファイルの`$commands`プロパティへクラス名を追加することで、自分でコマンドを登録することもできます。Artisanが起動すると、[サービスコンテナ](/docs/{{version}}/container)によりこのプロパティ中にリストされているコマンドは全て依存解決され、Artisanへ登録されます。

    protected $commands = [
        Commands\SendEmails::class
    ];

<a name="programmatically-executing-commands"></a>
## プログラムによるコマンド実行

ArtisanコマンドをCLI以外から実行したい場合もあるでしょう。たとえばルートやコントローラからArtisanコマンドを起動したい場合です。`Artisan`ファサードの`call`メソッドで実行できます。`call`メソッドでは第１引数にコマンド名を指定します。第２引数にコマンドのパラメーターを配列で指定します。exitコードが返されます。

    Route::get('/foo', function () {
        $exitCode = Artisan::call('email:send', [
            'user' => 1, '--queue' => 'default'
        ]);

        //
    });

`Artisan`ファサードの`queue`メソッドを使用すると、[キューワーカー](/docs/{{version}}/queues)によりバックグラウンドでArtisanコマンドが実行されるようにキューされます。このメソッドを使用する前に、キューの設定を確実に済ませ、キューリスナを実行してください。

    Route::get('/foo', function () {
        Artisan::queue('email:send', [
            'user' => 1, '--queue' => 'default'
        ]);

        //
    });

Artisanコマンドが実行される接続やキューを特定することもできます。

    Artisan::queue('email:send', [
        'user' => 1, '--queue' => 'default'
    ])->onConnection('redis')->onQueue('commands');

#### 配列値の引数

コマンドで配列を受け取るオプションを定義している場合、そのオプションに配列値を渡してください。

    Route::get('/foo', function () {
        $exitCode = Artisan::call('email:send', [
            'user' => 1, '--id' => [5, 13]
        ]);
    });

#### 論理値の引数

`migrate:refresh`コマンドの`--force`フラグのように、文字列値を受け取らないオプションに値を指定する必要がある場合は、`true`か`false`を渡す必要があります。

    $exitCode = Artisan::call('migrate:refresh', [
        '--force' => true,
    ]);

<a name="calling-commands-from-other-commands"></a>
### 他のコマンドからの呼び出し

存在するArtisanコマンドから別のコマンドを呼び出したい場合もあるでしょう。`call`メソッドで実行できます。この`call`メソッドへは、コマンド名とコマンドパラメーターの配列を指定します。

    /**
     * コンソールコマンドの実行
     *
     * @return mixed
     */
    public function handle()
    {
        $this->call('email:send', [
            'user' => 1, '--queue' => 'default'
        ]);

        //
    }

他のコンソールコマンドを実行しつつ、出力を全て無視したい場合は、`callSilent`メソッドを使用してください。`callSilent`メソッドの使い方は、`call`メソッドと同じです。

    $this->callSilent('email:send', [
        'user' => 1, '--queue' => 'default'
    ]);
