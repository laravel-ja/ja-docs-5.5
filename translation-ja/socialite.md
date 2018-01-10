# Laravel Socialite

- [イントロダクション](#introduction)
- [インストール](#installation)
- [設定](#configuration)
- [ルート](#routing)
- [オプションのパラメータ](#optional-parameters)
- [アクセススコープ](#access-scopes)
- [ステートレスな認証](#stateless-authentication)
- [ユーザー詳細情報の取得](#retrieving-user-details)

<a name="introduction"></a>
## イントロダクション

典型的なフォームベースの認証に付け加え、Laravelは[Laravel Socialite](https://github.com/laravel/socialite)(=名士)による、OAuthプロバイダによるシンプルで便利な認証方法も提供しています。Socialiteは現在、Facebook、Twitter、LinkedIn、Google、GitHub、Bitbucketによる認証をサポートしています。

> {tip} 他のプラットフォームのアダプタは、コミュニティにより管理されている[Socialiteプロバイダ](https://socialiteproviders.github.io/)Webサイトで一覧できます。

<a name="installation"></a>
## インストール

Socialiteを使用開始するには、Composerを使い、プロジェクトの依存パッケージに追加してください。

    composer require laravel/socialite

<a name="configuration"></a>
## 設定

Socialiteを使用する前に、アプリケーションで使用するOAUthサービスの認証情報も追加する必要があります。認証情報は`config/services.php`設定ファイルで指定し、アプリケーションの必要に合わせ、`facebook`、`twitter`、`linkedin`、`google`、`github`、`bitbucket`キーを使用してください。一例をご覧ください。

    'github' => [
        'client_id' => env('GITHUB_CLIENT_ID'),         // GitHubのClient ID
        'client_secret' => env('GITHUB_CLIENT_SECRET'), // GitHubのClient Secret
        'redirect' => 'http://your-callback-url',
    ],

> {tip} If the `redirect` option contains a relative path, it will automatically be resolved to a fully qualified URL.

<a name="routing"></a>
## ルート

これで、ユーザーを認証する準備ができました。２つのルートが必要になります。ひとつはOAuthプロバイダへユーザーをリダイレクトするルート、もう一つは認証後にプロバイダーからのコールバックを受けるルートです。`Socialite`ファサードを使用し、Socialiteにアクセスしましょう。

    <?php

    namespace App\Http\Controllers\Auth;

    use Socialite;

    class LoginController extends Controller
    {
        /**
         * GitHubの認証ページヘユーザーをリダイレクト
         *
         * @return \Illuminate\Http\Response
         */
        public function redirectToProvider()
        {
            return Socialite::driver('github')->redirect();
        }

        /**
         * GitHubからユーザー情報を取得
         *
         * @return \Illuminate\Http\Response
         */
        public function handleProviderCallback()
        {
            $user = Socialite::driver('github')->user();

            // $user->token;
        }
    }

`redirect`メソッドはユーザーをOAuthプロバイダへ送るのを担当します。一方の`user`メソッドは送られて来たリクエストを読み、プロバイダからユーザーの情報を取得します。

もちろん、コントローラメソッドへのルートを定義する必要があります。

    Route::get('login/github', 'Auth\LoginController@redirectToProvider');
    Route::get('login/github/callback', 'Auth\LoginController@handleProviderCallback');

<a name="optional-parameters"></a>
## オプションのパラメータ

多くのOAuthプロバイダがリダイレクトリクエスト中のオプションパラメータをサポートしています。リクエストにオプションパラメータを含めるには、`with`メソッドを呼び出し、連想配列を渡します。

    return Socialite::driver('google')
        ->with(['hd' => 'example.com'])
        ->redirect();

> {note} `with`メソッドを使用時は、`state`や`response_type`などの予約キーワードを渡さないように注意してください。

<a name="access-scopes"></a>
## アクセススコープ

ユーザーをリダイレクトする前に、`scopes`メソッドを使用し、リクエストへ「スコープ」を追加することもできます。このメソッドは、存在する全スコープを皆さんが指定したものへマージします。

    return Socialite::driver('github')
        ->scopes(['read:user', 'public_repo'])
        ->redirect();

`setScopes`メソッドを使用し、存在するスコープを全てオーバーライトできます。

    return Socialite::driver('github')
        ->setScopes(['read:user', 'public_repo'])
        ->redirect();

<a name="stateless-authentication"></a>
## ステートレスな認証

`stateless`メソッドはセッション状態の確認を無効にするために使用します。これはAPIへソーシャル認証を追加する場合に便利です。

    return Socialite::driver('google')->stateless()->user();

<a name="retrieving-user-details"></a>
## ユーザー詳細情報の取得

ユーザーインスタンスを取得したら、詳細情報をもっと取得できます。

    $user = Socialite::driver('github')->user();

    // OAuth Twoプロバイダ
    $token = $user->token;
    $refreshToken = $user->refreshToken; // not always provided
    $expiresIn = $user->expiresIn;

    // OAuth Oneプロバイダ
    $token = $user->token;
    $tokenSecret = $user->tokenSecret;

    // 全プロバイダ
    $user->getId();
    $user->getNickname();
    $user->getName();
    $user->getEmail();
    $user->getAvatar();

#### トークンからのユーザー詳細情報の取得(OAuth2)

ユーザーへの有効なアクセストークンを事前に取得している場合は、`userFromToken`メソッドを用い、詳細を取得できます。

    $user = Socialite::driver('github')->userFromToken($token);

#### トークンとSecretからのユーザー詳細情報の取得(OAuth1)

すでにユーザーに対する有効なトークン／secretペアを取得している場合は、`userFromTokenAndSecret`メソッドを用い、詳細を取得できます。

    $user = Socialite::driver('twitter')->userFromTokenAndSecret($token, $secret);
