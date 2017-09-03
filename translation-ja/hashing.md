# ハッシュ

- [イントロダクション](#introduction)
- [基本的な使用法](#basic-usage)

<a name="introduction"></a>
## イントロダクション

Laravelの`Hash`[ファサード](/docs/{{version}}/facades)は保存するユーザーパスワードに対し、安全なBcryptハッシュを提供します。Laravelアプリケーションに組み込まれている、`LoginController`と`RegisterController`を使用していれば、登録と認証で自動的にBcrypt使用します。

> {tip} Bcryptは「ストレッチ回数」が調整できるのでパスワードのハッシュには良い選択肢です。つまりハードウェアのパワーを上げればハッシュの生成時間を早くすることができます。

<a name="basic-usage"></a>
## 基本的な使用法

`Hash`ファサードの`make`メソッドを呼び出し、パスワードをハッシュできます。

    <?php

    namespace App\Http\Controllers;

    use Illuminate\Http\Request;
    use Illuminate\Support\Facades\Hash;
    use App\Http\Controllers\Controller;

    class UpdatePasswordController extends Controller
    {
        /**
         * ユーザーパスワードを更新
         *
         * @param  Request  $request
         * @return Response
         */
        public function update(Request $request)
        {
            // 新しいパスワードの長さのバリデーション…

            $request->user()->fill([
                'password' => Hash::make($request->newPassword)
            ])->save();
        }
    }

さらに、`make`メソッドでは`rounds`オプションを使い、bcryptハッシュアルゴリズムのワークファクタを管理できます。しかしながら、多くのアプリケーションではデフォルトのままで、構わないでしょう。

    $hashed = Hash::make('password', [
        'rounds' => 12
    ]);

#### パスワードとハッシュ値の比較

`check`メソッドにより指定した平文文字列と指定されたハッシュ値を比較確認できます。しかし[Laravelに含まれている](/docs/{{version}}/authentication)`LoginController`を使っている場合は、これを直接使用することはないでしょう。このコントローラーがこのメソッドを自動的に呼び出します。

    if (Hash::check('plain-text', $hashedPassword)) {
        // パスワード一致
    }

#### パスワードの再ハッシュが必要か確認

パスワードがハシュされてからハッシャーのストレッチ回数が変更されているかを調べるには、`needsRehash`メソッドを使います。

    if (Hash::needsRehash($hashed)) {
        $hashed = Hash::make('plain-text');
    }
