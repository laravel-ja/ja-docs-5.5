# HTTPテスト

- [イントロダクション](#introduction)
    - [Customizing Request Headers](#customizing-request-headers)
- [セッション／認証](#session-and-authentication)
- [JSON APIのテスト](#testing-json-apis)
- [ファイルアップロードのテスト](#testing-file-uploads)
- [利用可能なアサート](#available-assertions)

<a name="introduction"></a>
## イントロダクション

Laravelはアプリケーションに対するHTTPリクエストを作成し、出力を検査するためのとても記述的なAPIを用意しています。例として、以下のテストをご覧ください。

    <?php

    namespace Tests\Feature;

    use Tests\TestCase;
    use Illuminate\Foundation\Testing\WithoutMiddleware;
    use Illuminate\Foundation\Testing\DatabaseMigrations;
    use Illuminate\Foundation\Testing\DatabaseTransactions;

    class ExampleTest extends TestCase
    {
        /**
         * 基本的な機能テストの例
         *
         * @return void
         */
        public function testBasicTest()
        {
            $response = $this->get('/');

            $response->assertStatus(200);
        }
    }

`get`メソッドはアプリケーションに対して、`GET`リクエストを作成します。`assertStatus`メソッドは返されたレスポンスが指定したHTTPステータスコードを持っていることをアサートします。このシンプルな例に加え、レスポンスヘッダ、コンテンツ、JSON構造などを検査する様々なアサートをLaravelは用意しています。

<a name="customizing-request-headers"></a>
### リクエストヘッダのカスタマイズ

アプリーケーションに送られる前にリクエストヘッダをカスタマイズするには、 `withHeaders` メソッドを使います。これにより任意のカスタムヘッダをリクエストに追加することができます。

    <?php

    class ExampleTest extends TestCase
    {
        /**
         * 基本的な機能テストの例
         *
         * @return void
         */
        public function testBasicExample()
        {
            $response = $this->withHeaders([
                'X-Header' => 'Value',
            ])->json('POST', '/user', ['name' => 'Sally']);

            $response
                ->assertStatus(200)
                ->assertJson([
                    'created' => true,
                ]);
        }
    }

<a name="session-and-authentication"></a>
## セッション／認証

Laravelはテスト時にセッションを操作するたくさんのヘルパも提供しています。１つ目は指定した配列をセッションに保存する`withSession`メソッドです。これはアプリケーションのリクエストをテストする前に、データをセッションにロードしたい場合に便利です。

    <?php

    class ExampleTest extends TestCase
    {
        public function testApplication()
        {
            $response = $this->withSession(['foo' => 'bar'])
                             ->get('/');
        }
    }

もちろん認証済みのユーザーのようなユーザー状態をセッションへ保持するのは一般的です。`actingAs`ヘルパメソッドは現在認証済みのユーザーを指定する簡単な手段を提供します。例として、[モデルファクトリ](/docs/{{version}}/database-testing#writing-factories)でユーザーを生成し、認証してみましょう。

    <?php

    use App\User;

    class ExampleTest extends TestCase
    {
        public function testApplication()
        {
            $user = factory(User::class)->create();

            $response = $this->actingAs($user)
                             ->withSession(['foo' => 'bar'])
                             ->get('/');
        }
    }

ユーザーの認証にどのガードを使用するかを指定したい場合、`actingAs`メソッドの第２引数にガード名を渡します。

    $this->actingAs($user, 'api')

<a name="testing-json-apis"></a>
## JSON APIのテスト

LaravelはJSON APIとレスポンスをテストする数多くのヘルパを用意しています。たとえば、`json`、`get`、`post`、`put`、`patch`、`delete`メソッドはそれぞれのHTTP動詞のリクエストを発生させるために使用します。これらのメソッドには簡単にデータやヘッダを渡せます。手始めに、`/user`に対する`POST`リクエストを作成し、期待したデータが返されることをアサートするテストを書いてみましょう。

    <?php

    class ExampleTest extends TestCase
    {
        /**
         * 基本的な機能テスト例
         *
         * @return void
         */
        public function testBasicExample()
        {
            $response = $this->json('POST', '/user', ['name' => 'Sally']);

            $response
                ->assertStatus(200)
                ->assertJson([
                    'created' => true,
                ]);
        }
    }

> {tip} The `assertJson`メソッドはレスポンスを配列へ変換し、`PHPUnit::assertArraySubset`を使用しアプリケーションへ戻ってきたJSONレスポンスの中に、指定された配列が含まれているかを確認します。そのため、JSONレスポンスの中に他のプロパティが存在していても、このテストは指定した一部が残っている限り、テストはパスし続けます。

<a name="verifying-exact-match"></a>
### JSONとの完全一致を検証

アプリケーションから返されるJSONが、指定した配列と**完全に**一致することを検証したい場合は、`assertExactJson`メソッドを使用します。

    <?php

    class ExampleTest extends TestCase
    {
        /**
         * 基本的な機能テスト例
         *
         * @return void
         */
        public function testBasicExample()
        {
            $response = $this->json('POST', '/user', ['name' => 'Sally']);

            $response
                ->assertStatus(200)
                ->assertExactJson([
                    'created' => true,
                ]);
        }
    }

<a name="testing-file-uploads"></a>
## ファイルアップロードのテスト

`Illuminate\Http\UploadedFile`クラスは、テストのためにファイルやイメージのダミーを生成するための`fake`メソッドを用意しています。これを`Storage`ファサードの`fake`メソッドと組み合わせることで、ファイルアップロードのテストがとてもシンプルになります。例として、２つの機能を組み合わせて、アバターのアップロードをテストしてみましょう。

    <?php

    namespace Tests\Feature;

    use Tests\TestCase;
    use Illuminate\Http\UploadedFile;
    use Illuminate\Support\Facades\Storage;
    use Illuminate\Foundation\Testing\WithoutMiddleware;
    use Illuminate\Foundation\Testing\DatabaseMigrations;
    use Illuminate\Foundation\Testing\DatabaseTransactions;

    class ExampleTest extends TestCase
    {
        public function testAvatarUpload()
        {
            Storage::fake('avatars');

            $response = $this->json('POST', '/avatar', [
                'avatar' => UploadedFile::fake()->image('avatar.jpg')
            ]);

            // ファイルが保存されたことをアサートする
            Storage::disk('avatars')->assertExists('avatar.jpg');

            // ファイルが存在しないことをアサートする
            Storage::disk('avatars')->assertMissing('missing.jpg');
        }
    }

#### ダミーファイルのカスタマイズ

`fake`メソッドでファイルを生成するときには、バリデーションルールをより便利にテストできるように、画像の幅、高さ、サイズを指定できます。

    UploadedFile::fake()->image('avatar.jpg', $width, $height)->size(100);

画像の生成に付け加え、`create`メソッドで他のタイプのファイルも生成できます。

    UploadedFile::fake()->create('document.pdf', $sizeInKilobytes);

<a name="available-assertions"></a>
## 利用可能なアサート

[PHPUnit](https://phpunit.de/)テスト用に、数多くの追加アサートメソッドをLaravelは提供しています。以下のアサートで、`json`、`get`、`post`、`put`、`delete`テストメソッドから返されたレスポンスへアクセスしてください。

メソッド               |                           説明
-------------------------------|---------------------------------
`$response->assertSuccessful();`  |  レスポンスが成功のステータスコードであることをアサート。
`$response->assertStatus($code);`  |  クライアントのレスポンスが指定したコードであることをアサート。
`$response->assertRedirect($uri);`  |  クライアントが指定したURIへリダイレクトすることをアサート。
`$response->assertHeader($headerName, $value = null);`  |  レスポンスに指定したヘッダが存在していることをアサート。
`$response->assertCookie($cookieName, $value = null);`  |  レスポンスが指定したクッキーを持っていることをアサート。
`$response->assertPlainCookie($cookieName, $value = null);`  |  レスポンスが指定した暗号化されていないクッキーを持っていることをアサート。
`$response->assertSessionHas($key, $value = null);`  |  セッションが指定したデータを持っていることをアサート。
`$response->assertSessionHasErrors(array $keys, $errorBag = 'default');`  |  指定したフィールドに対するエラーがセッションに含まれていることをアサート。
`$response->assertSessionMissing($key);`  |  セッションが指定したキーを持っていないことをアサート。
`$response->assertJson(array $data);`  |  レスポンスが指定したJSONデータを持っていることをアサート。
`$response->assertJsonFragment(array $data);`  |  レスポンスが指定したJSONの一部を含んでいることをアサート。
`$response->assertJsonMissing(array $data);`  |  レスポンスが指定したJSONの一部を含んでいないことをアサート。
`$response->assertExactJson(array $data);`  |  レスポンスが指定したJSONデータと完全に一致するデータを持っていることをアサート。
`$response->assertJsonStructure(array $structure);`  |  レスポンスが指定したJSONの構造を持っていることをアサート。
`$response->assertViewIs($value);`  |  ルートにより、指定したビューが返されたことをアサート。
`$response->assertViewHas($key, $value = null);`  |  レスポンスビューが指定したデータを持っていることをアサート。
