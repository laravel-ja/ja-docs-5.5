# URL生成

- [イントロダクション](#introduction)
- [基礎](#the-basics)
    - [URL生成の基礎](#generating-basic-urls)
    - [現在のURLへのアクセス](#accessing-the-current-url)
- [名前付きルートのURL](#urls-for-named-routes)
- [コントローラアクションのURL](#urls-for-controller-actions)
- [デフォルト値](#default-values)

<a name="introduction"></a>
## イントロダクション

Laravelはアプリケーションに対するURL生成の手助けとなる、数多くのヘルパを提供しています。これは主にテンプレートやAPIレスポンスでリンクを構築するためにはもちろんのこと、リダイレクトレスポンスの生成や、アプリケーションの他の部分でも役立ちます。

<a name="the-basics"></a>
## 基礎

<a name="generating-basic-urls"></a>
### URL生成の基礎

`url`ヘルパは、アプリケーションに対する任意のURLを生成するために使用されます。生成されるURLには自動的に、現在のリクエストのスキーム（HTTP／HTTPS）とホストが使用されます。

    $post = App\Post::find(1);

    echo url("/posts/{$post->id}");

    // http://example.com/posts/1

<a name="accessing-the-current-url"></a>
### 現在のURLへのアクセス

`url`ヘルパにパスを指定しないと、`Illuminate\Routing\UrlGenerator`インスタンスが返され、現在のURLに関する情報へアクセスできます。

    // クエリ文字列を除いた現在のURL
    echo url()->current();

    // クエリ文字列を含んだ現在のURL
    echo url()->full();

    // 直前のリクエストの完全なURL
    echo url()->previous();

こうしたメソッドには、`URL`[ファサード](/docs/{{version}}/facades)を使用してもアクセスできます。

    use Illuminate\Support\Facades\URL;

    echo URL::current();

<a name="urls-for-named-routes"></a>
## 名前付きルートのURL

`route`ヘルパは、名前付きルートへのURLを生成するために使用します。名前付きルートにより、定義したルートの実際のURLを指定せずとも、URLを生成することができます。ですから、ルートのURLを変更しても、`route`関数の呼び出しを変更する必要はありません。例として以下のように、アプリケーションが次のルートを持っていると想像してください。

    Route::get('/post/{post}', function () {
        //
    })->name('post.show');

このルートへのURLを生成するには、次のように`route`ヘルパを使用します。

    echo route('post.show', ['post' => 1]);

    // http://example.com/post/1

[Eloquentモデル](/docs/{{version}}/eloquent)の主キーを使用するURLを生成することもよくあると思います。そのため、Eloquentモデルをパラメータ値として渡すことができます。`route`ヘルパは、そのモデルの主キーを自動的に取り出します。

    echo route('post.show', ['post' => $post]);

<a name="urls-for-controller-actions"></a>
## コントローラアクションのURL

`action`関数は、指定するコントローラアクションに対するURLを生成します。コントローラの完全な名前空間を渡す必要はありません。代わりに、`App\Http\Controllers`名前空間からの相対的なコントローラクラス名を指定してください。

    $url = action('HomeController@index');

コントローラメソッドが、ルートパラメータを受け取る場合、この関数の第２引数として渡すことができます。

    $url = action('UserController@profile', ['id' => 1]);

<a name="default-values"></a>
## デフォルト値

アプリケーションにより、特定のURLパラメータのデフォルト値をリクエスト全体で指定したい場合もあります。たとえば、多くのルートで`{locale}`パラメータを定義していると想像してください。

    Route::get('/{locale}/posts', function () {
        //
    })->name('post.index');

毎回`route`ヘルパを呼び出すごとに、`locale`をいつも指定するのは厄介です。そのため、現在のリクエストの間、常に適用されるこのパラメートのデフォルト値は、`URL::defaults`メソッドを使用し定義できます。現在のリクエストでアクセスできるように、[ルートミドルウェア](/docs/{{version}}/middleware#assigning-middleware-to-routes)から、このメソッドを呼び出したいかと思います。

    <?php

    namespace App\Http\Middleware;

    use Closure;
    use Illuminate\Support\Facades\URL;

    class SetDefaultLocaleForUrls
    {
        public function handle($request, Closure $next)
        {
            URL::defaults(['locale' => $request->user()->locale]);

            return $next($request);
        }
    }

一度`locale`パラメータに対するデフォルト値をセットしたら、`route`ヘルパを使いURLを生成する時に、値を渡す必要はもうありません。
