# ファイルストレージ

- [イントロダクション](#introduction)
- [設定](#configuration)
    - [パブリックディスク](#the-public-disk)
    - [ローカルディスク](#the-local-driver)
    - [ドライバ要件](#driver-prerequisites)
- [ディスクインスタンス取得](#obtaining-disk-instances)
- [ファイル取得](#retrieving-files)
    - [ファイルURL](#file-urls)
    - [ファイルメタ情報](#file-metadata)
- [ファイル保存](#storing-files)
    - [ファイルアップロード](#file-uploads)
    - [ファイル視認性](#file-visibility)
- [ファイル削除](#deleting-files)
- [ディレクトリ](#directories)
- [カスタムファイルシステム](#custom-filesystems)

<a name="introduction"></a>
## イントロダクション

LaravelはFrank de Jongeさんが作成したありがたいほど素晴らしい、抽象ファイルシステムである[Flysystem](https://github.com/thephpleague/flysystem) PHPパッケージを提供しています。LaravelとFlysystemの統合によりローカルのファイルシステム、Amazon S3、Rackspaceクラウドストレージを操作できる、シンプルなドライバが提供できました。更に素晴らしいことにそれぞれのシステムに対し同じAPIを使用しているため、ストレージをとても簡単に変更できるのです。

<a name="configuration"></a>
## 設定

ファイルシステムの設定ファイルは、`config/filesystems.php`です。このファイルの中に全「ディスク」の設定があります。それぞれのディスクは特定のストレージドライバと保存場所を表します。設定ファイルにはサポート済みのドライバそれぞれの設定例を用意しています。ですからストレージの設定と認証情報を反映するように、設定オプションを簡単に修正できます。

もちろん好きなだけディスクを設定できますし、同じドライバに対し複数のディスクを持つことも可能です。

<a name="the-public-disk"></a>
### パブリックディスク

`public`ディスクは一般公開へのアクセスを許すファイルを意味しています。デフォルトの`public`ディスクは、`local`ドライバを使用しており、`storage/app/public`下に存在しているファイルです。Webからのアクセスを許すには、`public/storage`から`storage/app/public`へシンボリックリンクを張る必要があります。この手法により、公開ファイルを一つのディレクトリへ留め、[Envoyer](https://envoyer.io)のようなリリース時のダウンタイムが起きない開発システムを使っている場合、各リリース間でファイルを簡単に共有できます。

シンボリックリンクを張るには、`storage:link` Artisanコマンドを使用します。

    php artisan storage:link

もちろん、ファイルを保存し、シンボリックリンクが張られたら、`asset`ヘルパを使いファイルへのURLを生成できます。

    echo asset('storage/file.txt');

<a name="the-local-driver"></a>
### ローカルディスク

`local`ドライバを使う場合、設定ファイルで指定した`root`ディレクトリからの相対位置で全ファイル操作が行われることに注意してください。デフォルトでこの値は`storage/app`ディレクトリに設定されています。そのため次のメソッドでファイルは`storage/app/file.txt`として保存されます。

    Storage::disk('local')->put('file.txt', 'Contents');

<a name="driver-prerequisites"></a>
### ドライバ要件

#### Composerパッケージ

S3とRackspaceドライバを使用する場合は、それに適合するパッケージをComposerでインストールする必要があります。

- Amazon S3: `league/flysystem-aws-s3-v3 ~1.0`
- Rackspace: `league/flysystem-rackspace ~1.0`

#### S3ドライバ設定

S3ドライバの設定情報は、`config/filesystems.php`設定ファイルにあります。このファイルはS3ドライバの設定配列のサンプルを含んでいます。この配列を自由に、自分のS3設定と認証情報に合わせて、変更してください。使いやすいように、環境変数はAWSで使用されている命名規約に合わせてあります。

#### FTPドライバ設定

Laravelのファイルシステム統合はFTPでも動作します。しかし、デフォルトでは、フレームワークの`filesystems.php`設定ファイルに、サンプルの設定を含めていません。FTPファイルシステムを設定する必要がある場合は、以下の設定例を利用してください。

    'ftp' => [
        'driver'   => 'ftp',
        'host'     => 'ftp.example.com',
        'username' => 'your-username',
        'password' => 'your-password',

        // 任意のFTP設定
        // 'port'     => 21,
        // 'root'     => '',
        // 'passive'  => true,
        // 'ssl'      => true,
        // 'timeout'  => 30,
    ],

#### Rackspaceドライバ設定

Laravelのファイルシステム統合はRackspaceでも動作します。しかし、デフォルトでは、フレームワークの`filesystems.php`設定ファイルに、サンプルの設定を含めていません。Rackspaceファイルシステムを設定する必要がある場合は、以下の設定例を利用してください。

    'rackspace' => [
        'driver'    => 'rackspace',
        'username'  => 'your-username',
        'key'       => 'your-key',
        'container' => 'your-container',
        'endpoint'  => 'https://identity.api.rackspacecloud.com/v2.0/',
        'region'    => 'IAD',
        'url_type'  => 'publicURL',
    ],

<a name="obtaining-disk-instances"></a>
## ディスクインスタンス取得

`Storage`ファサードを使い設定済みのディスクへの操作ができます。たとえばこのファサードの`put`メソッドを使用し、デフォルトディスクにアバターを保存できます。`Storage`ファサードのメソッド呼び出しで最初に`disk`メソッドを呼び出さない場合、そのメソッドの呼び出しは自動的にデフォルトディスクに対し実行されます。

    use Illuminate\Support\Facades\Storage;

    Storage::put('avatars/1', $fileContents);

複数ディスクを使用する場合、`Storage`ファサードの`disk`メソッドを使用し特定のディスクへアクセスできます。

    Storage::disk('s3')->put('avatars/1', $fileContents);

<a name="retrieving-files"></a>
## ファイル取得

`get`メソッドは指定したファイルの内容を取得するために使用します。ファイル内容がそのまま、メソッドより返されます。ファイルパスはすべて、ディスクに設定した「ルート(root)」からの相対位置で指定することを思い出してください。

    $contents = Storage::get('file.jpg');

`exists`メソッドは指定したディスクにファイルが存在しているかを判定するために使用します。

    $exists = Storage::disk('s3')->exists('file.jpg');

<a name="file-urls"></a>
### ファイルURL

指定したファイルのURLを取得するには、`url`メソッドを使います。`local`ドライバを使用している場合、通常このメソッドは指定したパスの先頭に`/strorage`を付け、そのファイルへの相対パスを返します。`s3`、もしくは`rackspace`ドライバを使用している場合、完全なリモートURLを返します。

    use Illuminate\Support\Facades\Storage;

    $url = Storage::url('file1.jpg');

> {note} `local`ドライバを使用する場合、一般公開するファイルはすべて、`storage/app/public`ディレクトリへ設置する必要があることを忘れないでください。さらに、`public/storage`から`storage/app/public`ディレクトリへ[シンボリックリンクを張る](#the-public-disk)必要もあります。

#### 一時的なURL

`s3`、もしくは`rackspace`ドライバを使用して保存したファイルに対し、指定ファイルの一時的なURLを作成する場合は、`temporaryUrl`メソッドを使用します。このメソッドはパスと、URLの有効期限を指定する`DateTime`インスタンスを引数に取ります。

    $url = Storage::temporaryUrl(
        'file1.jpg', now()->addMinutes(5)
    );

#### ローカルURLホストカスタマイズ

`local`ドライバを使用し、ディスク上に保存されるファイルのホストを事前定義したい場合は、ディスク設定配列へ`url`オプションを追加してください。

    'public' => [
        'driver' => 'local',
        'root' => storage_path('app/public'),
        'url' => env('APP_URL').'/storage',
        'visibility' => 'public',
    ],

<a name="file-metadata"></a>
### ファイルメタ情報

ファイルの読み書きに加え、Laravelはファイル自体に関する情報も提供します。たとえば、`size`メソッドはファイルサイズをバイトで取得するために、使用します。

    use Illuminate\Support\Facades\Storage;

    $size = Storage::size('file1.jpg');

`lastModified`メソッドは最後にファイルが更新された時のUnixタイムスタンプを返します。

    $time = Storage::lastModified('file1.jpg');

<a name="storing-files"></a>
## ファイル保存

`put`メソッドはファイル内容をディスクに保存するために使用します。`put`メソッドにはPHPの`resource`も渡すことができ、Flysystemの裏で動いているストリームサポートを使用します。大きなファイルを取り扱う場合は、ストリームの使用を強く推奨します。

    use Illuminate\Support\Facades\Storage;

    Storage::put('file.jpg', $contents);

    Storage::put('file.jpg', $resource);

#### 自動ストリーミング

指定したファイル位置のファイルのストリーミングを自動的にLaravelに管理させたい場合は、`putFile`か`putFileAs`メソッドを使います。このメソッドは、`Illuminate\Http\File`か`Illuminate\Http\UploadedFile`のインスタンスを引数に取り、希望する場所へファイルを自動的にストリームします。

    use Illuminate\Http\File;
    use Illuminate\Support\Facades\Storage;

    // 自動的に一意のIDがファイル名として指定される
    Storage::putFile('photos', new File('/path/to/photo'));

    // ファイル名を指定する
    Storage::putFileAs('photos', new File('/path/to/photo'), 'photo.jpg');

`putFile`メソッドにはいくつか重要な注意点があります。ファイル名ではなく、ディレクトリ名を指定することに注意してください。`putFile`メソッドは一意のIDを保存ファイル名として生成します。`putFile`メソッドからファイルへのパスが返されますので、生成されたファイル名も含めたパスをデータベースへ保存することができます。

`putFile`と`putFileAs`メソッドはさらに、保存ファイルの「視認性」を指定する引数も受け付けます。これは特にS3などのクラウドディスクにファイルを保存し、一般公開の視認性を設定したい場合に便利です。

    Storage::putFile('photos', new File('/path/to/photo'), 'public');

#### ファイルの先頭／末尾への追加

`prepend`と`append`メソッドで、ファイルの初めと終わりに内容を追加できます。

    Storage::prepend('file.log', 'Prepended Text');

    Storage::append('file.log', 'Appended Text');

#### ファイルコピーと移動

`copy`メソッドは、存在するファイルをディスク上の新しい場所へコピーするために使用します。一方の`move`メソッドはリネームや存在するファイルを新しい場所へ移動するために使用します。

    Storage::copy('old/file1.jpg', 'new/file1.jpg');

    Storage::move('old/file1.jpg', 'new/file1.jpg');

<a name="file-uploads"></a>
### ファイルアップロード

Webアプリケーションで、ファイルを保存する一般的なケースは、ユーザーがプロファイル画像や写真、ドキュメントをアップロードする場合でしょう。アップロードファイルインスタンスに`store`メソッドを使えば、アップロードファイルの保存はLaravelで簡単に行なえます。アップロードファイルを保存したいパスを指定し、`store`メソッドを呼び出すだけです。

    <?php

    namespace App\Http\Controllers;

    use Illuminate\Http\Request;
    use App\Http\Controllers\Controller;

    class UserAvatarController extends Controller
    {
        /**
         * ユーザーのアバターの更新
         *
         * @param  Request  $request
         * @return Response
         */
        public function update(Request $request)
        {
            $path = $request->file('avatar')->store('avatars');

            return $path;
        }
    }

この例には重要な点が含まれています。ファイル名ではなく、ディレクトリ名を指定している点に注目です。デフォルトで`store`メソッドは、一意のIDをファイル名として生成します。`store`メソッドからファイルパスが返されますので、生成されたファイル名を含めた、そのファイルパスをデータベースに保存できます。

上の例と同じ操作を行うため、`Strorage`ファサードの`putFile`メソッドを呼び出すこともできます。

    $path = Storage::putFile('avatars', $request->file('avatar'));

#### ファイル名の指定

保存ファイルが自動的に命名されたくなければ、ファイルパスとファイル名、（任意で）ディスクを引数に持つ`storeAs`メソッドを使ってください。

    $path = $request->file('avatar')->storeAs(
        'avatars', $request->user()->id
    );

もちろん、上記の例と同じファイル操作を行う、`Storage`ファサードの`putFileAs`メソッドも使用できます。

    $path = Storage::putFileAs(
        'avatars', $request->file('avatar'), $request->user()->id
    );

#### ディスクの指定

デフォルトで、`store`メソッドはデフォルトディスクを使用します。他のディスクを指定したい場合は第２引数として、ディスク名を渡してください。

    $path = $request->file('avatar')->store(
        'avatars/'.$request->user()->id, 's3'
    );

<a name="file-visibility"></a>
### ファイル視認性

LaravelのFlysystem統合では、複数のプラットフォームにおけるファイルパーミッションを「視認性」として抽象化しています。ファイルは`public`か`private`のどちらかとして宣言します。ファイルを`public`として宣言するのは、他からのアクセスを全般的に許可することを表明することです。たとえば、S3ドライバ使用時には、`public`ファイルのURLを取得することになります。

`put`メソッドでファイルをセットする時に、視認性を設定できます。

    use Illuminate\Support\Facades\Storage;

    Storage::put('file.jpg', $contents, 'public');

もし、ファイルが既に保存済みであるなら、`getVisibility`と`setVisibility`により、視認性を取得／設定できます。

    $visibility = Storage::getVisibility('file.jpg');

    Storage::setVisibility('file.jpg', 'public')

<a name="deleting-files"></a>
## ファイル削除

`delete`メソッドはディスクから削除するファイルを単独、もしくは配列で受け付けます。

    use Illuminate\Support\Facades\Storage;

    Storage::delete('file.jpg');

    Storage::delete(['file1.jpg', 'file2.jpg']);

必要であれば、削除先のディスクを指定できます。

    use Illuminate\Support\Facades\Storage;

    Storage::disk('s3')->delete('folder_path/file_name.jpg');

<a name="directories"></a>
## ディレクトリ

#### ディレクトリの全ファイル取得

`files`メソッドは指定したディレクトリの全ファイルの配列を返します。指定したディレクトリのサブディレクトリにある全ファイルのリストも取得したい場合は、`allFiles`メソッドを使ってください。

    use Illuminate\Support\Facades\Storage;

    $files = Storage::files($directory);

    $files = Storage::allFiles($directory);

#### ディレクトリの全ディレクトリ取得

`directories`メソッドは指定したディレクトリの全ディレクトリの配列を返します。指定したディレクトリ下の全ディレクトリと、サブディレクトリ下の全ディレクトリも取得したい場合は、`allDirectories`メソッドを使ってください。

    $directories = Storage::directories($directory);

    // 再帰的
    $directories = Storage::allDirectories($directory);

#### ディレクトリ作成

`makeDirectory`メソッドは必要なサブディレクトリを含め、指定したディレクトリを作成します。

    Storage::makeDirectory($directory);

#### ディレクトリ削除

最後の`deleteDirectory`は、ディレクトリと中に含まれている全ファイルを削除するために使用されます。

    Storage::deleteDirectory($directory);

<a name="custom-filesystems"></a>
## カスタムファイルシステム

LaravelのFlysystem統合には、最初から様々な「ドライバ」が含まれています。しかしFlysystemはこれらのドライバに限定されず、他の保存領域システムにも適用できます。皆さんのLaravelアプリケーションに適合した保存システムのカスタムドライバを作成することができます。

カスタムファイルシステムを準備するには、Flysystemアダプタが必要です。プロジェクトへコミュニティによりメンテナンスされているDropboxアダプタを追加してみましょう。

    composer require spatie/flysystem-dropbox

次に、たとえば`DropboxServiceProvider`のような、[サービスプロバイダ](/docs/{{version}}/providers)を用意してください。プロバイダの`boot`メソッドの中で`Storage`ファサードの`extend`メソッドを使い、カスタムドライバを定義できます。

    <?php

    namespace App\Providers;

    use Storage;
    use League\Flysystem\Filesystem;
    use Illuminate\Support\ServiceProvider;
    use Spatie\Dropbox\Client as DropboxClient;
    use Spatie\FlysystemDropbox\DropboxAdapter;

    class DropboxServiceProvider extends ServiceProvider
    {
        /**
         * サービスの初期処理登録後に実行
         *
         * @return void
         */
        public function boot()
        {
            Storage::extend('dropbox', function ($app, $config) {
                $client = new DropboxClient(
                    $config['authorizationToken']
                );

                return new Filesystem(new DropboxAdapter($client));
            });
        }

        /**
         * コンテナで結合の登録
         *
         * @return void
         */
        public function register()
        {
            //
        }
    }

`extend`メソッドの最初の引数はドライバの名前で、２つ目は`$app`と`$config`変数を受け取るクロージャです。このリゾルバークロージャは`League\Flysystem\Filesystem`のインスタンスを返す必要があります。`$config`変数は`config/filesystems.php`で定義したディスクの値を含んでいます。

拡張を登録するサービスプロバイダを作成したら、`config/filesystems.php`設定ファイルで`dropbox`ドライバを使用できます。
