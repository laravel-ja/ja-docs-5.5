# アップグレードガイド

- [5.5から、5.5.42へのアップグレード](#upgrade-5.5.42)
- [5.4から5.5.0へのアップグレード](#upgrade-5.5.0)

<a name="upgrade-5.5.42"></a>
## 5.5から、5.5.42へのアップグレード（セキュリティーリリース）

Laravel5.5.42はセキュリティーリリースのため、すべてのユーザーに対して即時にアップグレードすることを推奨します。また、Laravel5.5.42はクッキーの暗号化とシリアライズに関するロジックに、互換性がない変更を含んでいるため、アプリケーションのアップグレード時に以降の注釈を注意深く読んでください。

**この脆弱性はアプリケーションの暗号化キー（`APP_KEY`環境変数）が悪意のあるユーザーからアクセスされる場合にのみ悪用されます。** 通常、アプリケーションのユーザーが、この値にアクセスできる可能性はありません。しかしながら、以前雇用していた人が暗号化キーにアクセスできていた場合、アプリケーションへ攻撃するためにこのキーを利用することができます。悪意のある人々の手に、暗号化キーが渡ったと信じるいかなる理由がある場合は**いつでも**、キーの値を変更すべきです。

### クッキーのシリアライズ化

Laravel5.5.42では、クッキー値のシリアライズ化／非シリアライズ化を無効にしました。その理由は、Laravelのすべてのクッキーは、暗号化し、署名してあり、クライアントによる改ざんに対し、通常安全であると考えられます。**しかしながら、アプリケーションの暗号化キーが悪意のある人々の手に入れば、彼らは暗号化キーを使用しクッキー値を生成できます。アプリケーションの中の任意のクラスメソッドを呼び出すことなど、PHPオブジェクトのシリアライズ化／非シリアライズ化に継承される脆弱性を悪用できてしまいます。**

すべてのクッキー値に対するシリアライズ化を無効にすることで、アプリケーションのセッションはすべて無効になり、ユーザーはアプリケーションに再ログインする必要が起きます。更に、アプリケーションで使用している他の暗号化クッキーの設定値が、無効になります。このため、カスタムクッキー値が、期待する値のリストと一致する事をバリデートするロジックをアプリケーションに追加する必要があるかも知れません。そうしない場合、全てを破棄することになります。

#### クッキーのシリアライズ化の設定

アプリケーションの暗号化キーにアクセスできない場合、この脆弱性は悪用できないため、今回の変更に対してもアプリケーションの互換性を維持できるように、暗号化クッキーのシリアライズ化を最有効にする手段を提供します。クッキーのシリアライズ化を有効／無効にするには、`App\Http\Middleware\EncryptCookies` [ミドルウェア](https://github.com/laravel/laravel/blob/5.5/app/Http/Middleware/EncryptCookies.php)の`serialize`スタティックプロパティを変更してください。

    /**
     * クッキーをシリアライズするかの指定
     *
     * @var bool
     */
    protected static $serialize = true;

> **注意** 暗号化されたクッキーのシリアライズ化を有効にし、暗号化キーが悪意のある他者からアクセスできる場合、攻撃される可能性のある脆弱性がアプリケーションに起きます。キーが悪意のある人々の手に入ったと確信できる場合は、暗号化されたクッキーのシリアライズを有効にする前に、キーを新しい値に変更すべきです。

<a name="upgrade-5.5.0"></a>
## 5.4から5.5.0へのアップグレード

#### アップデートの見積もり時間：１時間

> {note} 私達は、互換性を失う可能性がある変更を全部ドキュメントにしようとしています。しかし、変更点のいくつかは、フレームワークの明確ではない部分で行われているため、一部の変更が実際にアプリケーションに影響を与えてしまう可能性があります。

### PHP

Laravel5.5では、PHP7.0.0以上が必要です。

### 依存パッケージのアップデート

`composer.json`ファイル中の、`laravel/framework`依存指定を`5.5.*`へ変更してください。さらに、`phpunit/phpunit`の依存指定を`~6.0`へ更新してください。次に、`composer.json`ファイルの`require-dev`セクションに、`filp/whoops`パッケージのバージョン`~2.0`を追加します。最後に、`composer.json`ファイルの`scripts`セクションで、`post-autoload-dump`イベントに対し`package:discover`コマンドを追加します。

    "scripts": {
        ...
        "post-autoload-dump": [
            "Illuminate\\Foundation\\ComposerScripts::postAutoloadDump",
            "@php artisan package:discover"
        ],
    }

`laravel/browser-kit-testing`パッケージを使用している場合は、`composer.json`ファイルの中で、パッケージのバージョンを`2.*`へアップデートしてください。

もちろん、アプリケーションで使用しているサードパーティ製パッケージを調べ、Laravel5.5に適切なバージョンを使っていることも忘れず確認してください。

#### Laravelインストーラ

> {tip} `laravel new`を使い、Laravelインストーラによりインストールしている方は、`composer global update`コマンドにより、インストーラパッケージを更新してください。

#### Laravel Dusk

Laravel5.5とヘッドレスChromeでのテストに対し互換性のある、Laravel Dusk `2.0.0`がリリースされました。

#### Pusher

Pusherイベントブロードキャストドライバーは、Pusher SDKのバージョン`~3.0`が必要となりました。

#### Swift Mailer

Laravel5.5では、バージョン`~6.0`のSwift Mailerが必要です。

### Artisan

#### コマンドのオートロード

Laravel5.5では、カーネルへ一々登録しなくても、Artisanはコマンドを自動的に見つけ出します。この新機能を利用するには、以下の行を`App\Console\Kernel`クラスの`commands`メソッドに追加してください。

    $this->load(__DIR__.'/Commands');

#### `fire`メソッド

Artisanコマンド中の`fire`メソッドは、`handle`へリネームしてください。

#### `optimize`コマンド

最近のPHPのオペコードキャッシュの向上により、`optimize` Artisanコマンドは必要なくなりました。将来のリリースでLaravelから削除されるため、デプロイスクリプトからこのコマンドを外してください。

### 認証

> {note} Laravel5.4から5.5へアップグレードすると、すべての`remember_me`クッキーは無効になり、ユーザーはログアウトされます。

#### `authorizeResource`コントローラメソッド

`authorizeResource`メソッドへ複数の単語のモデル名を渡したとき、リソースコントローラの振る舞いと合わせるため、ルートセグメントは「スネーク」ケースになりました。

#### `before`ポリシーメソッド

ポリシークラスにチェックしようとするアビリティと一致するメソッドが含まれていない場合、`before`メソッドは呼び出されなくなりました。

### キャッシュ

#### データベースドライバ

データベースキャッシュドライバを使用している場合、Laravel5.5へアップグレードしたアプリケーションを最初にデプロイする時に、`php artisan cache:clear`を実行してください。

### Eloquent

#### `belongsToMany`メソッド

Eloquentモデルの`belongsToMany`メソッドをオーバーライドしている場合、新しく追加された引数を反映するように、引数を更新してください。

    /**
     * 多対多リレーションの定義
     *
     * @param  string  $related
     * @param  string  $table
     * @param  string  $foreignPivotKey
     * @param  string  $relatedPivotKey
     * @param  string  $parentKey
     * @param  string  $relatedKey
     * @param  string  $relation
     * @return \Illuminate\Database\Eloquent\Relations\BelongsToMany
     */
    public function belongsToMany($related, $table = null, $foreignPivotKey = null,
                                  $relatedPivotKey = null, $parentKey = null,
                                  $relatedKey = null, $relation = null)
    {
        //
    }

#### BelongsToMany `getQualifiedRelatedKeyName`

`getQualifiedRelatedKeyName`メソッドは、`getQualifiedRelatedPivotKeyName`へリネームされました。

#### BelongsToMany `getQualifiedForeignKeyName`

`getQualifiedForeignKeyName`メソッドは、`getQualifiedForeignPivotKeyName`へリネームされました。

#### モデルの`is`メソッド

Eloquentモデルの`is`メソッドをオーバーライドしている場合、メソッドから`Model`のタイプヒントを削除してください。これにより、`is`メソッドが`null`を引数に取れるようになります。

    /**
     * ２つのモデルが同じIDで、同じテーブルに属しているか判定
     *
     * @param  \Illuminate\Database\Eloquent\Model|null  $model
     * @return bool
     */
    public function is($model)
    {
        //
    }

#### モデルの`$events`プロパティ

モデルの`$events`プロパティを`$dispatchesEvents`へリネームしてください。多くのユーザーが`events`リレーションを定義する必要がありましたが、古いプロパティ名と競合していたため、変更されました。

#### Pivotの`$parent`プロパティ

`Illuminate\Database\Eloquent\Relations\Pivot`クラスの`$parent` protectedプロパティを`$pivotParent`へ変更してください。

#### リレーションの`create`メソッド

`BelongsToMany`と`HasOneOrMany`、`MorphOneOrMany`クラスの`create`メソッドは、`$attributes`引数にデフォルト値を取るように変更されました。このメソッドをオーバーライドしている場合は、新しい定義に合わせて、引数を変更してください。

    public function create(array $attributes = [])
    {
        //
    }

#### モデルのソフトデリート

モデルの「ソフトデリート」時に、モデルの`exists`プロパティは`true`のままになりました。

#### `withCount`カラム形式

エイリアス使用時、`withCount`メソッドは、結果のカラム名に自動的に`_count`を付けなくなりました。たとえば、Laravel5.4ではクエリに`bar_count`カラムが結果に追加されていました。

    $users = User::withCount('foo as bar')->get();

しかし、Laravel5.5では、エイリアスは指定された通りに使用されます。結果のカラムに、`_count`を付けたい場合は、エイリアスを定義する時に、明確にサフィックスを指定します。

    $users = User::withCount('foo as bar_count')->get();

#### モデルのメソッドと属性名

配列アクセスを使う場合に、モデルのプライベートプロパティがアクセスされることを防ぐため、属性やプロパティと同じ名前のモデルメソッドを使えなくなりました。配列アクセス(`$user['name']`)や`data_get`ヘルパ関数により、モデルの属性へアクセスすることで、これが起きると例外が発生します。

### 例外フォーマット

Laravel5.5では、バリデーション例外を含むすべての例外は、例外ハンドラによりHTTPレスポンスへ変換されています。更に、JSONバリデーションエラーのデフォルトフォーマットが変更されました。新しいフォーマットは、以下の規約にしたがっています。

    {
        "message": "The given data was invalid.",
        "errors": {
            "field-1": [
                "Error 1",
                "Error 2"
            ],
            "field-2": [
                "Error 1",
                "Error 2"
            ],
        }
    }

Laraverl5.4のJSONエラーフォーマットをそのまま使用したい場合は、`App\Exceptions\Handler`クラスへ以下のメソッドを追加してください。

    use Illuminate\Validation\ValidationException;

    /**
     * バリデーション例外をJSONレスポンスへ変換
     *
     * @param  \Illuminate\Http\Request  $request
     * @param  \Illuminate\Validation\ValidationException  $exception
     * @return \Illuminate\Http\JsonResponse
     */
    protected function invalidJson($request, ValidationException $exception)
    {
        return response()->json($exception->errors(), $exception->status);
    }

#### JSON認証試行

この変更は、JSONで試みる認証のバリデーションエラーフォーマットにも影響を与えます。Laravel5.5では、上記で説明した新しいフォーマット規約に従い、JSON認証に失敗したエラーメッセージも送り返されます。

#### フォームRequestsの注意

個別のフォームリクエストの、レスポンスフォーマットをカスタマイズしている場合、`failedValidation`メソッドをオーバーライドし、カスタムレスポンスを含めた`HttpResponseException`例外インスタンスを投げてください。

    use Illuminate\Http\Exceptions\HttpResponseException;

    /**
     * バリデーション試行の失敗の処理
     *
     * @param  \Illuminate\Contracts\Validation\Validator  $validator
     * @return void
     *
     * @throws \Illuminate\Validation\ValidationException
     */
    protected function failedValidation(Validator $validator)
    {
        throw new HttpResponseException(response()->json(..., 422));
    }

### ファイルシステム

#### `files`メソッド

`Illuminate\Filesystem\Filesystem`クラスの`files`メソッドは、`$hidden`引数が追加され、`allFiles`メソッドのように`SplFileInfo`オブジェクトの配列を返すようになりました。以前は、パス名の配列を返していました。新しい引数は以下の通りです。

    public function files($directory, $hidden = false)

### メール

#### 未使用のパラメータ

`Illuminate\Contracts\Mail\MailQueue`契約の`queue`と`later`メソッドから、未使用の`$data`と`$callback`引数が削除されました。

    /**
     * 送信する新しいメールメッセージをキュー
     *
     * @param  string|array|MailableContract  $view
     * @param  string  $queue
     * @return mixed
     */
    public function queue($view, $queue = null);

    /**
     * （ｎ）秒後に送信する新しいメールメッセージをキュー
     *
     * @param  \DateTimeInterface|\DateInterval|int  $delay
     * @param  string|array|MailableContract  $view
     * @param  string  $queue
     * @return mixed
     */
    public function later($delay, $view, $queue = null);

### キュー

#### `dispatch`ヘルパ

即時に実行するジョブをディスパッチし、`handle`メソッドから値を返す場合は、ジョブのディスパッチに`dispatch_now`か、`Bus::dispatchNow`メソッドを使用してください。

    use Illuminate\Support\Facades\Bus;

    $value = dispatch_now(new Job);

    $value = Bus::dispatchNow(new Job);

### リクエスト

#### `all`メソッド

`Illuminate\Http\Request`の`all`メソッドをオーバーライドしてる場合は、新しい`$keys`引数の使用方法を反映してください。

    /**
     * リクエストの入力とファイルをすべて取得
     *
     * @param  array|mixed  $keys
     * @return array
     */
    public function all($keys = null)
    {
        //
    }

#### `has`メソッド

`$request->has`メソッドは、入力が空文字列や`null`であっても`true`を返すようになりました。以前の`has`の振る舞いを提供するため、新しい`$request->filled`メソッドが追加されました。

#### `intersect`メソッド

`intersect`メソッドは削除されました。この動作を模倣するため、`$request->only`の引数で、`array_filter`を使用してください。

    return array_filter($request->only('foo'));

#### `only`メソッド

`only`メソッドはリクエストペイロードに実際に存在する属性のみの返すようになりました。`only`メソッドの古い振る舞いを使いたい場合は、`all`メソッドを代わりに使用してください。

    return $request->all('foo');

#### `request()`ヘルパ

`request`ヘルパはネストしたキーを受け取らなくなりました。この振る舞いが必要であれば、リクエストの`input`メソッドを使用してください。

    return request()->input('filters.date');

### テスト

#### 認証のアサート

いくつかの認証アサートは、フレームワークの他とより一貫性を持たせるため、名前を変更しました。

<div class="content-list" markdown="1">
- `seeIsAuthenticated`は、`assertAuthenticated`へ変更
- `dontSeeIsAuthenticated`は、`assertGuest`へ変更
- `seeIsAuthenticatedAs`は、`assertAuthenticatedAs`へ変更
- `seeCredentials`は、`assertCredentials`へ変更
- `dontSeeCredentials`は、`assertInvalidCredentials`へ変更
</div>

#### Mail Fake

`Mail` fakeをリクエスト中にMailableが**キューされた**ことを判定するために使用しているなら、`Mail::assertSent`の代わりに`Mail::assertQueued`を使用してください。この区別により、メールがバックグランド送信のためにキューされたが、リクエスト間では送られないことをアサートで指定できるようになりました。

#### Tinker

Laravel Tinkerはアプリケーションクラスを参照する際の、省略した名前空間をサポートするようになりました。この機能にはComposerクラスマップの最適化が必要であるため、`composer.json`ファイルの`config`セクションへ、`optimize-autoloader`ディレクティブを追加する必要があります。

    "config": {
        ...
        "optimize-autoloader": true
    }

### 翻訳

#### `LoaderInterface`

`Illuminate\Translation\LoaderInterface`インターフェイスは、`Illuminate\Contracts\Translation\Loader`へ変更になりました。

### バリデーション

#### バリデータメソッド

バリデータの全バリデーションメソッドは、`protected`から`public`へ変更になりました。

### ビュー

#### 動的"with"変数名

ビューと変数を共有するため、動的`__call`メソッドが許されています。こうした変数は「キャメル」ケースとして自動的に使えるようになります。

    return view('pool')->withMaximumVotes(100);

`maximumVotes`変数へ、テンプレートから次のようにアクセスできます。

    {{ $maximumVotes }}

#### `@php` Bladeディレクティブ

`@php` Bladeディレクティブは、インラインタグを引数に取らなくなりました。代わりにディレクティブを完全に記述してください。

    @php
        $teamMember = true;
    @endphp

### その他

私達は皆さんへ、`laravel/laravel` [GitHubリポジトリ](https://github.com/laravel/laravel)で変更を確認することを勧めます。多くの変更は必須ではありませんが、皆さんのアプリケーションでは同期を取りたい変更もあることでしょう。そうした変更のいくつかは、このアップグレードガイドで取り扱っていますが、設定ファイルやコメントなどは載っていません。[GitHub差分比較ツール](https://github.com/laravel/laravel/compare/5.4...5.5)で変更を簡単に確認できますので、自分にとってどの変更が重要であるかを洗い出せます。
