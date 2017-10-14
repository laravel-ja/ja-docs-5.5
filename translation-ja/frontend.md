# JavaScriptとCSSスカフォールド

- [イントロダクション](#introduction)
- [CSSの出力](#writing-css)
- [JavaScriptの出力](#writing-javascript)
    - [Vueコンポーネントの出力](#writing-vue-components)
    - [Using React](#using-react)

<a name="introduction"></a>
## イントロダクション

LaravelはJavaScriptやCSSプリプロセッサの使用を規定してはいませんが、開発時点の元としてほとんどのアプリケーションで役立つだろう[Bootstrap](https://getbootstrap.com)と[Vue](https://vuejs.org)を提供しています。これらのフロントエンドパッケージをインストールするため、Laravelは[NPM](https://www.npmjs.org)を使用しています。

#### CSS

CSSをもっと楽しく取り扱うために役立つ、変数やmixinなどのパワフルな機能を通常のCSSへ付け加え、SASSとLESSをコンパイルするため、[Laravel Mix](/docs/{{version}}/mix)はクリーンで表現的なAPIを提供しています。このドキュメントでは、CSSコンパイル全般について簡単に説明します。SASSとLESSのコンパイルに関する情報は、[Laravel Mix documentation](/docs/{{version}}/mix)で確認してください。

#### JavaScript

アプリケーションを構築するために、特定のJavaScriptフレームワークやライブラリの使用をLaravelは求めていません。しかし、[Vue](https://vuejs.org)ライブラリを使用した近代的なJavaScriptを書き始めやすくできるように、基本的なスカフォールドを用意しています。Vueはコンポーネントを使った堅牢なJavaScriptアプリケーションを構築するために、記述的なAPIを提供しています。CSSに関しては、Laravel Mixを使用し、JavaScriptコンポーネントをブラウザでそのまま使用できる１ファイルへ、簡単に圧縮できます。

#### フロントエンドスカフォールドの削除

アプリケーションから、フロントエンド向けにスカフォールドした結果を削除したい場合は、`preset` Artisanコマンドを使用します。このコマンドを`none`オプションと一緒に使うと、アプリケーションからBootstrapとVue向けのスカフォールド結果を削除し、空のSASSファイルといくつかのJavaScriptユーティリティライブラリだけが残ります。

    php artisan preset none

<a name="writing-css"></a>
## CSSの出力

Laravelの`package.json`ファイルは、アプリケーションのフロントエンドのプロトタイピングを開始するのに便利なように、`bootstrap-sass`パッケージを含んでいます。しかし、アプリケーションでの必要性に応じ、自由にパッケージを`package.json`ファイルに追加／削除してください。Laravelアプリケーションを構築するため、Bootstrapフレームワークを使用する必要はありません。Bootstrapを選んでいる開発者にとって、開発しやすいように用意してあるだけです。

CSSのコンパイルを始める前に、プロジェクトのフロントエンド開発に必要な依存パッケージである、[Nodeプロジェクトマネージャー(NPM)](https://www.npmjs.org)を使用し、インストールしてください。

    npm install

`npm install`を使い、依存パッケージをインストールし終えたら、[Laravel Mix](/docs/{{version}}/mix#working-with-stylesheets)を使用して、SASSファイルを通常のCSSへコンパイルできます。`npm run dev`コマンドは`webpack.mix.js`ファイル中の指示を処理します。通常、コンパイル済みCSSは`public/css`ディレクトリへ設置されます。

    npm run dev

Laravelにデフォルトで含まれる`webpack.mix.js`は、`resources/assets/sass/app.scss` SASSファイルをコンパイルします。この`app.scss`ファイルはSASS変数のファイルと、ほとんどのアプリケーションの構築開始時に役に立つBootstrapをロードします。望みどおりに`app.scss`を自由にカスタマイズし、さらに[Laravel Mixを設定](/docs/{{version}}/mix)することで、まったく別のプリプロセッサを使用することさえ可能です。

<a name="writing-javascript"></a>
## JavaScriptの出力

アプリケーションで要求されている、JavaScriptの全依存パッケージは、プロジェクトルートディレクトリにある`package.json`ファイルで見つかります。このファイルは`composer.json`ファイルと似ていますが、PHPの依存パッケージの代わりにJavaScriptの依存が指定されている点が異なります。依存パッケージは、[Node package manager (NPM)](https://www.npmjs.org)を利用し、インストールできます。

    npm install

> {tip} JavaScriptアプリケーションを構築開始するために役立つよう、`vue`や`axios`のようなパッケージがデフォルトでLaravelの`package.json`ファイルに含まれています。自身のアプリケーションの要求に合わせ、`package.json`ファイルへ自由に追加、削除してください。

`webpack.mix.js` file:パッケージをインストールしたら、`npm run dev`コマンドで[アセットをコンパイル](/docs/{{version}}/mix)できます。Webpackは、モダンなJavaScriptアプリケーションのための、モジュールビルダです。`npm run dev`コマンドを実行すると、Webpackは`webpack.mix.js`ファイル中の指示を実行します。

    npm run dev

デフォルトでLaravelの`webpack.mix.js`ファイルは、SASSと`resources/assets/js/app.js`ファイルをコンパイルするように指示しています。`app.js`ファイルの中で、Vueコンポーネントを登録してください。もしくは、他のフーレムワークが好みであれば、自分のJavaScriptアプリケーションの設定を行えます。コンパイル済みのJavaScriptは通常、`public/js`ディレクトリへ出力されます。

> {tip} `app.js`ファイルは、Vue、Axios、jQuery、その他のJavaScript依存パッケージを起動し、設定する`resources/assets/js/bootstrap.js`ファイルをロードします。JacaScript依存パッケージを追加した場合、このファイルの中で設定してください。

<a name="writing-vue-components"></a>
### Vueコンポーネントの出力

インストールしたてのLaravelアプリケーションは、デフォルトとして、`resources/assets/js/components`ディレクトリの中に`ExampleComponent.vue` Vueコンポーネントを持っています。`ExampleComponent.vue`ファイルは、JacaScriptとHTMLテンプレートを同じファイル中で定義している、[シングルファイルVueコンポーネント](https://vuejs.org/guide/single-file-components)の一例です。シングルファイルコンポーネントは、JavaScript駆動アプリケーションを構築するために、とても便利なアプローチを提供しています。`app.js`ファイルで登録されています。このコンポーネントサンプルは、`app.js`ファイル中で登録されています。

    Vue.component(
        'example-component',
        require('./components/ExampleComponent.vue')
    );

アプリケーションでコンポ―エントを使用するには、HTMLの定形コードを挿入するだけです。たとえば、アプリケーションの認証のスカフォールドを行うために、`make:auth` Artisanコマンドを実行し、スクリーンを登録したら、`home.blade.php` Bladeテンプレートにコンポーネントを挿入できます。

    @extends('layouts.app')

    @section('content')
        <example-component></example-component>
    @endsection

> {tip} Vueコンポーネントを変更したら、毎回`npm run dev`コマンドを実行しなくてはならないことを覚えておきましょう。もしくは、`npm run watch`コマンドを実行して監視すれば、コンポーネントが更新されるたび、自動的に再コンパイルされます。

もちろん、Vueコンポーネントの記述を学ぶことに興味があれば、Vueフレームワーク全体についての概念を簡単に読み取れる、[Vueドキュメント](https://vuejs.org/guide/)を一読してください。

<a name="using-react"></a>
### Reactの使用

JavaScriptアプリケーションの構築に、Reactを使いたければ、Laravelは簡単にVueのスカフォールドをReactのスカフォールドへ簡単に変更します。真新しいLaravelアプリケーションで、`react`オプションを付け、`preset`コマンドを使用してください。

    php artisan preset react

このコマンド一つで、Vueのスカフォールドを削除し、サンプルコンポーネントを含んだReactのスカフォールドを設置します。
