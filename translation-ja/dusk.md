# ブラウザテスト(Laravel Dusk)

- [イントロダクション](#introduction)
- [インストール](#installation)
    - [他ブラウザの使用](#using-other-browsers)
- [利用の開始](#getting-started)
    - [テストの生成](#generating-tests)
    - [テストの実行](#running-tests)
    - [環境の処理](#environment-handling)
    - [ブラウザの生成](#creating-browsers)
    - [認証](#authentication)
- [要素の操作](#interacting-with-elements)
    - [Duskセレクタ](#dusk-selectors)
    - [リンクのクリック](#clicking-links)
    - [テキスト、値、属性](#text-values-and-attributes)
    - [フォームの使用](#using-forms)
    - [添付ファイル](#attaching-files)
    - [キーワードの使用](#using-the-keyboard)
    - [マウスの使用](#using-the-mouse)
    - [セレクタの範囲指定](#scoping-selectors)
    - [要素の待機](#waiting-for-elements)
    - [Veuアサーションの作成](#making-vue-assertions)
- [使用可能なアサート](#available-assertions)
- [ページ](#pages)
    - [ページの生成](#generating-pages)
    - [ページの設定](#configuring-pages)
    - [ページへのナビゲーション](#navigating-to-pages)
    - [セレクタの簡略記述](#shorthand-selectors)
    - [ページメソッド](#page-methods)
- [コンポーネント](#components)
    - [コンポーネント生成](#generating-components)
    - [コンポーネントの使用](#using-components)
- [継続的インテグレーション](#continuous-integration)
    - [Travis CI](#running-tests-on-travis-ci)
    - [CircleCI](#running-tests-on-circle-ci)
    - [Codeship](#running-tests-on-codeship)

<a name="introduction"></a>
## イントロダクション

Laravel Dusk（ダースク：夕暮れ）は、利用が簡単なブラウザの自動操作／テストAPIを提供します。デフォルトのDuskは皆さんのマシンへ、JDKやSeleniumのインストールを求めません。代わりにDuskはスタンドアローンの[ChromeDriver](https://sites.google.com/a/chromium.org/chromedriver/home)を使用します。しかし、好みのSeleniumコンパチドライバも自由に使用することもできます。

<a name="installation"></a>
## インストール

使用を開始するには、`laravel/dusk`コンポーサ依存パッケージをプロジェクトへ追加します。

    composer require --dev laravel/dusk

Duskをインストールしたら、`Laravel\Dusk\DuskServiceProvider`サービスプロバイダを登録する必要があります。通常、Laravelの自動サービスプロバイダ登録により、自動的に行われます。

> {note} 本番環境にDuskをインストールしてはいけません。インストールすると、アプリケーションに対する未認証でのアクセスを許すようになります。

Duskパッケージをインストールし終えたら、`dusk:install` Artisanコマンドを実行します。

    php artisan dusk:install

`test`ディレクトリ中に、サンプルテストを含んだ`Browser`ディレクトリが作成されます。次に、`.env`ファイルで`APP_URL`環境変数を指定します。この値は、ブラウザからアクセスするアプリケーションで使用するURLと一致させます。

テストを実行するには、`dusk` Artisanコマンドを使います。`dusk`コマンドには、`phpunit`コマンドが受け付ける引数を全て指定できます。

    php artisan dusk

<a name="using-other-browsers"></a>
### 他ブラウザの使用

デフォルトのDuskは、Google Chromeとスタンドアローンの[ChromeDriver](https://sites.google.com/a/chromium.org/chromedriver/home)をブラウザテスト実行に使用します。しかし、自身のSeleniumサーバを起動し、希望するブラウザに対しテストを実行することもできます。

開始するには、アプリケーションのベースDuskテストケースである、`tests/DuskTestCase.php`ファイルを開きます。このファイルの中の、`startChromeDriver`メソッド呼び出しを削除してください。これにより、ChromeDriverの自動起動を停止します。

    /**
     * Duskテスト実行準備
     *
     * @beforeClass
     * @return void
     */
    public static function prepare()
    {
        // static::startChromeDriver();
    }

次に、皆さんが選んだURLとポートへ接続するために、`driver`メソッドを変更します。WebDriverに渡すべき、"desired capabilities"を更新することもできます。

    /**
     * RemoteWebDriverインスタンスの生成
     *
     * @return \Facebook\WebDriver\Remote\RemoteWebDriver
     */
    protected function driver()
    {
        return RemoteWebDriver::create(
            'http://localhost:4444/wd/hub', DesiredCapabilities::phantomjs()
        );
    }

<a name="getting-started"></a>
## 利用の開始

<a name="generating-tests"></a>
### テストの生成

Duskのテストを生成するには、`dusk:make` Artisanコマンドを使います。生成されたテストは、`tests/Browser`ディレクトリへ設置されます。

    php artisan dusk:make LoginTest

<a name="running-tests"></a>
### テストの実行

ブラウザテストを実行するには、`dusk` Artisanコマンドを使用します。

    php artisan dusk

PHPUnitテストランナが通常受け付ける引数は、`dusk`コマンドでも指定できます。たとえば、指定した[グループ](https://phpunit.de/manual/current/en/appendixes.annotations.html#appendixes.annotations.group)のテストのみを実行するなどです。

    php artisan dusk --group=foo

#### ChromeDriverの手動起動

デフォルトのDuskは、ChromeDriverを自動的に起動しようとします。特定のシステムで自動起動しない場合は、`dusk`コマンドを実行する前に手動でChromeDriverを起動することもできます。ChromeDriverを手動起動する場合は、`tests/DuskTestCase.php`ファイルの以下の行をコメントアウトしてください。

    /**
     * Duskテスト実行準備
     *
     * @beforeClass
     * @return void
     */
    public static function prepare()
    {
        // static::startChromeDriver();
    }

また、ChromeDriverを9515以外のポートで起動した場合、同じクラスの`driver`メソッドを変更する必要があります。

    /**
     * RemoteWebDriverインスタンスの生成
     *
     * @return \Facebook\WebDriver\Remote\RemoteWebDriver
     */
    protected function driver()
    {
        return RemoteWebDriver::create(
            'http://localhost:9515', DesiredCapabilities::chrome()
        );
    }

<a name="environment-handling"></a>
### 環境の処理

テスト実行時に独自の環境ファイルを強制的に使用させるには、プロジェクトのルートに`.env.dusk.{environment}`ファイルを作成します。たとえば、`local`環境から`dusk`コマンドを起動する場合は、`.env.dusk.local`ファイルを作成します。

テストを実行すると、Duskは`.env`ファイルをバックアップし、皆さんのDusk環境を`.env`へリネームします。テストが完了したら、`.env`ファイルをリストアします。

<a name="creating-browsers"></a>
### ブラウザの生成

手始めに、アプリケーションへログインできることを確認するテストを書いてみましょう。テストを生成したら、ログインページへ移動し、認証情報を入力し、"Login"ボタンをクリックするように変更します。ブラウザインスタンスを生成するには、`browser`メソッドを呼び出します。

    <?php

    namespace Tests\Browser;

    use App\User;
    use Tests\DuskTestCase;
    use Laravel\Dusk\Chrome;
    use Illuminate\Foundation\Testing\DatabaseMigrations;

    class ExampleTest extends DuskTestCase
    {
        use DatabaseMigrations;

        /**
         * 基本的なブラウザテスト例
         *
         * @return void
         */
        public function testBasicExample()
        {
            $user = factory(User::class)->create([
                'email' => 'taylor@laravel.com',
            ]);

            $this->browse(function ($browser) use ($user) {
                $browser->visit('/login')
                        ->type('email', $user->email)
                        ->type('password', 'secret')
                        ->press('Login')
                        ->assertPathIs('/home');
            });
        }
    }

上記の例のように、`browse`メソッドはコールバックを引数に受けます。　Duskによりブラウザインスタンスは自動的にこのコールバックに渡され、このオブジェクトで、アプリケーションに対する操作やアサートを行います。

> {tip} このテストは、`make:auth` Artisanコマンドにより生成される、ログインスクリーンのテストに使用できます。

#### 複数ブラウザの生成

テストを行うために複数のブラウザが必要なこともあります。たとえば、Webソケットを使用するチャットスクリーンをテストするためには、複数のブラウザが必要でしょう。複数ブラウザを生成するには、単に`browse`メソッドに指定するコールバックの引数で、一つ以上のブラウザを指定するだけです。

    $this->browse(function ($first, $second) {
        $first->loginAs(User::find(1))
              ->visit('/home')
              ->waitForText('Message');

        $second->loginAs(User::find(2))
               ->visit('/home')
               ->waitForText('Message')
               ->type('message', 'Hey Taylor')
               ->press('Send');

        $first->waitForText('Hey Taylor')
              ->assertSee('Jeffrey Way');
    });

#### ブラウザウィンドウのリサイズ

ブラウザウインドウのサイズを調整するため、`resize`メソッドを使用できます。

    $browser->resize(1920, 1080);

ブラウザウィンドウを最大化するには、`maximize`メソッドを使います。

    $browser->maximize();

<a name="authentication"></a>
### 認証

認証が必要なページのテストはよくあります。毎回テストのたびにログインスクリーンを操作しなくても済むように、Duskの`loginAs`メソッドを使ってください。`loginAs`メソッドはユーザーIDかユーザーモデルインスタンスを引数に取ります。

    $this->browse(function ($first, $second) {
        $first->loginAs(User::find(1))
              ->visit('/home');
    });

> {note} `loginAs`メソッドを使用後、そのファイルに含まれるすべてのテストに対し、ユーザーセッションは保持されます。

<a name="interacting-with-elements"></a>
## 要素の操作

<a name="dusk-selectors"></a>
### Duskセレクタ

要素を操作するために、最適なCSSセレクタを選択するのは、Duskテストで一番難しい部分です。フロントエンドは繰り返し変更され、失敗するようになったテストを修正するため、CSSセレクタを何度も調整しました。

    // HTML

    <button>Login</button>

    // テスト

    $browser->click('.login-page .container div > button');

Duskセレクタにより、CSSセレクタを記憶するのではなく、効率的にテストを書くことに集中できるようになります。セレクタを定義するには、HTMLエレメントに`dusk`属性を追加します。それから、Duskテスト中の要素を操作するために、セレクタの先頭に`@`を付けてください。

    // HTML

    <button dusk="login-button">Login</button>

    // テスト

    $browser->click('@login-button');

<a name="clicking-links"></a>
### リンクのクリック

リンクをクリックするには、ブラウザインスタンスの`clickLink`メソッドを使います。`clickLink`メソッドは指定した表示テキストのリンクをクリックします。

    $browser->clickLink($linkText);

> {note} このメソッドはjQueryを操作します。jQueryがそのページで使用不可能な場合、Duskは自動的にそのページへ挿入し、テストの中に使用します。

<a name="text-values-and-attributes"></a>
### テキスト、値、属性

#### 値の取得／設定

Duskは現在表示されているテキスト、値、ページ要素の属性を操作する、数多くのメソッドを提供します。たとえば、指定したセレクタに一致する要素の「値(value)」を取得するには、`value`メソッドを使用します。

    // 値の取得
    $value = $browser->value('selector');

    // 値の設定
    $browser->value('selector', 'value');

#### テキストの取得

`text`メソッドは、指定したセレクタに一致する要素の表示テキストを取得します。

    $text = $browser->text('selector');

#### 属性の取得

最後の`attribute`メソッドは、セレクタに一致する要素の属性を取得します。

    $attribute = $browser->attribute('selector', 'value');

<a name="using-forms"></a>
### フォームの使用

#### 値のタイプ

Duskはフォームと入力要素を操作する、様々なメソッドを提供しています。最初に、入力フィールドへテキストをタイプする例を見てみましょう。

    $browser->type('email', 'taylor@laravel.com');

必要であれば受け付けますが、`type`メソッドにはCSSセレクタを渡す必要がないことに注意してください。CSSセレクタが指定されない場合、Duskは`name`属性に指定された入力フィールドを探します。最終的に、Duskは指定された`name`属性を持つ`textarea`を見つけようとします。

コンテンツをクリアせずに、フィールドへテキストを追加するには、`append`メソッドを使用します。

    $browser->type('tags', 'foo')
            ->append('tags', ', bar, baz');

入力値をクリアするには、`clear`メソッドを使用します。

    $browser->clear('email');

#### ドロップダウン

ドロップダウンの選択ボックスから値を選ぶには、`select`メソッドを使います。`type`メソッドと同様に、`select`メソッドも完全なCSSセレクタは必要ありません。`select`メソッドに引数を指定するとき、表示テキストの代わりに、オプション値を渡します。

    $browser->select('size', 'Large');

第２引数を省略した場合、ランダムにオプションを選択します。

    $browser->select('size');

#### チェックボックス

チェックボックスを「チェック(check)」するには、`check`メソッドを使います。他の関連する多くのメソッドと同様に、完全なCSSセレクタは必要ありません。完全に一致するセレクタが見つからないと、Duskは`name`属性に一致するチェックボックスを探します。

    $browser->check('terms');

    $browser->uncheck('terms');

#### ラジオボタン

ラジオボタンのオプションを「選択」するには、`radio`メソッドを使用します。他の関連する多くのメソッドと同様に、完全なセレクタは必要ありません。完全に一致するセレクタが見つからない場合、Duskは`name`と`value`属性に一致するラジオボタンを探します。

    $browser->radio('version', 'php7');

<a name="attaching-files"></a>
### 添付ファイル

`attach`メソッドは`file`入力要素で、ファイルを指定するために使用します。他の関連する入力メソッドと同様に、完全なCSSセレクタは必要ありません。完全なセレクタが見つからなければ、Duskは`name`属性と一致するファイル入力を探します。

    $browser->attach('photo', __DIR__.'/photos/me.png');

<a name="using-the-keyboard"></a>
### キーワードの使用

`keys`メソッドは、`type`メソッドによる、指定した要素に対する通常の入力よりも、複雑な入力を提供します。たとえば、モデファイヤキーを押しながら、値を入力するなどです。以下の例では、指定したセレクタに一致する要素へ、`taylor`を「シフト(`shift`)」キーを押しながら入力します。`Taylor`をタイプし終えると、`otwell`がモデファイヤキーを押さずにタイプされます。

    $browser->keys('selector', ['{shift}', 'taylor'], 'otwell');

アプリケーションを構成する主要なCSSセレクタへ「ホットキー」を送ることもできます。

    $browser->keys('.app', ['{command}', 'j']);

> {tip} モデファイヤキーは`{}`文字で囲み、`Facebook\WebDriver\WebDriverKeys`クラスで定義されている定数を指定します。[GitHubで確認](https://github.com/facebook/php-webdriver/blob/community/lib/WebDriverKeys.php)できます。

<a name="using-the-mouse"></a>
### マウスの使用

#### 要素のクリック

指定したセレクタに一致する要素を「クリック」するには、`click`メソッドを使います。

    $browser->click('.selector');

#### マウスオーバー

指定したセレクタに一致する要素を「マウスオーバー」したい場合は、`mouseover`メソッドを使います。

    $browser->mouseover('.selector');

#### ドラッグ＆ドロップ

`drag`メソッドは指定したセレクタに一致する要素をドラッグし、もう一つのエレメントへドロップします。

    $browser->drag('.from-selector', '.to-selector');

もしくは、特定の方向へ要素をドラッグすることもできます。

    $browser->dragLeft('.selector', 10);
    $browser->dragRight('.selector', 10);
    $browser->dragUp('.selector', 10);
    $browser->dragDown('.selector', 10);

<a name="scoping-selectors"></a>
### セレクタの範囲指定

特定のセレクタの中の全操作を範囲指定しつつ、多くの操作を行いたいこともあります。たとえば、いくつかのテーブル中にあるテキストが存在していることをアサートし、それからテーブル中のボタンをクリックしたい場合です。`with`メソッドで行なえます。`with`メソッドのコールバック中で行われた操作は全部、オリジナルのセレクタに対し限定されます。

    $browser->with('.table', function ($table) {
        $table->assertSee('Hello World')
              ->clickLink('Delete');
    });

<a name="waiting-for-elements"></a>
### 要素の待機

広範囲に渡りJavaScriptを使用しているアプリケーションのテストでは、テストを進める前に特定の要素やデータが利用可能になるまで、「待つ(wait)」必要がしばしば起きます。Duskではこれも簡単に行えます。数多くのメソッドを使い、ページで要素が見えるようになるまで、もしくはJavaScriptの評価が`true`になるまで待機できます。

#### 待機

指定したミリ秒の間、テストをポーズしたい場合は、`pause`メソッドを使用します。

    $browser->pause(1000);

#### セレクタの待機

`waitFor`メソッドはテストの実行を指定したCSSセレクタがページに表示されるまで中断します。例外が投げられるまで、デフォルトで最大５秒間テストを中断します。必要であれば、カスタムタイムアウトを秒でメソッドの第２引数として指定できます。

    // セレクタを最大５秒間待つ
    $browser->waitFor('.selector');

    // セレクタを最大１秒待つ
    $browser->waitFor('.selector', 1);

指定したセレクタがページから消えるまで待つこともできます。

    $browser->waitUntilMissing('.selector');

    $browser->waitUntilMissing('.selector', 1);

#### 利用可能時限定のセレクタ

指定したセレクタを待ち、それからそのセレクタに一致する要素を操作したい場合もよくあります。たとえば、モーダルウィンドウが現れるまで待ち、それからそのモーダルウィンドウ上の"OK"ボタンを押したい場合です。このケースでは`whenAvailable`メソッドを使用します。指定したコールバック内で行われた要素操作は全て、オリジナルのセレクタに対して限定されます。

    $browser->whenAvailable('.modal', function ($modal) {
        $modal->assertSee('Hello World')
              ->press('OK');
    });

#### テキストの待機

指定したテキストがページに表示されるまで待ちたい場合は、`waitForText`メソッドを使います。

    // テキストを最大５秒間待つ
    $browser->waitForText('Hello World');

    // テキストを最大１秒待つ
    $browser->waitForText('Hello World', 1);

#### リンクの待機

ページに指定したリンクテキストが表示されるまで待つ場合は、`waitForLink`メソッドを使います。

    // リンクを最大５秒間待つ
    $browser->waitForLink('Create');

    // リンクを最大１秒間待つ
    $browser->waitForLink('Create', 1);

#### ページロケーションの待機

`$browser->assertPathIs('/home')`のようなパスをアサートするときに、`window.location.pathname`が非同期更新中の場合、アサートは失敗するでしょう。指定値のロケーションを待機するために、`waitForLocation`メソッドを使ってください。

    $browser->waitForLocation('/secret');

#### ページリロードの待機

ページのリロード後にアサートする必要がある場合は、`waitForReload`メソッドを使ってください。

    $browser->click('.some-action')
            ->waitForReload()
            ->assertSee('something');

#### JavaScriptの評価の待機

指定したJavaScript式の評価が`true`になるまで、テストの実行を中断したい場合も時々あります。`waitUntil`メソッドで簡単に行えます。このメソッドに式を渡す時に、`return`キーワードや最後のセミコロンを含める必要はありません。

    // 式がtureになるまで最大５秒間待つ
    $browser->waitUntil('App.dataLoaded');

    $browser->waitUntil('App.data.servers.length > 0');

    // 式がtureになるまで最大１秒間待つ
    $browser->waitUntil('App.data.servers.length > 0', 1);

#### コールバックによる待機

Duskにある数多くの「待機」メソッドは、`waitUsing`メソッドを使用しています。このメソッドを直接利用し、コールバックが`true`を返すまで待機することができます。`waitUsing`メソッドは最長待ち秒数とクロージャを評価する間隔秒数、クロージャを引数に取ります。オプションとして、失敗時のメッセージを引数に取ります。

    $browser->waitUsing(10, 1, function () use ($something) {
        return $something->isReady();
    }, "Something wasn't ready in time.");

<a name="making-vue-assertions"></a>
### Vueアサーションの作成

Duskでは、[Vue](https://vuejs.org)コンポーネントデータの状態をアサートすることもできます。たとえば、アプリケーションに以下のVueコンポーネントが含まれていると想像してください。

    // HTML

    <profile dusk="profile-component"></profile>

    // コンポーネント定義

    Vue.component('profile', {
        template: '<div>{{ user.name }}</div>',

        data: function () {
            return {
                user: {
                  name: 'Taylor'
                }
            };
        }
    });

Vueコンポーネントの状態を以下のようにアサートできます。

    /**
     * 基本的なVueのテスト
     *
     * @return void
     */
    public function testVue()
    {
        $this->browse(function (Browser $browser) {
            $browser->visit('/')
                    ->assertVue('user.name', 'Taylor', '@profile-component');
        });
    }

<a name="available-assertions"></a>
## 使用可能なアサート

Duskはアプリケーションに対する数多くのアサートを提供しています。使用できるアサートを次の表にまとめます。

アサート  |  説明
----------|----------
`$browser->assertTitle($title)`  |  ページタイトルが指定したテキストであることをアサートする。
`$browser->assertTitleContains($title)`  |  ページタイトルに指定したテキストが含まれることをアサートする。
`$browser->assertPathBeginsWith($path)`  |  現在のURLが指定したパスで始まることをアサートする。
`$browser->assertPathIs('/home')`  |  現在のパスが指定したパスと一致することをアサートする。
`$browser->assertPathIsNot('/home')`  |  現在のパスが指定したパスと一致しないことをアサートする。
`$browser->assertRouteIs($name, $parameters)`  |  現在のURLが、指定した名前付きルートのURLと一致することをアサートする。
`$browser->assertQueryStringHas($name, $value)`  |  指定したクエリパラメータが存在し、指定した値であることをアサートする。
`$browser->assertQueryStringMissing($name)`  |  指定したクエリ文字列パラメータが存在しないことをアサートする。
`$browser->assertHasQueryStringParameter($name)`  |  指定したクエリパラータが存在することをアサートする。
`$browser->assertHasCookie($name)`  |  指定したクッキーが存在することをアサートする。
`$browser->assertCookieMissing($name)`  |  指定したクッキーが存在していないことをアサートする。
`$browser->assertCookieValue($name, $value)`  |  クッキーが指定した値であることをアサートする。
`$browser->assertPlainCookieValue($name, $value)`  |  暗号化されていないクッキーが指定した値であることをアサートする。
`$browser->assertSee($text)`  |  ページに指定したテキストが存在することをアサートする。
`$browser->assertDontSee($text)`  |  ページに指定したテキストが存在しないことをアサートする。
`$browser->assertSeeIn($selector, $text)`  |  セレクタの中に指定したテキストが存在することをアサートする。
`$browser->assertDontSeeIn($selector, $text)`  |  セレクタの中に指定したテキストが存在していないことをアサートする。
`$browser->assertSourceHas($code)`  |  ページに指定したソースコードが存在することをアサートする。
`$browser->assertSourceMissing($code)`  |  ページに指定したソースコードが存在しないことをアサートする。
`$browser->assertSeeLink($linkText)`  |  ページに指定したリンクが存在することをアサートする。
`$browser->assertDontSeeLink($linkText)`  |  ページに指定したリンクが存在しないことをアサートする。
`$browser->assertInputValue($field, $value)`  |  指定した入力フィールドが指定した値であることをアサートする。
`$browser->assertInputValueIsNot($field, $value)`  |  指定した入力フィールドが指定した値でないことをアサートする。
`$browser->assertChecked($field)`  |  指定したチェックボックスがチェック済みであることをアサートする。
`$browser->assertNotChecked($field)`  |  指定したチェックボックスがチェックされていないことをアサートする。
`$browser->assertRadioSelected($field, $value)`  |  指定したラジオフィールドが選択されていることをアサートする。
`$browser->assertRadioNotSelected($field, $value)` |  指定したラジオフィールドが選択されていないことをアサートする。
`$browser->assertSelected($field, $value)`  |  指定したドロップダウンの指定値が選択されていることをアサートする。
`$browser->assertNotSelected($field, $value)`  |  指定したドロップダウンの指定値が選択されていないことをアサートする。
`$browser->assertSelectHasOptions($field, $values)`  |  指定した配列の値が、選択可能であることをアサートする。
`$browser->assertSelectMissingOptions($field, $values)`  |  指定した配列の値が、選択不可能であることをアサートする。
`$browser->assertSelectHasOption($field, $value)`  |  指定した値が、指定フィールドで選択可能であることをアサートする。
`$browser->assertValue($selector, $value)`  |  指定したセレクタに一致する要素が、指定値であることをアサートする。
`$browser->assertVisible($selector)`  |  指定したセレクタに一致する要素がビジブルであることをアサートする。
`$browser->assertMissing($selector)`  |  指定したセレクタに一致する要素がビジブルでないことをアサートする。
`$browser->assertDialogOpened($message)`  |  指定したメッセージを表示するJavaScriptダイアログが開かれていることをアサートする。
`$browser->assertVue($property, $value, $component)`  |  指定したVueコンポーネントのデータプロパティが、指定した値と一致することをアサートする。
`$browser->assertVueIsNot($property, $value, $component)`  |  指定したVueコンポーネントのデータプロパティが、指定した値と一致しないことをアサートする。

<a name="pages"></a>
## ページ

時にテストで、連続して実行する複雑なアクションをたくさん要求されることがあります。これにより、テストは読みづらく、また理解しづらくなります。ページに対し一つのメソッドを使うだけで、指定ページで実行されるアクションを記述的に定義できます。ページはまた、アプリケーションやシングルページで一般的なセレクタの、短縮記法を定義する方法も提供しています。

<a name="generating-pages"></a>
### ページの生成

ページオプジェクトを生成するには、`dusk:page` Artisanコマンドを使います。全てのページオブジェクトは、`tests/Browser/Pages`ディレクトリへ設置します。

    php artisan dusk:page Login

<a name="configuring-pages"></a>
### ページの設定

デフォルトでページには、`url`、`assert`、`elements`の３メソッドが用意されています。`url`と`assert`メソッドは、この後説明します。`elements`メソッドについては、[のちほど詳細を紹介](#shorthand-selectors)します。

#### `url`メソッド

`url`メソッドでは、そのページを表すURLのパスを返します。Duskはブラウザでこのページへ移動するとき、このURLを使用します。

    /**
     * このページのURL取得
     *
     * @return string
     */
    public function url()
    {
        return '/login';
    }

#### `assert`メソッド

`assert`メソッドでは、ブラウザが実際に指定ページを表示した時に、確認が必要なアサーションを定義します。このメソッドで完全に行う必要はありません。ですが、もしお望みであれば自由にアサートを記述してください。記述されたアサートは、このページへ移行時に自動的に実行されます。

    /**
     * ブラウザがこのページにやって来たときのアサート
     *
     * @return void
     */
    public function assert(Browser $browser)
    {
        $browser->assertPathIs($this->url());
    }

<a name="navigating-to-pages"></a>
### ページへのナビゲーション

ページの設定を終えたら、`visit`メソッドを使い、ページへ移行できます。

    use Tests\Browser\Pages\Login;

    $browser->visit(new Login);

既に特定のページに移動済みで、現在のテストコンテキストへそのページのセレクタとメソッドを「ロード」する必要が起きることがあります。この状況は、明示的に移動していなくても、あるボタンを押すことで指定ページへリダイレクトしてしまう場合に発生します。そうした場合は、`on`メソッドで、そのページをロードできます。

    use Tests\Browser\Pages\CreatePlaylist;

    $browser->visit('/dashboard')
            ->clickLink('Create Playlist')
            ->on(new CreatePlaylist)
            ->assertSee('@create');

<a name="shorthand-selectors"></a>
### セレクタの簡略記述

ページの`elements`メソッドにより、覚えやすいCSSセレクタの短縮形を素早く定義できます。例として、アプリケーションのログインページの"email"入力フィールドの短縮形を定義してみましょう。

    /**
     * ページ要素の短縮形を取得
     *
     * @return array
     */
    public function elements()
    {
        return [
            '@email' => 'input[name=email]',
        ];
    }

これで、完全なCSSセレクタを指定する箇所ならどこでも、この短縮セレクタを使用できます。

    $browser->type('@email', 'taylor@laravel.com');

#### グローバルなセレクタ簡略記述

Duskをインストールすると、ベース`Page`クラスが`tests/Browser/Pages`ディレクトリへ設置されます。このクラスは、アプリケーション全部のどのページからでも利用可能な、グローバル短縮セレクタを定義する`siteElements`メソッドを含んでいます。

    /**
     * サイトのグローバル要素短縮形の取得
     *
     * @return array
     */
    public static function siteElements()
    {
        return [
            '@element' => '#selector',
        ];
    }

<a name="page-methods"></a>
### ページメソッド

ページに対し定義済みのデフォルトメソッドに加え、テスト全体で使用できる追加メソッドも定義できます。たとえば、音楽管理アプリケーションを構築中だと想像してみましょう。アプリケーションのあるページでプレイリストを作成するのは、よくあるアクションです。各テストごとにプレイリスト作成のロジックを書き直す代わりに、ページクラスに`createPlaylist`メソッドを定義することができます。

    <?php

    namespace Tests\Browser\Pages;

    use Laravel\Dusk\Browser;

    class Dashboard extends Page
    {
        // 他のページメソッドの定義…

        /**
         * 新しいプレイリストの作成
         *
         * @param  \Laravel\Dusk\Browser  $browser
         * @param  string  $name
         * @return void
         */
        public function createPlaylist(Browser $browser, $name)
        {
            $browser->type('name', $name)
                    ->check('share')
                    ->press('Create Playlist');
        }
    }

メソッドを定義すれば、このページを使用するすべてのテストの中で使用できます。ブラウザインスタンスは自動的にページメソッドへ渡されます。

    use Tests\Browser\Pages\Dashboard;

    $browser->visit(new Dashboard)
            ->createPlaylist('My Playlist')
            ->assertSee('My Playlist');

<a name="components"></a>
## コンポーネント

コンポーネントはDuskの「ページオブジェクト」と似ていますが、ナビゲーションバーや通知ウィンドウのような、UI群と機能をアプリケーション全体で再利用するためのものです。コンポーネントは特定のURLと結びついていません。

<a name="generating-components"></a>
### コンポーネント生成

コンポーネントを生成するには、`dusk:component` Artisanコマンドを使用します。新しいコンポーネントは、`test/Browser/Components`ディレクトリに設置されます。

    php artisan dusk:component DatePicker

上記の「デートピッカー」は、アプリケーション全体の様々なページで利用されるコンポーネントの一例です。テストスーツ全体の何ダースものテスト中で、日付を選択するブラウザ自動化ロジックを一々書くのは大変な手間です。その代わりに、デートピッカーを表すDuskコンポーネントを定義し、そうしたロジックをコンポーネントへカプセル化することができます。

    <?php

    namespace Tests\Browser\Components;

    use Laravel\Dusk\Browser;
    use Laravel\Dusk\Component as BaseComponent;

    class DatePicker extends BaseComponent
    {
        /**
         * コンポーネントのルートセレクタ取得
         *
         * @return string
         */
        public function selector()
        {
            return '.date-picker';
        }

        /**
         * ブラウザページにそのコンポーネントが含まれていることをアサート
         *
         * @param  Browser  $browser
         * @return void
         */
        public function assert(Browser $browser)
        {
            $browser->assertVisible($this->selector());
        }

        /**
         * コンポーネントの要素のショートカットを取得
         *
         * @return array
         */
        public function elements()
        {
            return [
                '@date-field' => 'input.datepicker-input',
                '@month-list' => 'div > div.datepicker-months',
                '@day-list' => 'div > div.datepicker-days',
            ];
        }

        /**
         * 指定日付のセレクト
         *
         * @param  \Laravel\Dusk\Browser  $browser
         * @param  int  $month
         * @param  int  $year
         * @return void
         */
        public function selectDate($browser, $month, $year)
        {
            $browser->click('@date-field')
                    ->within('@month-list', function ($browser) use ($month) {
                        $browser->click($month);
                    })
                    ->within('@day-list', function ($browser) use ($day) {
                        $browser->click($day);
                    });
        }
    }

<a name="using-components"></a>
### コンポーネントの使用

コンポーネントを定義したら、全テスト中からデートピッカーの中の指定日付を簡単にセレクトできます。日付選択に必要なロジックに変更が起きたら、このコンポーネントを更新するだけです。

    <?php

    namespace Tests\Browser;

    use Tests\DuskTestCase;
    use Laravel\Dusk\Browser;
    use Tests\Browser\Components\DatePicker;
    use Illuminate\Foundation\Testing\DatabaseMigrations;

    class ExampleTest extends DuskTestCase
    {
        /**
         * 基本的なコンポーネントテスト例
         *
         * @return void
         */
        public function testBasicExample()
        {
            $this->browse(function (Browser $browser) {
                $browser->visit('/')
                        ->within(new DatePicker, function ($browser) {
                            $browser->selectDate(1, 2018);
                        })
                        ->assertSee('January');
            });
        }
    }

<a name="continuous-integration"></a>
## 継続的インテグレーション

<a name="running-tests-on-travis-ci"></a>
### Travis CI

Travis CI上でDuskテストを実行するためには、Ubuntu 14.04 (Trusty)環境を"sudo-enabled"で使用する必要があります。Travis CIはグラフィカルな環境ではないため、Chromeブラウザを実行するには追加の手順を行う必要があります。さらに、PHPの組み込みWebサーバを起動するために、`php artisan serve`を使用する必要もあるでしょう。

    sudo: required
    dist: trusty

    addons:
       chrome: stable

    install:
       - cp .env.testing .env
       - travis_retry composer install --no-interaction --prefer-dist --no-suggest

    before_script:
       - google-chrome-stable --headless --disable-gpu --remote-debugging-port=9222 http://localhost &
       - php artisan serve &

    script:
       - php artisan dusk

<a name="running-tests-on-circle-ci"></a>
### CircleCI

#### CircleCI 1.0

CircleCI1.0を使用し、Duskテストを実行する場合、以下の設定ファイルを手始めに利用できます。TravisCIと同様に、`php artisan serve`コマンドを使用し、PHP組み込みWebサーバを起動できます。

	dependencies:
	  pre:
	      - curl -L -o google-chrome.deb https://dl.google.com/linux/direct/google-chrome-stable_current_amd64.deb
	      - sudo dpkg -i google-chrome.deb
	      - sudo sed -i 's|HERE/chrome\"|HERE/chrome\" --disable-setuid-sandbox|g' /opt/google/chrome/google-chrome
	      - rm google-chrome.deb

    test:
        pre:
            - "./vendor/laravel/dusk/bin/chromedriver-linux":
                background: true
            - cp .env.testing .env
            - "php artisan serve":
                background: true

        override:
            - php artisan dusk

 #### CircleCI 2.0

DuskテストにCircleCI2.0を使用する場合、ビルドに以下のステップを追加してください。

     version: 2
     jobs:
         build:
             steps:
                - run: sudo apt-get install -y libsqlite3-dev
                - run: cp .env.testing .env
                - run: composer install -n --ignore-platform-reqs
                - run: npm install
                - run: npm run production
                - run: vendor/bin/phpunit

                - run:
                   name: Start Chrome Driver
                   command: ./vendor/laravel/dusk/bin/chromedriver-linux
                   background: true

                - run:
                   name: Run Laravel Server
                   command: php artisan serve
                   background: true

                - run:
                   name: Run Laravel Dusk Tests
                   command: php artisan dusk

<a name="running-tests-on-codeship"></a>
### Codeship

Duskのテストを[Codeship](https://codeship.com)で実行するには、以下のコマンドをCodeshipプロジェクトへ追加してください。もちろん、以下のコマンドはシンプルな参考例ですので、必要に応じ自由にコマンドを追加してください。

    phpenv local 7.1
    cp .env.testing .env
    composer install --no-interaction
    nohup bash -c "./vendor/laravel/dusk/bin/chromedriver-linux 2>&1 &"
    nohup bash -c "php artisan serve 2>&1 &" && sleep 5
    php artisan dusk
