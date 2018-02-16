# Laravel Valet

- [イントロダクション](#introduction)
    - [ValetとHomestead](#valet-or-homestead)
- [インストール](#installation)
    - [アップグレード](#upgrading)
- [サイト動作](#serving-sites)
    - ["Park"コマンド](#the-park-command)
    - ["Link"コマンド](#the-link-command)
    - [TLSによる安全なサイト](#securing-sites)
- [サイトの共有](#sharing-sites)
- [カスタムValetドライバ](#custom-valet-drivers)
    - [ローカルドライバ](#local-drivers)
- [その他のValetコマンド](#other-valet-commands)

<a name="introduction"></a>
## イントロダクション

Valet（ベレット：従者）はMacミニマニストのためのLaravel開発環境です。Vagrantも不要、`/etc/hosts`ファイルも不要です。更に、ローカルトンネルを使って、サイトを公開し、シェアすることもできます。*ええ、私達はこういうのも好きなんですよね。*

Laravel Valetはマシン起動時にバックグランドで[Nginx](https://www.nginx.com/)がいつも実行されるように、Macを設定します。そのため、[DnsMasq](https://en.wikipedia.org/wiki/Dnsmasq)を使用し、Valetは`*.test`ドメインへの全リクエストを、ローカルマシンへインストールしたサイトへ向けるようにプロキシ動作します。

言い換えれば、大体7MBのRAMを使う、とても早いLaravelの開発環境です。ValetはVagrantやHomesteadを完全に置き換えるものではありませんが、柔軟な基礎、特にスピード重視であるか、RAMが限られているマシンで動作させるのには、素晴らしい代替になります。

Valetは以下をサポートしていますが、これらに限定されません。

<div class="content-list" markdown="1">
- [Laravel](https://laravel.com)
- [Lumen](https://lumen.laravel.com)
- [Bedrock](https://roots.io/bedrock/)
- [CakePHP 3](https://cakephp.org)
- [Concrete5](http://www.concrete5.org/)
- [Contao](https://contao.org/en/)
- [Craft](https://craftcms.com)
- [Drupal](https://www.drupal.org/)
- [Jigsaw](http://jigsaw.tighten.co)
- [Joomla](https://www.joomla.org/)
- [Katana](https://github.com/themsaid/katana)
- [Kirby](https://getkirby.com/)
- [Magento](https://magento.com/)
- [OctoberCMS](https://octobercms.com/)
- [Sculpin](https://sculpin.io/)
- [Slim](https://www.slimframework.com)
- [Statamic](https://statamic.com)
- Static HTML
- [Symfony](https://symfony.com)
- [WordPress](https://wordpress.org)
- [Zend](https://framework.zend.com)
</div>

独自の[カスタムドライバ](#custom-valet-drivers)でValetを拡張できます。

<a name="valet-or-homestead"></a>
### ValetとHomestead

ご存知のように、ローカルのLaravel開発環境として[Homestead](/docs/{{version}}/homestead)も用意しています。HomesteadとValetは利用者の目的とローカルの開発についてのアプローチが異なります。Homesteadは自動的にNginx設定を行うUbuntuの完全な仮想マシンを提供しています。HomesteadはLinux開発環境の完全な仮想化を行いたい、もしくはWindows／Linux上で動作させたい場合、素晴らしい選択肢になります。

ValetはMac上でのみサポートされ、PHPとデータベースサーバを直接ローカルマシンへインストールする必要があります。[Homebrew](http://brew.sh/)を利用し、`brew install php72`と`brew install mysql`のようなコマンドを実行すれば、簡単にインストールできます。Valetは最低限度のリソースを使い、とても早いローカル開発環境を提供します。そのため、PHPとMySQLだけが必要で、完全な仮想開発環境は必要ない場合にぴったりです。

ValetとHomesteadのどちらを選んでも、Laravelの開発環境に向け設定されており、良い選択になるでしょう。どちらを選ぶかは、自分の好みとチームの必要により決まるでしょう。

<a name="installation"></a>
## インストール

**ValetにはMacオペレーティングシステムと[Homebrew](http://brew.sh/)が必要です。インストールする前に、ApacheやNginxのようなローカルマシンの８０番ポートへバインドするプログラムがないことを確認してください。**

<div class="content-list" markdown="1">
- `brew update`で最新バージョンの[Homebrew](http://brew.sh/)をインストール、もしくはアップデートしてください。
- Homebrewを使い、`brew install homebrew/php/php72`でPHP7.2をインストールしてください。
- `composer global require laravel/valet`でValetをインストールしてください。`~/.composer/vendor/bin`ディレクトリが実行パスに含まれていることを確認してください。
- `valet install`コマンドを実行してください。これによりValetとDnsMasqがインストール／設定され、システム起動時に起動されるValetのデーモンが登録されます。
</div>

Valetがインストールできたら、`ping foobar.test`のようなコマンドで、ターミナルから`*.test`ドメインに対してpingを実行してください。Valetが正しくインストールされていれば、このドメインは`127.0.0.1`へ対応していることが分かるでしょう。

Valetはマシンが起動されると、毎回デーモンを自動的に起動します。Valetが完全にインストールされていれば、`valet start`や`valet install`を再び実行する必要は永久にありません。

#### 他のドメインの使用

デフォルトではValetは`.test` TLDをプロジェクトのドメインとして処理します。他のドメインを使いたい場合、`valet domain tld-name`コマンドを使ってください。

たとえば、`.test`の代わりに`.app`を使用したければ、`valet domain app`と実行します。Valetは`*.app`をプロジェクトのために自動的に使い始めます。

#### データベース

データベースを使いたい場合、コマンドラインで`brew install mysql`を実行し、MySQLを試してください。MySQLがインストールできたら、`brew services start mysql`コマンドを使い、起動します。`127.0.0.1`でデータベースに接続し、ユーザー名は`root`、パスワードは空文字列です。

<a name="upgrading"></a>
### アップグレード

Valetインストールをアップデートするには、ターミナルで`composer global update`コマンドを実行します。アップグレードできたら、`valet install`コマンドを実行し、必要な設定ファイルの追加アップグレードを行うのは、グッドプラクティスです。

#### アップグレード To Valet 2.0

Valet 2.0では、Valetの裏で動作するWebサーバをCaddyからNginxへ移動しました。このバージョン並行するは、以下のコマンドを実行し、既存のCaddyデーモンを停止し、アンインストールしてください。

    valet stop
    valet uninstall

次に、Valetの最新バージョンへアップグレードします。Valetをどのようにインストールしたかにより、通常GitかComposerを使用して行います。ValetをComposerによりインストールしている場合は、最新のメジャーバージョンにアップデートするために、以下のコマンドを実行してください。

    composer global require laravel/valet

真新しいValetのソースコードがダウンロードできたら、`install`コマンドを実行します。

    valet install
    valet restart

アップグレードできたら、`park`と`link`をやり直してください。

<a name="serving-sites"></a>
## サイト動作

Valetがインストールできたら、サイトを動作させる準備ができました。Laravelサイトを動作させるために役立つ、`park`と`link`の２コマンドを用意しています。

<a name="the-park-command"></a>
**`park`コマンド**

<div class="content-list" markdown="1">
- `mkdir ~/Sites`のように、Mac上に新しいディレクトリを作成ししてください。次に`cd ~/Sites`し、`valet park`を実行します。このコマンドはカレントワーキングディレクトリをValetがサイトを探す親パスとして登録します。
- 次に、このディレクトリ内で、新しいLaravelサイトを作成します。`laravel new blog`
- `http://blog.test`をブラウザで開きます。
</div>

**必要なのはこれだけです。** これで"parked"ディレクトリ内で作成されたLaravelプロジェクトは、`http://フォルダ名.test`規約に従い、自動的に動作します。

<a name="the-link-command"></a>
**`link`コマンド**

`link`コマンドは`park`のように親ディレクトリを指定するのではなく、各ディレクトリ中で一つのサイトを動作させるのに便利です。

<div class="content-list" markdown="1">
- ターミナルでプロジェクトのディレクトリへ移動し、`valet link アプリケーション名`を実行します。Valetはカレントワーキングディレクトリから`~/.valet/Sites`内へシンボリックリンクを張ります。
- `link`コマンド実行後、ブラウザで`http://アプリケーション名.test`にアクセスできます。
</div>

リンクされた全ディレクトリをリストするには、`valet links`コマンドを実行してください。シンボリックリンクを外すときは、`valet unlink app-name`を使います。

> {tip} 複数の（サブ）ドメインで同じプロジェクトを動かすために、`valet link`を使用できます。サブドメインや他のドメインをプロジェクトに追加するためには、プロジェクトフォルダから`valet link subdomain.app-name`を実行します。

<a name="securing-sites"></a>
**TLSによる安全なサイト**

Valetはデフォルトで通常のHTTP通信で接続します。しかし、HTTP/2を使った暗号化されたTLSで通信したい場合は、`secure`コマンドを使ってください。たとえば、`laravel.test`ドメインでValetによりサイトが動作している場合、以下のコマンドを実行することで安全な通信を行います。

    valet secure laravel

サイトを「安全でない」状態へ戻し、通常のHTTP通信を使いたい場合は、`unsecure`コマンドです。`secure`コマンドと同様に、セキュアな通信を辞めたいホスト名を指定します。

    valet unsecure laravel

<a name="sharing-sites"></a>
## サイトの共有

Valetはローカルサイトを世界と共有するコマンドも用意しています。Valetがインストールしてあれば、他のソフトウェアは必要ありません。

サイトを共有するには、ターミナルでサイトのディレクトリに移動し、`valet share`コマンドを実行します。公開用のURLはクリップボードにコピーされますので、ブラウザに直接ペーストしてください。これだけです。

サイトの共有を止めるには、`Control + C`を入力し、プロセスをキャンセルしてください。

> {note} `valet secure`コマンドを使用してセキュアにしたサイトの共有は、現在`valet share`はサポートしていません。

<a name="custom-valet-drivers"></a>
## カスタムValetドライバ

Valetでサポートされていない、他のフレームワークやCMSでPHPアプリケーションを実行するには、独自のValet「ドライバ」を書く必要があります。Valetをインストールすると作成される、`~/.valet/Drivers`ディレクトリに`SampleValetDriver.php`ファイルが存在しています。このファイルは、カスタムドライバーをどのように書いたら良いかをデモンストレートするサンプルドライバの実装コードです。ドライバを書くために必要な`serves`、`isStaticFile`、`frontControllerPath`の３メソッドを実装するだけです。

全３メソッドは`$sitePath`、`$siteName`、`$uri`を引数で受け取ります。`$sitePath`は、`/Users/Lisa/Sites/my-project`のように、サイトプロジェクトへのフルパスです。`$siteName`は"ホスト" / "サイト名"記法のドメイン(`my-project`)です。`$uri`はやって来たリクエストのURI(`/foo/bar`)です。

カスタムValetドライバを書き上げたら、`フレームワークValetDriver.php`命名規則をつかい、`~/.valet/Drivers`ディレクトリ下に設置してください。たとえば、WordPress用にカスタムValetドライバを書いたら、ファイル名は`WordPressValetDriver.php`になります。

カスタムValetドライバで実装する各メソッドのサンプルコードを見ていきましょう。

#### `serves`メソッド

`serves`メソッドは、そのドライバがやって来たリクエストを処理すべき場合に、`true`を返してください。それ以外の場合は`false`を返してください。そのためには、メソッドの中で、渡された`$sitePath`の内容が、動作させようとするプロジェクトタイプを含んでいるかを判定します。

では擬似サンプルとして、`WordPressValetDriver`を書いてみましょう。servesメソッドは以下のようになります。

    /**
     * このドライバでリクエストを処理するか決める
     *
     * @param  string  $sitePath
     * @param  string  $siteName
     * @param  string  $uri
     * @return bool
     */
    public function serves($sitePath, $siteName, $uri)
    {
        return is_dir($sitePath.'/wp-admin');
    }

#### `isStaticFile`メソッド

`isStaticFile`はリクエストが画像やスタイルシートのような「静的」なファイルであるかを判定します。ファイルが静的なものであれば、そのファイルが存在するディスク上のフルパスを返します。リクエストが静的ファイルでない場合は、`false`を返します。

    /**
     * リクエストが静的なファイルであるかを判定する
     *
     * @param  string  $sitePath
     * @param  string  $siteName
     * @param  string  $uri
     * @return string|false
     */
    public function isStaticFile($sitePath, $siteName, $uri)
    {
        if (file_exists($staticFilePath = $sitePath.'/public/'.$uri)) {
            return $staticFilePath;
        }

        return false;
    }

> {note} `isStaticFile`メソッドは、リクエストのURIが`/`ではなく、`serves`メソッドで`true`が返された場合のみ呼びだされます。

#### `frontControllerPath`メソッド

`frontControllerPath`メソッドは、アプリケーションの「フロントコントローラ」への絶対パスを返します。通常は"index.php`ファイルか、似たようなファイルでしょう。

    /**
     * アプリケーションのフロントコントローラへの絶対パスの取得
     *
     * @param  string  $sitePath
     * @param  string  $siteName
     * @param  string  $uri
     * @return string
     */
    public function frontControllerPath($sitePath, $siteName, $uri)
    {
        return $sitePath.'/public/index.php';
    }

<a name="local-drivers"></a>
### ローカルドライバ

一つのアプリケーションに対して、Valetのカスタムドライバを定義する場合は、アプリケーションのルートディレクトリに`LocalValetDriver.php`を作成してください。カスタムドライバは、ベースの`ValetDriver`クラスか、`LaravelValetDriver`のような、既存のアプリケーション専用のドライバを拡張します。

    class LocalValetDriver extends LaravelValetDriver
    {
        /**
         * リクエストに対し、このドライバを動作させるかを決める
         *
         * @param  string  $sitePath
         * @param  string  $siteName
         * @param  string  $uri
         * @return bool
         */
        public function serves($sitePath, $siteName, $uri)
        {
            return true;
        }

        /**
         * アプリケーションのフロントコントローラに対する完全な解決済みパスを取得する
         *
         * @param  string  $sitePath
         * @param  string  $siteName
         * @param  string  $uri
         * @return string
         */
        public function frontControllerPath($sitePath, $siteName, $uri)
        {
            return $sitePath.'/public_html/index.php';
        }
    }

<a name="other-valet-commands"></a>
## その他のValetコマンド

コマンド |  説明
-----------|--------------------
`valet forget` | "park"された（サイト検索の親ディレクトリとして登録されたJ)ディレクトリでこのコマンドを実行し、サイト検索対象のディレクトリリストから外します。
`valet paths` | "park"されたすべてのパスを表示します。
`valet restart` | Valetデーモンをリスタートします。
`valet start` | Valetデーモンをスタートします。
`valet stop` | Valetデーモンを停止します。
`valet uninstall` | Valetデーモンを完全にアンインストールします。
