# 暗号化

- [イントロダクション](#introduction)
- [設定](#configuration)
- [エンクリプタの使用](#using-the-encrypter)

<a name="introduction"></a>
## イントロダクション

Laravelのエンクリプタ(encrypter)はOpenSSLを使い、AES-256とAES-128暗号化を提供しています。Laravelに組み込まれている暗号化機能を使用し、「自前」の暗号化アルゴリズムに走らないことを強くおすすめします。Laravelの全暗号化済み値は、メッセージ認証コード(MAC)を使用し署名され、一度暗号化されると値を変更できません。

<a name="configuration"></a>
## 設定

Laravelのエンクリプタを使用する準備として、`config/app.php`設定ファイルの`key`オプションをセットしてください。`php artisan key:generate`コマンドを使用し、このキーを生成すべきです。このArtisanコマンドはPHPの安全なランダムバイトジェネレータを使用し、キーを作成します。この値が確実に指定されていないと、Laravelにより暗号化された値は、すべて安全ではありません。

<a name="using-the-encrypter"></a>
## エンクリプタの使用

#### 値の暗号化

`encrypt`ヘルパを使用し、値を暗号化できます。OpenSSLと`AES-256-CBC`アルゴリズムが使用され、全ての値は暗号化されます。さらに、全暗号化済み値はメッセージ認証コード(MAC)を使用し署名されますので、暗号化済み値の変更は感知されます。

    <?php

    namespace App\Http\Controllers;

    use App\User;
    use Illuminate\Http\Request;
    use App\Http\Controllers\Controller;

    class UserController extends Controller
    {
        /**
         * ユーザーの秘密のメッセージを保存
         *
         * @param  Request  $request
         * @param  int  $id
         * @return Response
         */
        public function storeSecret(Request $request, $id)
        {
            $user = User::findOrFail($id);

            $user->fill([
                'secret' => encrypt($request->secret)
            ])->save();
        }
    }

#### シリアライズしない暗号化

暗号化の過程で暗号化する値は「シリアライズ(`serialize`)」されます。これにより、オブジェクトや配列の暗号化が可能になります。そのため、PHPではないクライアントで暗号化された値を受け取る場合、データを「非シリアライズ(`unserialize`)」する必要が起きるでしょう。もし、シリアライズせずに値を暗号化／復号したい場合は、`Crypt`ファサードの`encryptString`と`decryptString`を使用してください。

    use Illuminate\Support\Facades\Crypt;

    $encrypted = Crypt::encryptString('Hello world.');

    $decrypted = Crypt::decryptString($encrypted);

#### 値の復号

`decrypt`ヘルパにより、値を復号することができます。MACが無効な場合など、その値が正しくない時は`Illuminate\Contracts\Encryption\DecryptException`が投げられます。

    use Illuminate\Contracts\Encryption\DecryptException;

    try {
        $decrypted = decrypt($encryptedValue);
    } catch (DecryptException $e) {
        //
    }
