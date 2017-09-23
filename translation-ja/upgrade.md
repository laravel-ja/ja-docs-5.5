# アップグレードガイド

- [5.4から5.5.0へのアップグレード](#upgrade-5.5.0)

<a name="upgrade-5.5.0"></a>
## 5.4から5.5.0へのアップグレード

#### アップデートの見積もり時間：１時間

> {note} 私達は、互換性を失う可能性がある変更を全部ドキュメントにしようとしています。しかし、変更点のいくつかは、フレームワークの明確ではない部分で行われているため、一部の変更が実際にアプリケーションに影響を与えてしまう可能性があります。

### PHP

Laravel5.5では、PHP7.0.0以上が必要です。

### 依存パッケージのアップデート

Update your `laravel/framework` dependency to `5.5.*` in your `composer.json` file. In addition, you should update your `phpunit/phpunit` dependency to `~6.0`. Next, add the `filp/whoops` package with version `~2.0` to the `require-dev` section of your `composer.json` file. Finally, in the `scripts` section of your `composer.json` file, add the `package:discover` command to the `post-autoload-dump` event:

    "scripts": {
        ...
        "post-autoload-dump": [
            "Illuminate\Foundation\ComposerScripts::postAutoloadDump",
            "@php artisan package:discover"
        ],
    }

もちろん、アプリケーションで使用しているサードパーティ製パッケージを調べ、Laravel5.5に適切なバージョンを使っていることを忘れず確認してください。

> {tip} `laravel new`を使い、Laravelインストーラによりインストールしている方は、`composer global update`コマンドにより、インストーラパッケージを更新してください。

#### Laravel Dusk

Laravel5.5とヘッドレスChromeでのテストに対し互換性のある、Laravel Dusk `2.0.0`がリリースされました。

#### Pusher

Pusherイベントブロードキャストドライバーは、Pusher SDKのバージョン`~3.0`が必要となりました。

#### Swift Mailer

Laravel5.5では、バージョン`~6.0`のSwift Mailerが必要です。

### Artisan

#### `fire`メソッド

Artisanコマンド中の`fire`メソッドは、`handle`へリネームしてください。

#### `optimize`コマンド

最近のPHPのオペコードキャッシュの向上により、`optimize` Artisanコマンドは必要なくなりました。将来のリリースでLaravelから削除されるため、デプロイスクリプトからこのコマンドを外してください。

### 認証

#### `authorizeResource`コントローラメソッド

`authorizeResource`メソッドへ複数の単語のモデル名を渡したとき、リソースコントローラの振る舞いと合わせるため、ルートセグメントは「スネーク」ケースになりました。

#### `before`ポリシーメソッド

ポリシークラスにチェックしようとするアビリティの名前と一致する、メソッドが含まれていない場合、`before`メソッドは呼び出されなくなりました。

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

即時に実行するジョブをディスパッチし、`handle`メソッドから値を返す場合は、ジョブのディスパッチに`dispatch_now`か、`Bus::dispatch`メソッドを使用してください。

    use Illuminate\Support\Facades\Bus;

    $value = dispatch_now(new Job);

    $value = Bus::dispatch(new Job);

### リクエスト

#### `all`メソッド

`Illuminate\Http\Request`の`all`メソッドをオーバーライドしてる場合は、新しい
`$keys`引数の使用方法を反映してください。

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

### その他

私達は皆さんへ、`laravel/laravel` [GitHubリポジトリ](https://github.com/laravel/laravel)で変更を確認することを勧めます。多くの変更は必須ではありませんが、皆さんのアプリケーションでは同期を取りたい変更もあることでしょう。そうした変更のいくつかは、このアップグレードガイドで取り扱っていますが、設定ファイルやコメントなどは載っていません。[GitHub差分比較ツール](https://github.com/laravel/laravel/compare/5.4...master)で変更を簡単に確認できますので、自分にとってどの変更が重要であるかを洗い出せます。
