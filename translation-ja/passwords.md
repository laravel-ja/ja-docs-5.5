# パスワードリセット

- [イントロダクション](#introduction)
- [データベースの検討事項](#resetting-database)
- [ルート定義](#resetting-routing)
- [ビュー](#resetting-views)
- [パスワードリセット後の処理](#after-resetting-passwords)
- [カスタマイズ](#password-customization)

<a name="introduction"></a>
## イントロダクション

> {tip} **さっそく取り掛かりたいですか？** 真新しいLaravelアプリケーションで`php artisan make:auth`を実行し、ブラウザで`http://your-app.dev/register`か、アプリケーションで割りつけた別のURLへアクセスするだけです。この一つのコマンドで、パスワードリセットを含めた、認証システム全体のスカフォールディングの面倒をみます。

大抵のWebアプリケーションはパスワードをリセットする手段を提供しています。それぞれのアプリケーションで何度も実装する代わりに、Laravelはパスワードリマインダを送り、パスワードリセットを実行する便利な方法を提供しています。

> {note} Laravelのパスワードリセット機能を使用開始する前に、ユーザーが`Illuminate\Notifications\Notifiable`トレイトを使用していることを確認してください。

<a name="resetting-database"></a>
## データベースの検討事項

利用開始するには、`App\User`モデルが`Illuminate\Contracts\Auth\CanResetPassword`契約を実装しているか確認してください。もちろん、フレームワークに用意されている`App\User`モデルでは、既にこのインターフェイスが実装されています。`Illuminate\Auth\Passwords\CanResetPassword`トレイトで、このインターフェイスで実装する必要のあるメソッドが定義されています。

#### リセットトークンテーブルマイグレーションの生成

次にパスワードリセットトークンを保存しておくためのテーブルを作成します。このテーブルのマイグレーションは、最初からLaravelの`database/migrations`ディレクトリに含まれています。ですから、データベースマイグレートするために必要なのは次のコマンド実行だけです。

    php artisan migrate

<a name="resetting-routing"></a>
## ルート定義

Laravelはパスワードリセットリンクのメールを送信し、ユーザーのパスワードをリセットするために必要なロジックを全部含んでいる、`Auth\ForgotPasswordController`と`Auth\PasswordController`を用意しています。パスワードリセットに必要な全ルートは、`make:auth` Artisanコマンドで生成します。

    php artisan make:auth

<a name="resetting-views"></a>
## ビュー

ビューについても、`make:auth`コマンドを実行すれば、パスワードリセットに必要な全てが生成されます。ビューは`resources/views/auth/passwords`に生成されます。アプリケーションに合わせ、自由にカスタマイズしてください。

<a name="after-resetting-passwords"></a>
## パスワードリセット後の処理

ユーザーのパスワードをリセットするルートとビューを定義できたら、ブラウザーで`/password/reset`のルートへアクセスできます。フレームワークに含まれている `ForgotPasswordController`は、パスワードリセットリンクを含むメールを送信するロジックを含んでいます。一方の`ResetPasswordController`はユーザーパスワードのリセットロジックを含んでいます。

パスワードがリセットされたら、そのユーザーは自動的にアプリケーションにログインされ、`/home`へリダイレクトされます。パスワードリセット後のリダイレクト先をカスタマイズするには、`ResetPasswordController`の`redirectTo`プロパティを定義してください。

    protected $redirectTo = '/dashboard';

> {note} デフォルトでパスワードリセットトークンは、一時間有効です。これは、`config/auth.php`ファイルの`expire`オプションにより変更できます。

<a name="password-customization"></a>
## カスタマイズ

#### 認証ガードのカスタマイズ

`auth.php`設定ファイルにより、複数のユーザーテーブルごとに認証の振る舞いを定義するために使用する、「ガード」をそれぞれ設定できます。用意されている`ResetPasswordController`コントローラの`guard`メソッドをオーバーライドすることにより、選択したガードを使用するようにカスタマイズできます。このメソッドは、ガードインスタンスを返す必要があります。

    use Illuminate\Support\Facades\Auth;

    protected function guard()
    {
        return Auth::guard('guard-name');
    }

#### パスワードブローカーのカスタマイズ

複数のユーザーテーブルに対するパスワードをリセットするために使用する、別々のパスワード「ブローカー」を`auth.php`ファイルで設定できます。用意されている`ForgotPasswordController`と`ResetPasswordController`の`broker`メソッドをオーバーライドし、選んだブローカーを使用するようにカスタマイズができます。

    use Illuminate\Support\Facades\Password;

    /**
     *パスワードリセットに使われるブローカの取得
     *
     * @return PasswordBroker
     */
    protected function broker()
    {
        return Password::broker('name');
    }

#### リセットメールのカスタマイズ

パスワードリセットリンクをユーザーへ送るために使用する、通知クラスは簡単に変更できます。手始めに、`User`モデルの`sendPasswordResetNotification`メソッドをオーバーライドしましょう。このメソッドの中で、皆さんが選んだ通知クラスを使用し、通知を送信できます。パスワードリセット`$token`は、メソッドの第1引数として受け取ります。

    /**
     * パスワードリセット通知の送信
     *
     * @param  string  $token
     * @return void
     */
    public function sendPasswordResetNotification($token)
    {
        $this->notify(new ResetPasswordNotification($token));
    }

