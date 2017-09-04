# エラーとログ

- [イントロダクション](#introduction)
- [設定](#configuration)
    - [エラー詳細](#error-detail)
    - [ログストレージ](#log-storage)
    - [ログレベル](#log-severity-levels)
    - [カスタムMonolog設定](#custom-monolog-configuration)
- [例外ハンドラ](#the-exception-handler)
    - [Reportメソッド](#report-method)
    - [Renderメソッド](#render-method)
    - [Reportable／Renderable例外](#renderable-exceptions)
- [HTTP例外](#http-exceptions)
    - [カスタムHTTPエラーページ](#custom-http-error-pages)
- [ログ](#logging)

<a name="introduction"></a>
## イントロダクション

新しいLaravelプロジェクトを開始する時点で、エラーと例外の処理は既に設定済みです。`App\Exceptions\Handler`クラスはアプリケーションで発生する全例外をログし、ユーザーへ表示するためのクラスです。このドキュメントでは、このクラスの詳細を確認していきます。

パワフルで様々なログハンドラをサポートしている[Monolog](https://github.com/Seldaek/monolog)をLaravelはログに利用しています。単一ログファイルへのログ、一定期間ごとのログファイル切り替え、システムログへのエラー情報の書き込みを選べるように、Laravelはハンドラをあらかじめ準備しています。

<a name="configuration"></a>
## 設定

<a name="error-detail"></a>
### エラー詳細

アプリケーションエラー発生時にユーザーに対し表示する詳細の表示量は、`config/app.php`設定ファイルの`debug`設定オプションで決定します。デフォルト状態でこの設定オプションは、`.env`ファイルで指定される`APP_DEBUG`環境変数の値を反映します。

local環境では`APP_DEBUG`環境変数を`true`に設定すべきでしょう。実働環境ではこの値をいつも`false`にすべきです。実働環境でこの値を`true`にしてしまうと、アプリケーションのエンドユーザーへ、セキュリティリスクになりえる設定情報を表示するリスクを犯すことになります。

<a name="log-storage"></a>
### ログストレージ

Laravelはデフォルトで、単一ファイル(`single`)、日別ファイル(`daily`)、 システムログ(`syslog`)、エラーログ(`errorlog`)ログモードへのログ情報書き込みをサポートしています。Laravelが使用するストレージ機構の設定は、`config/app.php`設定ファイルの`log`オプションを変更します。たとえば単一ファイルではなく、日別のログファイルを使用したい場合、`app`設定ファイルの`log`の値に、`daily`を設定してください。

    'log' => 'daily'

#### 日別ログファイルの最大数

`daily`ログモード使用時にLaravelはデフォルトで、５日分のログファイルだけを残します。保存しておくファイルの数を調整したいときは、`app`設定ファイルの`log_max_files`設定値を変更してください。

    'log_max_files' => 30

<a name="log-severity-levels"></a>
### ログレベル

Monologでは、異なった重大さのレベルのログを扱えます。Laravelはデフォルトで全ログレベルをストレージに書き込みます。しかし、実働環境では`app.php`設定ファイルの`log_level`オプションへ、ログすべき最低の重大さレベルを指定したいことでしょう。

このオプションを一度設定すると、Laravelは指定された重大さ以上の全レベルをログします。たとえば、`log_level`に`error`を設定すると、**error**、**critical**、**alert**、**emergency**メッセージがログされます。

    'log_level' => env('APP_LOG_LEVEL', 'error'),

> {tip} Monologは次のレベルを取り扱います：`debug`、`info`、`notice`、`warning`、`error`、`critical`、`alert`、`emergency`

<a name="custom-monolog-configuration"></a>
### カスタムMonolog設定

Monologがアプリケーションのために設定している内容を完全にコントロールしたければ、アプリケーションの`configureMonologUsing`メソッドを使ってください。このメソッドは`bootstrap/app.php`ファイルの中の`$app`変数が返される直前に呼びださなくてはなりません。

    $app->configureMonologUsing(function ($monolog) {
        $monolog->pushHandler(...);
    });

    return $app;

#### チャンネル名のカスタマイズ

Monologはデフォルトで、`production`や`local`のような現在の環境に合わせた名前でインスタンス化されます。この値を変えるには、`app.php`設定ファイルに、`log_channel`オプションを追加してください。

    'log_channel' => env('APP_LOG_CHANNEL', 'my-app-name'),

<a name="the-exception-handler"></a>
## 例外ハンドラ

<a name="report-method"></a>
### reportメソッド

全例外は`App\Exceptions\Handler`クラスで処理されます。このクラスは`report`と`render`二つのメソッドで構成されています。両メソッドの詳細を見ていきましょう。`report`メソッドは例外をログするか、[BugSnag](https://bugsnag.com)や[Sentry](https://github.com/getsentry/sentry-laravel)のような外部サービスに送信するために使います。デフォルト状態の`report`メソッドは、ただ渡された例外をベースクラスに渡し、そこで例外はログされます。しかし好きなように例外をログすることが可能です。

たとえば異なった例外を別々の方法レポートする必要がある場合、PHPの`instanceof`比較演算子を使ってください。

    /**
     * 例外をレポート、もしくはログする
     *
     * ここは例外をSentryやBugsnagなどへ送るために適した場所
     *
     * @param  \Exception  $exception
     * @return void
     */
    public function report(Exception $exception)
    {
        if ($exception instanceof CustomException) {
            //
        }

        return parent::report($exception);
    }

#### `report`ヘルパ

場合により、例外をレポートする必要があるが、現在のリクエストの処理を継続したい場合があります。`report`ヘルパ関数により、エラーページをレンダすること無く、例外ハンドラの`report`メソッドを使用し、例外を簡単にレポートできます。

    public function isValid($value)
    {
        try {
            // 値の確認…
        } catch (Exception $e) {
            report($e);

            return false;
        }
    }

#### タイプによる例外の無視

例外ハンドラの`$dontReport`プロパティーは、ログしない例外のタイプの配列で構成します。たとえば、404エラー例外と同様に、他のタイプの例外もログしたくない場合です。必要に応じてこの配列へ、他の例外を付け加えてください。

    /**
     * レポートしない例外のリスト
     *
     * @var array
     */
    protected $dontReport = [
        \Illuminate\Auth\AuthenticationException::class,
        \Illuminate\Auth\Access\AuthorizationException::class,
        \Symfony\Component\HttpKernel\Exception\HttpException::class,
        \Illuminate\Database\Eloquent\ModelNotFoundException::class,
        \Illuminate\Validation\ValidationException::class,
    ];

<a name="render-method"></a>
### renderメソッド

`render`メソッドは与えられた例外をブラウザーに送り返すHTTPレスポンスへ変換することに責任を持っています。デフォルトで例外はベースクラスに渡され、そこでレスポンスが生成されます。しかし例外のタイプをチェックし、お好きなようにレスポンスを返してかまいません。

    /**
     * HTTPレスポンスへ例外をレンダー
     *
     * @param  \Illuminate\Http\Request  $request
     * @param  \Exception  $exception
     * @return \Illuminate\Http\Response
     */
    public function render($request, Exception $exception)
    {
        if ($exception instanceof CustomException) {
            return response()->view('errors.custom', [], 500);
        }

        return parent::render($request, $exception);
    }

<a name="renderable-exceptions"></a>
### Reportable／Renderable例外

例外ハンドラの中の`report`と`render`メソッドの中で、例外のタイプをチェックする代わりに、自身のカスタム例外で`report`と`render`メソッドを定義できます。これらのメソッドが存在すると、フレームワークにより自動的に呼び出されます。

    <?php

    namespace App\Exceptions;

    use Exception;

    class RenderException extends Exception
    {
        /**
         * 例外のレポート
         *
         * @return void
         */
        public function report()
        {
            //
        }

        /**
         * 例外をＨＴＴＰレスポンスへレンダ
         *
         * @param  \Illuminate\Http\Request
         * @return \Illuminate\Http\Response
         */
        public function render($request)
        {
            return response(...);
        }
    }

<a name="http-exceptions"></a>
## HTTP例外

例外の中にはサーバでのHTTPエラーコードを表しているものがあります。たとえば「ページが見つかりません」エラー(404)や「未認証エラー」(401)、開発者が生成した500エラーなどです。アプリケーションのどこからでもこの種のレスポンスを生成するには、abortヘルパを使用します。

    abort(404);

`abort`ヘルパは即座に例外を発生させ、その例外は例外ハンドラによりレンダーされることになります。オプションとしてレスポンスのテキストを指定することもできます。

    abort(403, 'Unauthorized action.');

<a name="custom-http-error-pages"></a>
### カスタムHTTPエラーページ

様々なHTTPステータスコードごとに、Laravelはカスタムエラーページを簡単に返せます。たとえば404 HTTPステータスコードに対してカスタムエラーページを返したければ、`resources/views/errors/404.blade.php`を作成してください。このファイルはアプリケーションで起こされる全404エラーに対し動作します。ビューはこのディレクトリに置かれ、対応するHTTPコードと一致した名前にしなくてはなりません。`abort`ヘルパが生成する`HttpException`インスタンスは、`$exception`変数として渡されます。

    <h2>{{ $exception->getMessage() }}</h2>

<a name="logging"></a>
## ログ

Laravelのログ機能は、強力な[Monolog](https://github.com/seldaek/monolog)ライブラリーのシンプルな上位レイヤーを提供します。Laravelはデフォルトで、アプリケーションの日別ログファイルを`storage/logs`ディレクトリへ作成するように設定されています。ログへ情報を書き込むには`Log`[ファサード](/docs/{{version}}/facades)を使います。

    <?php

    namespace App\Http\Controllers;

    use App\User;
    use Illuminate\Support\Facades\Log;
    use App\Http\Controllers\Controller;

    class UserController extends Controller
    {
        /**
         * 指定ユーザーのプロフィールを表示
         *
         * @param  int  $id
         * @return Response
         */
        public function showProfile($id)
        {
            Log::info('ユーザーのプロフィール表示。 ID:'.$id);

            return view('user.profile', ['user' => User::findOrFail($id)]);
        }
    }

ログは[RFC 5424](http://tools.ietf.org/html/rfc5424)で定義されている**emergency**、**alert**、**critical**、**error**、**warning**、**notice**、**info** 、**debug**の８レベルをサポートしています。

    Log::emergency($message);
    Log::alert($message);
    Log::critical($message);
    Log::error($message);
    Log::warning($message);
    Log::notice($message);
    Log::info($message);
    Log::debug($message);

#### コンテキストデータ

ログメソッドにはコンテキストデータを配列で渡すこともできます。このコンテキストデータはログメッセージと一緒に整形され、表示されます。

    Log::info('User failed to login.', ['id' => $user->id]);

#### 裏で動作するMonologインスタンスへのアクセス

Monologはログに使用できる様々な追加のハンドラを提供しています。必要であれば、Laravelが裏で使用しているMonologインスタンスへアクセスすることもできます。

    $monolog = Log::getMonolog();
