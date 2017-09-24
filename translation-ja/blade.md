# Bladeテンプレート

- [イントロダクション](#introduction)
- [テンプレートの継承](#template-inheritance)
    - [レイアウトの定義](#defining-a-layout)
    - [レイアウトの拡張](#extending-a-layout)
- [コンポーネントとスロット](#components-and-slots)
- [データ表示](#displaying-data)
    - [BladeとJavaScriptフレームワーク](#blade-and-javascript-frameworks)
- [制御構文](#control-structures)
    - [If文](#if-statements)
    - [Switch Statements](#switch-statements)
    - [繰り返し](#loops)
    - [ループ変数](#the-loop-variable)
    - [コメント](#comments)
    - [PHP](#php)
- [サブビューの読み込み](#including-sub-views)
    - [コレクションのレンダービュー](#rendering-views-for-collections)
- [スタック](#stacks)
- [サービス注入](#service-injection)
- [Blade拡張](#extending-blade)
    - [Custom If Statements](#custom-if-statements)

<a name="introduction"></a>
## イントロダクション

BladeはシンプルながらパワフルなLaravelのテンプレートエンジンです。他の人気のあるPHPテンプレートエンジンとは異なり、ビューの中にPHPを直接記述することを許しています。全BladeビューはPHPへコンパイルされ、変更があるまでキャッシュされます。つまりアプリケーションのオーバーヘッドは基本的に０です。Bladeビューには`.blade.php`ファイル拡張子を付け、通常は`resources/views`ディレクトリの中に設置します。

<a name="template-inheritance"></a>
## テンプレートの継承

<a name="defining-a-layout"></a>
### レイアウト定義

Bladeを使用する主な利点は、**テンプレートの継承**と**セクション**です。初めに簡単な例を見てください。通常ほとんどのアプリケーションでは、数多くのページを同じ全体的なレイアウトの中に表示するので、最初は"master"ページレイアウトを確認しましょう。レイアウトは一つのBladeビューとして、簡単に定義できます。

    <!-- resources/views/layouts/app.blade.phpとして保存 -->

    <html>
        <head>
            <title>アプリ名 - @yield('title')</title>
        </head>
        <body>
            @section('sidebar')
                ここがメインのサイドバー
            @show

            <div class="container">
                @yield('content')
            </div>
        </body>
    </html>

ご覧の通り、典型的なHTMLマークアップで構成されたファイルです。しかし、`@section`や`@yield`ディレクティブに注目です。`@section`ディレクティブは名前が示す通りにコンテンツのセクションを定義し、一方の`@yield`ディレクティブは指定したセションの内容を表示するために使用します。

これでアプリケーションのレイアウトが定義できました。このレイアウトを継承する、子のページを定義しましょう。

<a name="extending-a-layout"></a>
### レイアウト拡張

子のページを定義するには、「継承」するレイアウトを指定する、Blade `@extends`ディレクティブを使用します。Bladeレイアウトを拡張するビューは、`@section`ディレクティブを使用し、レイアウトのセクションに内容を挿入します。前例にあるようにレイアウトでセクションを表示するには`@yield`を使用します。

    <!-- resources/views/child.blade.phpとして保存 -->

    @extends('layouts.app')

    @section('title', 'Page Title')

    @section('sidebar')
        @@parent

        <p>ここはメインのサイドバーに追加される</p>
    @endsection

    @section('content')
        <p>ここが本文のコンテンツ</p>
    @endsection

この例の`sidebar`セクションでは、レイアウトのサイドバーの内容をコンテンツに上書きするのではなく追加するために`@@parent`ディレクティブを使用しています。`@@parent`ディレクティブはビューをレンダーするときに、レイアウトの内容に置き換わります。

> {tip} 直前の例とは異なり、この`sidebar`セクションは`@show`の代わりに`@endsection`で終わっています。`@endsection`ディレクティブはセクションを定義するだけに対し、`@show`は定義しつつ、そのセクションを**即時にその場所に取り込みます**。

Bladeビューはグローバルな`view`ヘルパを使用し、ルートから返すことができます。

    Route::get('blade', function () {
        return view('child');
    });

<a name="components-and-slots"></a>
## コンポーネントとスロット

コンポーネントとスロットは、セクションとレイアウトと似た利便を提供します。しかし、コンポーネントとスロットのメンタルモデルのほうが簡単に理解できることに気づくはずです。最初に、アプリケーション全体で再利用する、"alert"コンポーネントをイメージしてください。

    <!-- /resources/views/alert.blade.php -->

    <div class="alert alert-danger">
        {{ $slot }}
    </div>

`{{ $slot }}`変数は、このコンポーネントへ注入しようとする内容を含んでいます。では、このコンポーネントを構築するため、`@component` Bladeディレクティブを使いましょう。

    @component('alert')
        <strong>Whoops!</strong> Something went wrong!
    @endcomponent

一つのコンポーネントに対し、複数のスロットを定義するのも、役立つことがあるでしょう。"title"を注入できるようにalertコンポーネントを改造してみましょう。名前付きスロットは、名前に一致する変数をただ"echo"します。

    <!-- /resources/views/alert.blade.php -->

    <div class="alert alert-danger">
        <div class="alert-title">{{ $title }}</div>

        {{ $slot }}
    </div>

これで、`@slot`ディレクティブを使い、名前付きスロットへ内容を注入できます。`@slot`ディレクティブ範囲外の全内容は、`$slot`変数の中のコンポーネントへ渡されます。

    @component('alert')
        @slot('title')
            Forbidden
        @endslot

        You are not allowed to access this resource!
    @endcomponent

#### コンポーネントへ追加のデータを渡す

場合により、コンポーネントへ追加のデータを渡す必要が起きます。そのため、`@component`ディレクティブの第２引数で、データの配列が渡せます。全データはテンプレート中で変数として利用できます。

    @component('alert', ['foo' => 'bar'])
        ...
    @endcomponent

<a name="displaying-data"></a>
## データ表示

Bladeビューに渡されたデータは、波括弧で変数を囲うことで表示できます。たとえば、次のルートがあるとしましょう。

    Route::get('greeting', function () {
        return view('welcome', ['name' => 'Samantha']);
    });

`name`変数の内容を表示しましょう。

    Hello, {{ $name }}.

もちろんビューに渡された変数の内容を表示するだけに限られません。PHP関数の結果をechoすることもできます。実際、どんなPHPコードもBladeのecho文の中に書けます。

    The current UNIX timestamp is {{ time() }}.

> {tip} Bladeの`{{ }}`記法はXSS攻撃を防ぐため、自動的にPHPの`htmlspecialchars`関数を通されます。

#### エスケープしないデータの表示

デフォルトでブレードの`{{ }}`文はXSS攻撃を防ぐために、PHPの`htmlspecialchars`関数を自動的に通されます。しかしデータをエスケープしたくない場合は、以下の構文を使ってください。

    Hello, {!! $name !!}.

> {note} アプリケーションでユーザーの入力内容をechoする場合は注意が必要です。ユーザーの入力を表示するときは、常に二重の波括弧の記法でHTMLエンティティにエスケープすべきです。

#### JSONのレンダ

JavaScriptの変数を初期化するために、配列をビューに渡してJSONとして描画することができます。

    <script>
        var app = <?php echo json_encode($array); ?>;
    </script>

その際、`json_encode`を使う代わりに、`@json`ディレクティブを使うことができます。

    <script>
        var app = @json($array)
    </script>

<a name="blade-and-javascript-frameworks"></a>
### BladeとJavaScriptフレームワーク

多くのJavaScriptフレームワークでも、与えられた式をブラウザに表示するために波括弧を使っていますので、`@`シンボルでBladeレンダリングエンジンに波括弧の展開をしないように指示することが可能です。

    <h1>Laravel</h1>

    Hello, @{{ name }}.

この例で、`@`記号はBladeにより削除されます。しかし、`{{ name }}`式はBladeエンジンにより変更されずにそのまま残り、JavaScriptフレームワークによりレンダリングできるようになります。

#### `@verbatim`ディレクティブ

テンプレートの広い箇所でJavaScript変数を表示する場合は、HTMLを`@verbatim`ディレクティブで囲めば、各Blade echo文の先頭に`@`記号を付ける必要はなくなります。

    @verbatim
        <div class="container">
            Hello, {{ name }}.
        </div>
    @endverbatim

<a name="control-structures"></a>
## 制御構文

テンプレートの継承とデータ表示に付け加え、Bladeは条件文やループなどの一般的なPHPの制御構文の便利な短縮記述方法を提供しています。こうした短縮形は、PHP制御構文の美しく簡潔な記述を提供しつつも、対応するPHPの構文と似せています。

<a name="if-statements"></a>
### If文

`if`文の構文には、`@if`、`@elseif`、`@else`、`@endif`ディレクティブを使用します。これらの使い方はPHPの構文と同じです。

    @if (count($records) === 1)
        １レコードある！
    @elseif (count($records) > 1)
        複数レコードある！
    @else
        レコードがない！
    @endif

便利な@unlessディレクティブも提供しています。

    @unless (Auth::check())
        あなたはログインしていません。
    @endunless

さらに、既に説明したように`@isset`と`@empty`ディレクティブも、同名のPHP関数の便利なショートカットとして使用できます。

    @isset($records)
        // $recordsは定義済みでnullでない
    @endisset

    @empty($records)
        // $recordsが「空」だ
    @endempty

#### 認証ショートカット

`@auth`と`@guest`ディレクティブは、現在のユーザーが認証されているか、もしくはゲストであるかを簡単に判定するために使用します。

    @auth
        // ユーザーは認証済み
    @endauth

    @guest
        // ユーザーは認証されていない
    @endguest

必要であれば、`@auth`と`@guest`ディレクティブ使用時に、確認すべき[認証ガード](/docs/{{version}}/authentication)を指定できます。

    @auth('admin')
        // ユーザーは認証済み
    @endauth

    @guest('admin')
        // ユーザーは認証されていない
    @endguest

<a name="switch-statements"></a>
### Switch文

`@switch`、`@case`、`@break`、`@default`、`@endswitch`ディレクティブを使用し、switch文を構成できます。

    @switch($i)
        @case(1)
            最初のケース
            @break

        @case(2)
            ２番めのケース
            @break

        @default
            デフォルトのケース
    @endswitch

<a name="loops"></a>
### 繰り返し

条件文に加え、BladeはPHPがサポートしている繰り返し構文も提供しています。これらも、対応するPHP構文と使い方は一緒です。

    @for ($i = 0; $i < 10; $i++)
        現在の値は： {{ $i }}
    @endfor

    @foreach ($users as $user)
        <p>これは {{ $user->id }} ユーザーです。</p>
    @endforeach

    @forelse ($users as $user)
        <li>{{ $user->name }}</li>
    @empty
        <p>ユーザーなし</p>
    @endforelse

    @while (true)
        <p>無限ループ中</p>
    @endwhile

> {tip} 繰り返し中は[ループ変数](#the-loop-variable)を使い、繰り返しの最初や最後なのかなど、繰り返し情報を取得できます。

繰り返しを使用する場合、ループを終了するか、現在の繰り返しをスキップすることもできます。

    @foreach ($users as $user)
        @if ($user->type == 1)
            @continue
        @endif

        <li>{{ $user->name }}</li>

        @if ($user->number == 5)
            @break
        @endif
    @endforeach

もしくは、ディレクティブに条件を記述して、一行で済ますこともできます。

    @foreach ($users as $user)
        @continue($user->type == 1)

        <li>{{ $user->name }}</li>

        @break($user->number == 5)
    @endforeach

<a name="the-loop-variable"></a>
### ループ変数

繰り返し中は、`$loop`変数が使用できます。この変数により、現在のループインデックスや繰り返しの最初／最後なのかなど、便利な情報にアクセスできます。

    @foreach ($users as $user)
        @if ($loop->first)
            これは最初の繰り返し
        @endif

        @if ($loop->last)
            これは最後の繰り返し
        @endif

        <p>これは {{ $user->id }} ユーザーです。</p>
    @endforeach

ネストした繰り返しでは、親のループの`$loop`変数に`parent`プロパティを通じアクセスできます。

    @foreach ($users as $user)
        @foreach ($user->posts as $post)
            @if ($loop->parent->first)
                これは親のループの最初の繰り返しだ
            @endif
        @endforeach
    @endforeach

`$loop`変数は、他にもいろいろと便利なプロパティを持っています。

プロパティ  | 説明
------------- | -------------
`$loop->index`  |  現在のループのインデックス（初期値0）
`$loop->iteration`  |  現在の繰り返し数（初期値1）
`$loop->remaining`  |  繰り返しの残り数
`$loop->count`  |  繰り返し中の配列の総アイテム数
`$loop->first`  |  ループの最初の繰り返しか
`$loop->last`  |  ループの最後の繰り返しか
`$loop->depth`  |  現在のループのネストレベル
`$loop->parent`  |  ループがネストしている場合、親のループ変数

<a name="comments"></a>
### コメント

Bladeでビューにコメントを書くこともできます。HTMLコメントと異なり、Bladeのコメントはアプリケーションから返されるHTMLには含まれません。

    {{-- このコメントはレンダー後のHTMLには現れない --}}

<a name="php"></a>
### PHP

PHPコードをビューへ埋め込むと便利な場合もあります。Bladeの`@php`ディレクティブを使えば、テンプレートの中でプレーンなPHPブロックを実行できます。

    @php
        //
    @endphp

> {tip} Bladeはこの機能を提供していますが、数多く使用しているのであれば、それはテンプレートへ多すぎるロジックを埋め込んでいるサインです。

<a name="including-sub-views"></a>
## サブビューの読み込み

Bladeの`@include`ディレクディブを使えば、ビューの中から簡単に他のBladeビューを取り込めます。読み込み元のビューで使用可能な変数は、取り込み先のビューでも利用可能です。

    <div>
        @include('shared.errors')

        <form>
            <!-- フォームの内容 -->
        </form>
    </div>

親のビューの全データ変数が取り込み先のビューに継承されますが、追加のデータも配列で渡すことができます。

    @include('view.name', ['some' => 'data'])

もちろん、存在しないビューを`@include`すれば、Laravelはエラーを投げます。存在しているかどうかわからないビューを取り込みたい場合は、`@includeIf`ディレクティブを使います。

    @includeIf('view.name', ['some' => 'data'])

指定した論理条件にもとづいて`@include`したい場合は、`@includeWhen`ディレクティブを使用します。

    @includeWhen($boolean, 'view.name', ['some' => 'data'])

> {note} Bladeビューの中では`__DIR__`や`__FILE__`を使わないでください。キャッシュされたコンパイル済みのビューのパスが返されるからです。

<a name="rendering-views-for-collections"></a>
### コレクションのレンダービュー

Bladeの`@each`ディレクティブを使い、ループとビューの読み込みを組み合わせられます。

    @each('view.name', $jobs, 'job')

最初の引数は配列かコレクションの各要素をレンダーするための部分ビューです。第２引数は繰り返し処理する配列かコレクションで、第３引数はビューの中の繰り返し値が代入される変数名です。ですから、たとえば`jobs`配列を繰り返す場合なら、部分ビューの中で各ジョブには`job`変数としてアクセスしたいと通常は考えるでしょう。 現在の繰り返しのキーは、部分ビューの中の`key`変数で参照できます。

`@each`ディレクティブには第４引数を渡たせます。この引数は配列が空の場合にレンダーされるビューを指定します。

    @each('view.name', $jobs, 'job', 'view.empty')

> {note} `@each`を使ってレンダされるビューは、親のビューから変数を継承しません。子ビューで親ビューの変数が必要な場合は、代わりに`@foreach`と`@include`を使用してください。

<a name="stacks"></a>
## スタック

Bladeはさらに、他のビューやレイアウトでレンダーできるように、名前付きのスタックへ内容を退避できます。子ビューで必要なJavaScriptを指定する場合に、便利です。

    @push('scripts')
        <script src="/example.js"></script>
    @endpush

必要なだけ何回もスタックをプッシュできます。スタックした内容をレンダーするには、`@stack`ディレクティブにスタック名を指定してください。

    <head>
        <!-- Headの内容 -->

        @stack('scripts')
    </head>

<a name="service-injection"></a>
## サービス注入

`@inject`ディレクティブはLaravelの[サービスコンテナ](/docs/{{version}}/container)からサービスを取得するために使用します。`@inject`の最初の引数はそのサービスを取り込む変数名で、第２引数は依存解決したいクラス／インターフェイス名です。

    @inject('metrics', 'App\Services\MetricsService')

    <div>
        Monthly Revenue: {{ $metrics->monthlyRevenue() }}.
    </div>

<a name="extending-blade"></a>
## Blade拡張

Bladeでは`directive`メソッドを使い、自分のカスタムディレクティブを定義することができます。Bladeコンパイラがそのカスタムディレクティブを見つけると、そのディレクティブに渡される引数をコールバックへの引数として呼び出します。

次の例は`@datetime($var)`ディレクティブを作成し、渡される`DateTime`インスタンスの`$var`をフォーマットします。

    <?php

    namespace App\Providers;

    use Illuminate\Support\Facades\Blade;
    use Illuminate\Support\ServiceProvider;

    class AppServiceProvider extends ServiceProvider
    {
        /**
         * サービスの初期起動後に登録する
         *
         * @return void
         */
        public function boot()
        {
            Blade::directive('datetime', function ($expression) {
                return "<?php echo ($expression)->format('m/d/Y H:i'); ?>";
            });
        }

        /**
         * コンテナへ結合を登録する
         *
         * @return void
         */
        public function register()
        {
            //
        }
    }

ご覧の通り、ディレクティブがどんなものであれ、渡された引数に`format`メソッドをチェーンし、呼び出しています。そのため、この例のディレクティブの場合、最終的に生成されるPHPコードは、次のようになります。

    <?php echo ($var)->format('m/d/Y H:i'); ?>

> {note} Bladeディレクティブのロジックを更新した後に、Bladeビューのキャッシュを全部削除する必要があります。`view:clear` Artisanコマンドで、キャッシュされているBladeビューを削除できます。

<a name="custom-if-statements"></a>
### カスタムif文

シンプルなカスタム条件文を定義する時、必要以上にカスタムディレクティブのプログラミングが複雑になってしまうことが、時々起きます。そのため、Bladeはクロージャを使用し、カスタム条件ディレクティブを素早く定義できるように、`Blade::if`メソッドを提供しています。例として、現在のアプリケーション環境をチェックするカスタム条件を定義してみましょう。`AppServiceProvider`の`boot`メソッドで行います。

    use Illuminate\Support\Facades\Blade;

    /**
     * サービスの初期処理後に実行
     *
     * @return void
     */
    public function boot()
    {
        Blade::if('env', function ($environment) {
            return app()->environment($environment);
        });
    }

カスタム条件を定義したら、テンプレートの中で簡単に利用できます。

    @env('local')
        // アプリケーションはlocal環境
    @else
        // アプリケーションはlocal環境ではない
    @endenv
