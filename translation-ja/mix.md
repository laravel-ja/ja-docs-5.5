# アセットのコンパイル(Laravel Mix)

- [イントロダクション](#introduction)
- [インストールと準備](#installation)
- [Mixの実行](#running-mix)
- [スタイルシートの操作](#working-with-stylesheets)
    - [Less](#less)
    - [Sass](#sass)
    - [Stylus](#stylus)
    - [PostCSS](#postcss)
    - [平文CSS](#plain-css)
    - [URL処理](#url-processing)
    - [ソースマップ](#css-source-maps)
- [JavaScriptの操作](#working-with-scripts)
    - [ベンダの抽出](#vendor-extraction)
    - [React](#react)
    - [バニラJS](#vanilla-js)
    - [Webpackカスタム設定](#custom-webpack-configuration)
- [ファイル／ディレクトリコピー](#copying-files-and-directories)
- [バージョン付け／キャッシュ対策](#versioning-and-cache-busting)
- [Browsersyncリロード](#browsersync-reloading)
- [環境変数](#environment-variables)
- [通知](#notifications)

<a name="introduction"></a>
## イントロダクション

[Laravel Mix](https://github.com/JeffreyWay/laravel-mix)は多くの一般的なCSSとJavaScriptのプリプロセッサを使用し、Laravelアプリケーションために、構築過程をWebpackでスラスラと定義できるAPIを提供しています。シンプルなメソッドチェーンを使用しているため、アセットパイプラインを流暢に定義できます。例を見てください。

    mix.js('resources/assets/js/app.js', 'public/js')
       .sass('resources/assets/sass/app.scss', 'public/css');

Webpackやアセットのコンパイルを始めようとして、混乱と圧倒を感じているならLaravel Mixを気に入ってもらえるでしょう。しかし、アプリケーションの開発時に必要だというわけではありません。どんなアセットパイプラインツールを使用してもかまいませんし、使わなくても良いのです。

<a name="installation"></a>
## インストールと準備

#### Nodeのインストール

Mixを始める前、最初にNode.jsとNPMを開発機へ確実にインストールしてください。

    node -v
    npm -v

Laravel Homesteadならデフォルトのままでも必要なものが全部そろっています。しかし、Vagrantを使っていなくても、[ダウンロードページ](https://nodejs.org/ja/download/)を読めば、NodeとNPMは簡単にインストールできます。

#### Laravel Mix

残っているステップはLaravel Mixのインストールだけです。新しくLaravelをインストールすると、ルートディレクトリに`package.json`があることに気づくでしょう。PHPの代わりにNodeの依存パッケージが定義されている所が異なりますが、`composer.json`ファイルと同じようなものだと考えてください。以下のコマンドで、依存パッケージをインストールしてください。

    npm install

<a name="running-mix"></a>
## Mixの実行

Mixは[Webpack](https://webpack.js.org)上の設定レイヤーですから、Laravelに含まれる`package.json`ファイル上のNPMスクリプトの一つをMixの実行で起動してください。

    // 全タスク実行
    npm run dev

    // 全タスク実行を実行し、出力を圧縮
    npm run production

#### アセット変更の監視

`npm run watch`コマンドはターミナルで実行し続け、関連ファイル全部の変更を監視します。Webpackは変更を感知すると、アセットを自動的に再コンパイルします。

    npm run watch

特定の環境のWebpackでは、ファイル変更時に更新されないことがあります。自分のシステムでこれが起きた場合は、`watch-poll`コマンドを使用してください。

    npm run watch-poll

<a name="working-with-stylesheets"></a>
## スタイルシートの操作

`webpack.mix.js`ファイルは全アセットコンパイルのエントリポイントです。Webpackの軽い設定ラッパーだと考えてください。Mixタスクはアセットをどのようにコンパイルすべきかを正確に定義するため、チェーンでつなげます。

<a name="less"></a>
### Less

[Less](http://lesscss.org/)をCSSへコンパイルするには`less`メソッドを使用します。最も主要な`app.less`ファイルを`public/css/app.css`としてコンパイルしてみましょう。

    mix.less('resources/assets/less/app.less', 'public/css');

複数のファイルをコンパイルするには、`less`メソッドを複数回呼び出します。

    mix.less('resources/assets/less/app.less', 'public/css')
       .less('resources/assets/less/admin.less', 'public/css');

コンパイル済みのCSSのファイル名をカスタマイズしたい場合は、`less`の第２引数にファイルのフルパスを指定してください。

    mix.less('resources/assets/less/app.less', 'public/stylesheets/styles.css');

[裏で動作しているLessプラグインのオプション](https://github.com/webpack-contrib/less-loader#options)をオーバーライドする必要が起きたら、`mix.less()`の第３引数にオブジェクトを渡してください。

    mix.less('resources/assets/less/app.less', 'public/css', {
        strictMath: true
    });

<a name="sass"></a>
### Sass

`sass`メソッドは、[Sass](http://sass-lang.com/)をCSSへコンパイルします。次のように使用します。

    mix.sass('resources/assets/sass/app.scss', 'public/css');

`less`メソッドでも、複数のSassファイルを別々のCSSファイルへコンパイルできますし、結果のCSSの出力ディレクトリをカスタマイズ可能です。

    mix.sass('resources/assets/sass/app.sass', 'public/css')
       .sass('resources/assets/sass/admin.sass', 'public/css/admin');

さらに、[Node-Sassプラグインオプション](https://github.com/sass/node-sass#options)を第３引数に指定できます。

    mix.sass('resources/assets/sass/app.sass', 'public/css', {
        precision: 5
    });

<a name="stylus"></a>
### Stylus

LessやSassと同様に、`stylus`メソッドにより、[Stylus](http://stylus-lang.com/)をCSSへコンパイルできます。

    mix.stylus('resources/assets/stylus/app.styl', 'public/css');

さらに、[Rupture](https://github.com/jescalan/rupture)のような、追加のStylusプラグインをインストールすることもできます。最初に、NPM （`npm install rupture`）による質問でプラグインをインストールし、それから`mix.stylus()`の中で呼び出してください。

    mix.stylus('resources/assets/stylus/app.styl', 'public/css', {
        use: [
            require('rupture')()
        ]
    });

<a name="postcss"></a>
### PostCSS

強力なCSS加工ツールである[PostCSS](http://postcss.org/)も、Laravel Mixには最初から含まれています。デフォルトでは、自動的に必要なCSS3ベンダープレフィックスを適用する、人気の[Autoprefixer](https://github.com/postcss/autoprefixer)プラグインを利用します。しかし、アプリケーションに適したプラグインを自由に追加できます。最初に、NPMにより希望のプラグインをインストールし、それから`webpack.mix.js`の中から参照してください。

    mix.sass('resources/assets/sass/app.scss', 'public/css')
       .options({
            postCss: [
                require('postcss-css-variables')()
            ]
       });

<a name="plain-css"></a>
### 平文CSS

平文のCSSスタイルシートを一つのファイルへ結合したい場合は、`styles`メソッドを使って下さい。

    mix.styles([
        'public/css/vendor/normalize.css',
        'public/css/vendor/videojs.css'
    ], 'public/css/all.css');

<a name="url-processing"></a>
### URL処理

Laravel MixはWebpack上に構築されているため、Webpackのコンセプトを理解することが重要です。CSSコンパイルのため、Webpackはスタイルシート中の`url()`呼び出しをリライトし、最適化します。最初は奇妙に聴こえるかもしれませんが、これは非常に強力な機能の一部です。画像への相対URLを含んだSassをコンパイルするのを想像してください。

    .example {
        background: url('../images/example.png');
    }

> {note} 絶対パスを`url()`へ指定しても、URL書き換えの対象外になります。たとえば、`url('/images/thing.png')`や、`url('http://example.com/images/thing.png')`は変更されません。

デフォルト動作として、Laravel MixとWebpackが`example.png`を見つけると、それを`public/images`フォルダへコピーします。それから、生成したスタイルシート中の`url()`を書き換えます。

    .example {
      background: url(/images/example.png?d41d8cd98f00b204e9800998ecf8427e);
    }

この機能は便利ですが、好きなようにフォルダ構造を設定することもできます。その場合、以下のように`url()`リライトを停止してください。

    mix.sass('resources/assets/app/app.scss', 'public/css')
       .options({
          processCssUrls: false
       });

これを`webpack.mix.js`ファイルへ追加することで、Mixはどんな`url()`に一致することも、publicディレクトリへアセットをコピーすることもしなくなります。言い換えれば、コンパイル済みのCSSは、みなさんがタイプした内容そのままに見えるでしょう。

    .example {
        background: url("../images/thing.png");
    }

<a name="css-source-maps"></a>
### ソースマップ

デフォルトでは無効になっているため、`webpack.mix.js`ファイルで`mix.sourceMaps()`を呼び出すことで、ソースマップは有効になります。コンパイルコストと実行パフォーマンスコストはかかりますが、これによりコンパイル済みアセットを使用する時に、ブラウザの開発ツール向けの追加デバッグ情報が提供されます。

    mix.js('resources/assets/js/app.js', 'public/js')
       .sourceMaps();

<a name="working-with-scripts"></a>
## JavaScriptの操作

MixはECMAScript 2015のコンパイル、モジュール結合、圧縮やJavaScriptファイルの結合などの操作を手助けする、多くの機能を提供しています。それだけでなく、設定をカスタマイズする必要は全く無く、全てがシームレスに動作します。

    mix.js('resources/assets/js/app.js', 'public/js');

このコード一行で、以下の利点を享受できます。

<div class="content-list" markdown="1">
- ES2015記法
- モジュール
- `.vue`ファイルのコンパイル
- 開発環境向けに圧縮
</div>

<a name="vendor-extraction"></a>
### ベンダの抽出

全ベンダーライブラリーをアプリケーション固有のJavaScriptとして結合することにより、発生する可能性がある欠点は、長期間のキャッシュが効きづらくなることです。たとえば、アプリケーションコードの一箇所を更新すれば、変更のないベンダーライブラリーの全てを再度ダウンロードするように、ブラウザに共用することになります。

アプリケーションのJavaScriptを頻繁に更新したい場合は、全ベンダーライブラリを専用のファイルへ分ける方法を考慮する必要があります。これにより、アプリケーションコードの変更は、大きな`vendor.js`ファイルのキャッシュへ影響しなくなります。Mixの`extract`メソッドで簡単に実現できます。

    mix.js('resources/assets/js/app.js', 'public/js')
       .extract(['vue'])

`extract`メソッドは全ライブラリとモジュールの配列を受け取り、`vendor.js`へ別にまとめます。上記のコードを例にすると、Mix配下のファイルを生成します。

<div class="content-list" markdown="1">
- `public/js/manifest.js`: **Webpackマニフェストランタイム**
- `public/js/vendor.js`: **ベンダーライブラリ**
- `public/js/app.js`: **アプリケーションコード**
</div>

JavaScriptエラーを起こさないために、以下のファイルは順番通りにロードしてください。

    <script src="/js/manifest.js"></script>
    <script src="/js/vendor.js"></script>
    <script src="/js/app.js"></script>

<a name="react"></a>
### React

MixはReactをサポートするために、Babelプラグインを自動的にインストールします。使用開始するためには、`mix.js()`の呼び出しを`mix.react()`に置換してください。

    mix.react('resources/assets/js/app.jsx', 'public/js');

実際には、Mixは最適なBabelプラグインとして`babel-preset-react`をダウンロードし、取り込んでいます。

<a name="vanilla-js"></a>
### バニラJS

スタイルシートを`mix.styles()`により結合するのと同様に、多くのJavaScriptファイルを`scripts()`メソッドで結合し、圧縮できます。

    mix.scripts([
        'public/js/admin.js',
        'public/js/dashboard.js'
    ], 'public/js/all.js');

この選択肢は特にWebpackによる操作を必要としないレガシープロジェクトに便利です。

> {tip} `mix.scripts()`のちょっとしたバリエーションとして`mix.babel()`があります。このメソッドは`scripts`と同じ使い方です。しかし、結合したファイルはBabelにより編集され、ES2015コードを全てのブラウザーが理解できるバニラJavaScriptへ変換します。

<a name="custom-webpack-configuration"></a>
### Webpackカスタム設定

Laravel Mixはできるだけ素早く実行できるように、裏で事前に設定済みの`webpack.config.js`ファイルを参照しています。時々、このファイルを変更する必要が起きるでしょう。参照する必要がある特別なローダやプラグインがあったり、Sassの代わりにStylusを使うのが好みであるかもしれません。そうした場合、２つの選択肢があります。

#### カスタム設定のマージ

Mixはオーバーライドする短いWebpack設定をマージできるように、便利な`webpackConfig`メソッドを提供しています。これは、`webpack.config.js`ファイルをコピーし、独自バージョンをメンテナンスする必要がないため、やや魅力的な選択しです。`webpackConfig`メソッドは、適用したい[Webpack限定設定](https://webpack.js.org/configuration/)を含むオブジェクトを引数に取ります。

    mix.webpackConfig({
        resolve: {
            modules: [
                path.resolve(__dirname, 'vendor/laravel/spark/resources/assets/js')
            ]
        }
    });

#### カスタム設定ファイル

Webpack設置をすべてカスタマイズしたい場合は、`node_modules/laravel-mix/setup/webpack.config.js`をプロジェクトのルートディレクトリへコピーしてください。次に、`package.json`ファイル中の`--config`参照を全て新しくコピーした設定ファイルに変更します。カスタマイズにこのアプローチを取る場合は、Mixの`webpack.config.js`に対するアップストリームの機能変更を自分でカスタマイズするファイルへマージする必要があります。

<a name="copying-files-and-directories"></a>
## ファイル／ディレクトリコピー

`copy`メソッドは、ファイルやディレクトリを新しい場所へコピーします。これは`node_modules`ディレクトリ中の特定のアセットを`public`フォルダへ再配置する必要がある場合に便利です。

    mix.copy('node_modules/foo/bar.css', 'public/css/bar.css');

ディレクトリのコピー時は、`copy`メソッドはディレクトリ構造をフラットにします。元の構造を保持したい場合は、`copyDirectory`メソッドを代わりに使用してください。

    mix.copyDirectory('assets/img', 'public/img');

<a name="versioning-and-cache-busting"></a>
## バージョン付け／キャッシュ対策

多くの開発者は古いコードが提供され続けないように、新しいアセットがブラウザにロードされるよう、タイムスタンプやユニークなトークンをコンパイル済みのアセットへサフィックスとして付加しています。Mixでは、`version`メソッドを使用し、これを処理できます。

`version`メソッドは自動的に全コンパイルファイルのファイル名へ一意のハッシュを追加し、キャッシュを更新できるようにします。

    mix.js('resources/assets/js/app.js', 'public/js')
       .version();

バージョン付けしたファイルを生成すると、皆さんは実際のファイル名がわからなくなります。そのため、適切なハッシュ付きアセットをロードするために、[ビュー](/docs/{{version}}/views)中でLaravelのグローバル`mix`関数を使う必要があります。`mix`関数はハッシュ付きファイルの最新の名前を自動的に判定します。

    <link rel="stylesheet" href="{{ mix('/css/app.css') }}">

バージョン付けしたファイルは、通常開発に必要ないため、`npm run production`を実行するときのみ、バージョン付けするように指示したいと思うでしょう。

    mix.js('resources/assets/js/app.js', 'public/js');

    if (mix.inProduction()) {
        mix.version();
    }

<a name="browsersync-reloading"></a>
## Browsersyncリロード

[BrowserSync](https://browsersync.io/)は自動的にファイルの変更を監視し、手動で再読込しなくても変更をブラウザに反映してくれます。`mix.browserSync()`メソッドを呼び出し、有効にします。

    mix.browserSync('my-domain.test');

    // もしくは

    // https://browsersync.io/docs/options
    mix.browserSync({
        proxy: 'my-domain.test'
    });

このメソッドには文字列（プロキシ）かオブジェクト（BrowserSync設定）のどちらかを渡します。次に、`npm run watch`コマンドにより、Webpackの開発サーバを起動します。これでスクリプトかPHPファイルを変更すると、すぐにページが再読込され、変更が反映されるのを目にするでしょう。

<a name="environment-variables"></a>
## 環境変数

`.env`ファイルの中でキーに`MIX_`をプリフィックスとしてつけることで、環境変数をMixへ注入できます。

    MIX_SENTRY_DSN_PUBLIC=http://example.com

`.env`ファイルで定義すると、`process.env`オブジェクトを通じてアクセスできるようになります。`watch`タスク実行中に、その値が変化した場合は、タスクを再起動する必要があります。

    process.env.MIX_SENTRY_DSN_PUBLIC

<a name="notifications"></a>
## 通知

可能な場合、Mixは自動的に各バンドルをＯＳ通知で表示します。これにより、コンパイルが成功したのか、失敗したのか、即座にフィードバックが得られます。しかし、こうした通知を無効にしたい場合もあるでしょう。一例は、実働サーバでMixを起動する場合です。`disableNotifications`メソッドにより、通知を無効にできます。

    mix.disableNotifications();
