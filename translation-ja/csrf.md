# CSRF保護

- [イントロダクション](#csrf-introduction)
- [URIの除外](#csrf-excluding-uris)
- [X-CSRF-Token](#csrf-x-csrf-token)
- [X-XSRF-Token](#csrf-x-xsrf-token)

<a name="csrf-introduction"></a>
## イントロダクション

Laravelでは、[クロス・サイト・リクエスト・フォージェリ](https://en.wikipedia.org/wiki/Cross-site_request_forgery)(CSRF)からアプリケーションを簡単に守れます。クロス・サイト・リクエスト・フォージェリは悪意のあるエクスプロイトの一種であり、信頼できるユーザーになり代わり、認められていないコマンドを実行します。

Laravelは、アプリケーションにより管理されているアクティブなユーザーの各セッションごとに、CSRF「トークン」を自動的に生成しています。このトークンを認証済みのユーザーが、実装にアプリケーションに対してリクエストを送信しているのかを確認するために利用します。

アプリケーションでHTMLフォームを定義する場合はいつでも、隠しCSRFトークンフィールドをフォームに埋め込み、CSRF保護ミドルウェアがリクエストの有効性をチェックできるようにしなければなりません。トークン隠しフィールドを生成するには、`csrf_field`ヘルパ関数を使ってください。

    <form method="POST" action="/profile">
        {{ csrf_field() }}
        ...
    </form>

`web`ミドルウェアグループに含まれている、`VerifyCsrfToken` [ミドルウェア](/docs/{{version}}/middleware)が、リクエスト中のトークンとセッションに保存されているトークンが一致するか、確認しています。

#### CSRFトークンとJavaScript

JacaScriptで駆動するアプリケーションを構築する場合、JavaScript HTTPライブラリーに対し、全ての送信リクエストへCSRFトークンを自動的に追加させると便利です。デフォルトでAxios HTTPライブラリにより、`csrf-token`メタタグの値が、`resources/assets/js/bootstrap.js`ファイルに登録されます。このライブラリを使用しない場合、自身のアプリケーションでこの振る舞いを用意する必要があります。

<a name="csrf-excluding-uris"></a>
## URIの除外

一連のURIをCSRF保護より除外したい場合もあります。たとえば、[Stripe](https://stripe.com)を課金処理に採用しており、そのWebフックシステムを利用している時、LaravelのCSRF保護よりWebフック処理ルートを除外する必要があるでしょう。なぜならルートに送るべきCSRFトークンがどんなものか、Stripeは知らないからです。

通常、この種のルートは`RouteServiceProvider`が`routes/web.php`ファイル中の全ルートへ適用する、`web`ミドルウェアから外しておくべきです。しかし、`VerifyCsrfToken`ミドルウェアの`$except`プロパティへ、そうしたURIを追加することによっても、ルートを除外することができます。

    <?php

    namespace App\Http\Middleware;

    use Illuminate\Foundation\Http\Middleware\VerifyCsrfToken as Middleware;

    class VerifyCsrfToken extends Middleware
    {
        /**
         * CSRFバリデーションから除外するURI
         *
         * @var array
         */
        protected $except = [
            'stripe/*',
            'http://example.com/foo/bar',
            'http://example.com/foo/*',
        ];
    }

<a name="csrf-x-csrf-token"></a>
## X-CSRF-TOKEN

更に追加でPOSTパラメーターとしてCSRFトークンを確認したい場合は、Laravelの`VerifyCsrfToken`ミドルウェアが`X-CSRF-TOKEN`リクエストヘッダもチェックします。たとえば、HTML中の`meta`タグにトークンを保存します。

    <meta name="csrf-token" content="{{ csrf_token() }}">

`meta`タグを作成したら、jQueryのようなライブラリーで、全リクエストヘッダにトークンを追加できます。この手法によりAJAXベースのアプリケーションにシンプルで便利なCSRF保護を提供できます。

    $.ajaxSetup({
        headers: {
            'X-CSRF-TOKEN': $('meta[name="csrf-token"]').attr('content')
        }
    });

> {tip} デフォルトでAxios HTTPライブラリにより、`csrf-token`メタタグの値が、`resources/assets/js/bootstrap.js`へ保持されます。このライブラリを使用しない場合、自分のアプリケーションでこの振る舞いを実現する必要があります。

<a name="csrf-x-xsrf-token"></a>
## X-XSRF-TOKEN

LaravelはCSRFトークンをフレームワークにより生成されるリクエストに含まれる、`XSRF-TOKEN`クッキーの中に保存します。このクッキーの値を`X-XSRF-TOKEN`リクエストヘッダにセットすることが可能です。

いくつかのJavaScriptフレームワークや、AngularとAxiosのようなライブラリーでは、自動的に値を`X-XSRF-TOKEN`ヘッダに設定するため、利便性を主な目的としてこのクッキーを送ります。
