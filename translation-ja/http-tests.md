# HTTPテスト

- [イントロダクション](#introduction)
    - [リクエストヘッダのカスタマイズ](#customizing-request-headers)
- [セッション／認証](#session-and-authentication)
- [JSON APIのテスト](#testing-json-apis)
- [ファイルアップロードのテスト](#testing-file-uploads)
- [利用可能なアサート](#available-assertions)
    - [レスポンスのアサート](#response-assertions)
    - [認証のアサート](#authentication-assertions)

<a name="introduction"></a>
## イントロダクション

Laravelはアプリケーションに対するHTTPリクエストを作成し、出力を検査するためのとても記述的なAPIを用意しています。例として、以下のテストをご覧ください。

    <?php

    namespace Tests\Feature;

    use Tests\TestCase;
    use Illuminate\Foundation\Testing\RefreshDatabase;
    use Illuminate\Foundation\Testing\WithoutMiddleware;

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

アプリーケーションへ送り返す前に、リクエストヘッダをカスタマイズするには、`withHeaders`メソッドを使います。これにより任意のカスタムヘッダをリクエストに追加することができます。

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
    use Illuminate\Foundation\Testing\RefreshDatabase;
    use Illuminate\Foundation\Testing\WithoutMiddleware;

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

<a name="response-assertions"></a>
### レスポンスのアサート

[PHPUnit](https://phpunit.de/)テスト用に、数多くの追加アサートメソッドをLaravelは提供しています。以下のアサートで、`json`、`get`、`post`、`put`、`delete`テストメソッドから返されたレスポンスへアクセスしてください。

<style>
    .collection-method-list > p {
        column-count: 3; -moz-column-count: 3; -webkit-column-count: 3;
        column-gap: 2em; -moz-column-gap: 2em; -webkit-column-gap: 2em;
    }

    .collection-method-list a {
        display: block;
    }
</style>

<div class="collection-method-list" markdown="1">

[assertCookie](#assert-cookie)
[assertCookieExpired](#assert-cookie-expired)
[assertCookieMissing](#assert-cookie-missing)
[assertDontSee](#assert-dont-see)
[assertDontSeeText](#assert-dont-see-text)
[assertExactJson](#assert-exact-json)
[assertHeader](#assert-header)
[assertHeaderMissing](#assert-header-missing)
[assertJson](#assert-json)
[assertJsonFragment](#assert-json-fragment)
[assertJsonMissing](#assert-json-missing)
[assertJsonMissingExact](#assert-json-missing-exact)
[assertJsonStructure](#assert-json-structure)
[assertJsonValidationErrors](#assert-json-validation-errors)
[assertPlainCookie](#assert-plain-cookie)
[assertRedirect](#assert-redirect)
[assertSee](#assert-see)
[assertSeeText](#assert-see-text)
[assertSessionHas](#assert-session-has)
[assertSessionHasAll](#assert-session-has-all)
[assertSessionHasErrors](#assert-session-has-errors)
[assertSessionHasErrorsIn](#assert-session-has-errors-in)
[assertSessionMissing](#assert-session-missing)
[assertStatus](#assert-status)
[assertSuccessful](#assert-successful)
[assertViewHas](#assert-view-has)
[assertViewHasAll](#assert-view-has-all)
[assertViewIs](#assert-view-is)
[assertViewMissing](#assert-view-missing)

</div>

<a name="assert-cookie"></a>
#### assertCookie

レスポンスが指定したクッキーを持っていることをアサート。

    $response->assertCookie($cookieName, $value = null);

<a name="assert-cookie-expired"></a>
#### assertCookieExpired

レスポンスが指定したクッキーを持っており、期限切れであることをアサート。

    $response->assertCookieExpired($cookieName);

<a name="assert-cookie-missing"></a>
#### assertCookieMissing

レスポンスが指定したクッキーを持っていないことをアサート。

    $response->assertCookieMissing($cookieName);

<a name="assert-dont-see"></a>
#### assertDontSee

指定した文字列がレスポンスに含まれていないことをアサート。

    $response->assertDontSee($value);

<a name="assert-dont-see-text"></a>
#### assertDontSeeText

指定した文字列がレスポンステキストに含まれていないことをアサート。

    $response->assertDontSeeText($value);

<a name="assert-exact-json"></a>
#### assertExactJson

レスポンスが指定したJSONデータと完全に一致するデータを持っていることをアサート。

    $response->assertExactJson(array $data);

<a name="assert-header"></a>
#### assertHeader

レスポンスに指定したヘッダが存在していることをアサート。

    $response->assertHeader($headerName, $value = null);

<a name="assert-header-missing"></a>
#### assertHeaderMissing

レスポンスに指定したヘッダが存在していないことをアサート。

    $response->assertHeaderMissing($headerName);

<a name="assert-json"></a>
#### assertJson

レスポンスが指定したJSONデータを持っていることをアサート。

    $response->assertJson(array $data);

<a name="assert-json-fragment"></a>
#### assertJsonFragment

レスポンスが指定したJSONの一部を含んでいることをアサート。

    $response->assertJsonFragment(array $data);

<a name="assert-json-missing"></a>
#### assertJsonMissing

レスポンスが指定したJSONの一部を含んでいないことをアサート。

    $response->assertJsonMissing(array $data);

<a name="assert-json-missing-exact"></a>
#### assertJsonMissingExact

レスポンスがJSONの一部をそのまま含んでいないことをアサート。

    $response->assertJsonMissingExact(array $data);

<a name="assert-json-structure"></a>
#### assertJsonStructure

レスポンスが指定したJSONの構造を持っていることをアサート。

    $response->assertJsonStructure(array $structure);

<a name="assert-json-validation-errors"></a>
#### assertJsonValidationErrors

レスポンスが指定したキーの、指定したJSONバリデーションエラーを持っていることをアサート

    $response->assertJsonValidationErrors($keys);

<a name="assert-plain-cookie"></a>
#### assertPlainCookie

レスポンスが指定した暗号化されていないクッキーを持っていることをアサート。

    $response->assertPlainCookie($cookieName, $value = null);

<a name="assert-redirect"></a>
#### assertRedirect

クライアントが指定したURIへリダイレクトすることをアサート。

    $response->assertRedirect($uri);

<a name="assert-see"></a>
#### assertSee

指定した文字列がレスポンスに含まれていることをアサート。

    $response->assertSee($value);

<a name="assert-see-text"></a>
#### assertSeeText

指定した文字列がレスポンステキストに含まれていることをアサート。

    $response->assertSeeText($value);

<a name="assert-session-has"></a>
#### assertSessionHas

セッションが指定したデータを持っていることをアサート。

    $response->assertSessionHas($key, $value = null);

<a name="assert-session-has-all"></a>
#### assertSessionHasAll

セッションが指定したリストの値を持っていることをアサート。

    $response->assertSessionHasAll(array $data);

<a name="assert-session-has-errors"></a>
#### assertSessionHasErrors

セッションが指定したフィールドに対するエラーを含んでいることをアサート。

    $response->assertSessionHasErrors(array $keys, $format = null, $errorBag = 'default');

<a name="assert-session-has-errors-in"></a>
#### assertSessionHasErrorsIn

セッションが指定したエラーを持っていることをアサート。

    $response->assertSessionHasErrorsIn($errorBag, $keys = [], $format = null);

<a name="assert-session-missing"></a>
#### assertSessionMissing

セッションが指定したキーを持っていないことをアサート。

    $response->assertSessionMissing($key);

<a name="assert-status"></a>
#### assertStatus

クライアントのレスポンスが指定したコードであることをアサート。

    $response->assertStatus($code);

<a name="assert-successful"></a>
#### assertSuccessful

レスポンスが成功のステータスコードであることをアサート。

    $response->assertSuccessful();

<a name="assert-view-has"></a>
#### assertViewHas

レスポンスビューが指定したデータを持っていることをアサート。

    $response->assertViewHas($key, $value = null);

<a name="assert-view-has-all"></a>
#### assertViewHasAll

レスポンスビューが指定したリストのデータを持っていることをアサート。

    $response->assertViewHasAll(array $data);

<a name="assert-view-is"></a>
#### assertViewIs

ルートにより、指定したビューが返されたことをアサート。

    $response->assertViewIs($value);

<a name="assert-view-missing"></a>
#### assertViewMissing

レスポンスビューが指定したデータを持っていないことをアサート。

    $response->assertViewMissing($key);

<a name="authentication-assertions"></a>
### 認証のアサート

Laravelは、[PHPUnit](https://phpunit.de/)テストのために認証関連の様々なアサーションも提供しています。

メソッド           |                 説明
------------------------------------------ | --------------------------------------------
`$this->assertAuthenticated($guard = null);`  |  ユーザーが認証されていることをアサート。
`$this->assertGuest($guard = null);`  |  ユーザーが認証されていないことをアサート。
`$this->assertAuthenticatedAs($user, $guard = null);`  |  指定したユーザーが認証されていることをアサート。
`$this->assertCredentials(array $credentials, $guard = null);`  |  指定した認証情報が有効であることをアサート。
`$this->assertInvalidCredentials(array $credentials, $guard = null);`  |  指定した認証情報が無効であることをアサート。
