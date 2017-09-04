# Laravel Homestead

- [イントロダクション](#introduction)
- [インストールと設定](#installation-and-setup)
    - [最初の段階](#first-steps)
    - [Homestead設定](#configuring-homestead)
    - [Vagrant Boxの実行](#launching-the-vagrant-box)
    - [プロジェクトごとのインストール](#per-project-installation)
    - [MariaDBのインストール](#installing-mariadb)
- [使用方法](#daily-usage)
    - [Homesteadへのグローバルアクセス](#accessing-homestead-globally)
    - [SSH接続](#connecting-via-ssh)
    - [データベース接続](#connecting-to-databases)
    - [サイトの追加](#adding-additional-sites)
    - [Cronスケジュール設定](#configuring-cron-schedules)
    - [ポート](#ports)
    - [環境の共有](#sharing-your-environment)
    - [複数のPHPバージョン](#multiple-php-versions)
- [ネットワークインターフェイス](#network-interfaces)
- [Homesteadの更新](#updating-homestead)
- [旧バージョン](#old-versions)
- [プロパイダ固有の設定](#provider-specific-settings)
    - [VirtualBox](#provider-specific-virtualbox)

<a name="introduction"></a>
## イントロダクション

ローカル開発環境を含め、PHP開発全体を愉快なものにしようとLaravelは努力しています。[Vagrant](https://vagrantup.com)は、仮想マシンの管理と事前設定を行う、簡単でエレガントな手段を提供しています。

Laravel Homestead（入植農地、「ホームステード」）はパッケージを事前に済ませた、Laravel公式の"box"で、PHPやWebサーバ、その他のサーバソフトウェアをローカルマシンにインストールする必要なく、素晴らしい開発環境を準備できます。オペレーティングシステムでごちゃごちゃになる心配はもうありません！　Vagrant boxは完全に使い捨てできます。何かの調子が悪くなれば壊して、数分のうちにそのboxを再生成できます！

HomesteadはWindowsやMac、Linuxシステム上で実行でき、Nginx WebサーバとPHP7.1、MySQL、PostgreSQL、Redis、Memcached、Node、他にも素晴らしいLaravelプリケーションを開発するために必要となるものすべてを含んでいます。

> {note} Windowsを使用している場合は、ハードウェア仮想化(VT-x)を有効にする必要があります。通常、BIOSにより有効にできます。UEFI system上のHyper-Vを使用している場合は、VT-xへアクセスするため、さらにHyper-Vを無効にする必要があります。

<a name="included-software"></a>
### 含まれるソフトウェア

- Ubuntu 16.04
- Git
- PHP 7.1
- Nginx
- MySQL
- MariaDB
- Sqlite3
- PostgreSQL
- Composer
- Node (Yarn、Bower、Bower、Grunt、Gulpを含む)
- Redis
- Memcached
- Beanstalkd
- Mailhog
- ngrok

<a name="installation-and-setup"></a>
## インストールと設定

<a name="first-steps"></a>
### 最初の段階

Homestead環境を起動する前に、[VirtualBox 5.1](https://www.virtualbox.org/wiki/Downloads)と[VMWare](https://www.vmware.com)、もしくは[Parallels](http://www.parallels.com/products/desktop/)、それと[Vagrant](https://www.vagrantup.com/downloads.html)をインストールする必要があります。全ソフトウェア共に簡単に使用できるビジュアルインストーラが、人気のあるオペレーティングシステム全部に用意されています。

VMwareプロバイダを使用するには、VMware Fusion/Workstationと[VMware Vagrantプラグイン](https://www.vagrantup.com/vmware)を購入する必要があります。無料ではありませんが、VMwareが提供する共有フォルダーは最初からよりスピーディーです。

Parallelsプロバイダを使用するには、[Parallels Vagrantプラグイン](https://github.com/Parallels/vagrant-parallels)をインストールする必要があります。これは無料です。

#### Homestead Vagrant Boxのインストール

VirtualBox/VMwareとVagrantをインストールし終えたら、`laravel/homestead` boxをVagrantへ追加するため次のコマンドを端末で実行する必要があります。boxをダウンロードし終えるまで、接続速度にもよりますが数分かかるでしょう。

    vagrant box add laravel/homestead

このコマンドが失敗する場合、Vagrantを更新する必要があります。

#### Homesteadのインストール

リポジトリーをクローンするだけでHomesteadをインストールできます。自分の「ホーム」ディレクトリの中の`Homestead`フォルダーへリポジトリーをクローンするのことは、自分のLaravel（とPHP）の全プロジェクトをホストしておくHomestead Boxを用意するのだと考えてください。

    cd ~

    git clone https://github.com/laravel/homestead.git Homestead

`master`ブランチは常に安定しているわけではないため、バージョンタグがついたHomesteadをチェックアウトすべきでしょう。最新の安定バージョンは、[GitHubのリリースページ](https://github.com/laravel/homestead/releases)で見つかります。

    cd Homestead

    // クローンしたいリリースバージョン
    git checkout v6.1.0

Homesteadリポジトリをクローンしたら、`Homestead.yaml`設定ファイルを生成するために、`bash init.sh`コマンドをHomesteadディレクトリで実行します。

    // Mac / Linux...
    bash init.sh

    // Windows...
    init.bat

<a name="configuring-homestead"></a>
### Homestead設定

#### プロバイダの設定

`Homestead.yaml`ファイル中の`provider`キーは、Vagrantのプロバイダとして、`virtualbox`、`vmware_fusion`、`vmware_workstation`、`parallels`のどれを使用するかを指定します。使用するプロバイダの値を指定してください。

    provider: virtualbox

#### 共有フォルダーの設定

`Homestead.yaml`ファイルの`folders`プロパティーには、Homestead環境と共有したい全フォルダーがリストされています。これらのフォルダーの中のファイルが変更されると、ローカルマシンとHomestead環境との間で同期されます。必要なだけ共有フォルダーを設定してください！

    folders:
        - map: ~/Code
          to: /home/vagrant/Code

[NFS](https://www.vagrantup.com/v2/synced-folders/nfs.html)を有効にするには、同期するフォルダーにフラグを指定するだけです。

    folders:
        - map: ~/Code
          to: /home/vagrant/Code
          type: "nfs"

> {note} NFSを使用する場合は、[vagrant-bindfs](https://github.com/gael-ian/vagrant-bindfs)プラグインのインストールを考慮してください。このプラグインは、Homestead下のファイルとディレクトリのユーザー／グループパーミッションを正しく維持します。

さらに、Vagrantの[同期フォルダ](https://www.vagrantup.com/docs/synced-folders/basic_usage.html)でサポートされている任意のオプションを、`options`キーの下に列挙して渡すことができます。

    folders:
        - map: ~/Code
          to: /home/vagrant/Code
          type: "rsync"
          options:
              rsync__args: ["--verbose", "--archive", "--delete", "-zz"]
              rsync__exclude: ["node_modules"]


#### Nginxサイトの設定

Nginxには詳しくない？　問題ありません。`sites`プロパティーでHomestead環境上のフォルダーと「ドメイン」を簡単にマップできます。サイト設定のサンプルは、`Homestead.yaml`ファイルに含まれています。これも必要に応じ、Homestead環境へサイトを好きなだけ追加してください。便利に使えるように、Homesteadは皆さんが作業する全てのLaravelプロジェクトの仮想環境を提供します。

    sites:
        - map: homestead.app
          to: /home/vagrant/Code/Laravel/public

`sites`プロパティをHomestead boxのプロビジョニング後に変更した場合、仮想マシンのNginx設定を更新するため、`vagrant reload --provision`を再実行する必要があります。

#### hostsファイル

Nginxサイトの"domains"に追加したサイトをあなたのコンピューターの`hosts`ファイルにも追加してください。`hosts`ファイルはローカルドメインへのリクエストをHomestead環境へ転送してくれます。MacとLinuxでは、`/etc/hosts`にこのファイルがあります。Windows環境では、`C:\Windows\System32\drivers\etc\hosts`です。次の行のように追加してください。

    192.168.10.10  homestead.app

設定するIPアドレスには`Homestead.yaml`ファイルの中の値を確実に指定してください。ドメインを`hosts`ファイルへ追加したら、Webブラウザーでサイトにアクセスできます。

    http://homestead.app

<a name="launching-the-vagrant-box"></a>
### Vagrant Boxの実行

`Homestead.yaml`のリンクを編集終えたら、Homesteadディレクトリで`vagrant up`コマンドを実行してください。Vagrantは仮想マシンを起動し、共有フォルダーとNginxサイトを自動的に設定します。

仮想マシンを破壊するには、`vagrant destroy --force`コマンドを使用します。

<a name="per-project-installation"></a>
### プロジェクトごとにインストール

Homesteadをグローバルにインストールし、全プロジェクトで同じHomestead Boxを共有する代わりに、Homesteadインスタンスを管理下のプロジェクトごとに設定することもできます。プロジェクトごとにHomesteadをインストールする利点は、`Vagrantfile`をプロジェクトに用意すれば、プロジェクトに参加している他の人達も、`vagrant up`だけで仕事にとりかかれることです。

Homesteadをプロジェクトに直接インストールするには、Composerを使います。

    composer require laravel/homestead --dev

Homesteadがインストールできたら、`Vagrantfile`と`Homestead.yaml`ファイルをプロジェクトルートへ生成するために`make`コマンドを使ってください。`make`コマンドは`Homestead.yaml`ファイルの`sites`と`folders`ディレクティブを自動的に設定します。

Mac / Linux:

    php vendor/bin/homestead make

Windows:

    vendor\bin\homestead make

次に`vagrant up`コマンドを端末で実行し、ブラウザで`http://homestead.app`のプロジェクトへアクセスしてください。`/etc/hosts`ファイルに`homestead.app`か選んだドメインのエントリーを追加する必要はあることを覚えておきましょう。

<a name="installing-mariadb"></a>
### MariaDBのインストール

MySQLの代わりにMariaDBを使用したい場合は、`mariadb`オプションを`Homestead.yaml`ファイルへ追加してください。このオプションはMySQLを削除し、MariaDBをインストールします。MariaDBはMySQLとそのまま置き換えられる代用ソフトウェアですので、`mysql`データベースドライバをそのままアプリケーションで使用できます。

    box: laravel/homestead
    ip: "192.168.20.20"
    memory: 2048
    cpus: 4
    provider: virtualbox
    mariadb: true

<a name="daily-usage"></a>
## 使用方法

<a name="accessing-homestead-globally"></a>
### Homesteadへグローバルにアクセスする

MacとLinuxシステムでは、Bashプロファイルへ簡単なBash関数を追加すれば実現できます。Windowsでは、`PATH`に「バッチ」ファイルを追加すれば、行えます。以下のスクリプトはシステムのどこからでも、どんなVagrantコマンドでも実行できるようにし、自動的にHomesteadをインストール済みのディレクトリで実行します。

#### Mac / Linux

    function homestead() {
        ( cd ~/Homestead && vagrant $* )
    }

エイリアス中の`~/Homestead`パスを実際にHomesteadをインストール場所を示すように調整してください。関数がインストールできたら、システムのどこからでも`homestead up`や`homestead ssh`のように実行できます。

#### Windows

以下の内容の`homestead.bat`バッチファイルを、マシン上に作成してください。

    @echo off

    set cwd=%cd%
    set homesteadVagrant=C:\Homestead

    cd /d %homesteadVagrant% && vagrant %*
    cd /d %cwd%

    set cwd=
    set homesteadVagrant=

スクリプト例中の`C:\Homestead`パスは、実際にHomesteadをインストールした場所を指すように調整してください。ファイルを作成したら、`PATH`へファイルの場所を追加します。これで`homestead up`や`homestead ssh`のようなコマンドをシステムのどこからでも実行できます。

<a name="connecting-via-ssh"></a>
### SSH接続

Homesteadディレクトリで`vagrant ssh`端末コマンドを実行すれば、仮想マシンにSSHで接続できます。

しかし、Homesteadマシンには頻繁にSSHでアクセスする必要があると思いますから、ホストマシンから素早くHomestead boxへSSH接続できるように、上記の「関数」を追加することを検討してください。

<a name="connecting-to-databases"></a>
### データベースへの接続

`homestead`のデータベースは、初めからMySQLとPostgreSQLの両方を設定できます。更に便利なように、Laravelの`.env`ファイルは、Laravelでこれらのデータベースが最初から利用できるように設定されています。

ホストマシンのデータベースクライアントから、MySQLかPostgreSQLデータベースへ接続するには、`127.0.0.1`のポート`33060`(MySQL)か、ポート`54320`(PostgreSQL)へ接続する必要があります。ユーザー名は`homestead`、パスワードは`secret`です。

> {note} ホストマシンからデータベースへ接続するには、標準的ではないポートだけを使用してください。Laravelのデータベース設定ファイル中では、デフォルトの3306と5432ポートを使用することができます。Laravelは仮想マシンの内部で動作しているからです。

<a name="adding-additional-sites"></a>
### サイトの追加

Homestead環境をプロビジョニングし、実働した後に、LaravelアプリケーションをNginxサイトへ追加したいこともあるでしょう。希望するだけのLaravelアプリケーションを一つのHomestead環境上で実行することができます。新しいサイトを追加するには、`Homestead.yaml`ファイルへ追加するだけです。

    sites:
        - map: homestead.app
          to: /home/vagrant/Code/Laravel/public
        - map: another.app
          to: /home/vagrant/Code/another/public

Vagrantが"hosts"ファイルを自動的に管理しない場合は、新しいサイトを追加する必要があります。

    192.168.10.10  homestead.app
    192.168.10.10  another.app

サイトを追加したら、`vagrant reload --provision`コマンドをHomesteadディレクトリで実行します。

<a name="site-types"></a>
#### サイトタイプ

Laravelベースではないプロジェクトも簡単に実行できるようにするため、Homesteadは様々なタイプのサイトをサポートしています。たとえば、`symfony2`サイトタイプを使えば、HomesteadにSymfonyアプリケーションを簡単に追加できます。

    sites:
        - map: symfony2.app
          to: /home/vagrant/Code/Symfony/web
          type: symfony2

指定できるサイトタイプは`apache`、`laravel`（デフォルト）、`proxy`、`silverstripe`、`statamic`、`symfony2`、`symfony4`です。

<a name="site-parameters"></a>
#### サイトパラメータ

`params`サイトディレクティブを使用し、Nginxの`fastcgi_param`値を追加できます。例として、値に`BAR`を持つ`FOO`パラメータを追加してみましょう。

    sites:
        - map: homestead.app
          to: /home/vagrant/Code/Laravel/public
          params:
              - key: FOO
                value: BAR

<a name="configuring-cron-schedules"></a>
### Cronスケジュール設定

`schedule:run` Artisanコマンドだけを毎分実行することにより、[Cronジョブのスケジュール](/docs/{{version}}/scheduling)を簡単に行う方法をLaravelは提供しています。`schedule:run`コマンドは`App\Console\Kernel`クラスの定義を調べ、どのジョブを実行すべきかを決定します。

Homesteadサイトで`schedule:run`コマンドを実行したい場合は、サイトを定義するときに`schedule`オプションを`true`に設定してください。

    sites:
        - map: homestead.app
          to: /home/vagrant/Code/Laravel/public
          schedule: true

こうしたサイト用のCronジョブは、仮想マシンの`/etc/cron.d`フォルダーの中に定義されます。

<a name="ports"></a>
### ポート

以下のポートが、Homestead環境へポートフォワードされています。

- **SSH:** 2222 &rarr;  フォワード先 22
- **HTTP:** 8000 &rarr; フォワード先 80
- **HTTPS:** 44300 &rarr; フォワード先 443
- **MySQL:** 33060 &rarr; フォワード先 3306
- **PostgreSQL:** 54320 &rarr; Forwards To 5432
- **Mailhog:** 8025 &rarr; フォワード先 8025

#### 追加のフォワードポート

ご希望ならば追加のポートをVagrant Boxへフォワードすることもできます。プロトコルを指定することもできます。

    ports:
        - send: 50000
          to: 5000
        - send: 7777
          to: 777
          protocol: udp

<a name="sharing-your-environment"></a>
### 環境の共有

共同作業者やクライアントと、現在作業中の内容を共有したい場合もあるでしょう。Vagrantには、`vagrant share`により、これをサポートする方法が組み込み済みです。しかし、この方法は`Homestead.yaml`ファイルに複数サイトを設定している場合には動作しません。

この問題を解決するため、Homesteadは独自の`share`コマンドを持っています。使用を開始するには、`vagrant ssh`によりHomesteadマシンとSSH接続し、`share homestead.app`を実行してください。これにより、`Homestead.yaml`設定ファイルの`homestead.app`サイトが共有されます。もちろん、`homestead.app`の代わりに他の設定済みサイトを指定できます。

    share homestead.app

コマンド実行後、ログと共有サイトへアクセスするURLを含んだ、Ngrokスクリーンが現れます。カスタムリージョン、サブドメイン、その他のNgrok実行オプションをカスタマイズしたい場合は、`share`コマンドへ追加してください。

    share homestead.app -region=eu -subdomain=laravel

> {note} Vagrantは本質的に安全なものではなく、`share`コマンドによりインターネット上に自分の仮想マシンを晒すことになることを覚えておいてください。

<a name="multiple-php-versions"></a>
### 複数のPHPバージョン

> {note} この機能は、Nginx使用時のみ利用できます。

Homestead6から、同一仮想マシン上での複数PHPバージョンをサポートを開始しました。`Homestead.yaml`ファイルで、特定のサイトでどのバージョンのPHPを使用するのかを指定できます。利用できるPHPバージョンは、"5.6"、"7.0"、"7.1"です。

    sites:
        - map: homestead.app
          to: /home/vagrant/Code/Laravel/public
          php: "5.6"

さらに、コマンドラインではサポート済みPHPバージョンをすべて利用できます。

    php5.6 artisan list
    php7.0 artisan list
    php7.1 artisan list

<a name="network-interfaces"></a>
## ネットワークインターフェイス

`Homestead.yaml`ファイルの`network`プロパティは、Homestead環境のネットワークインターフェイスを設定します。多くのインターフェイスを必要に応じ設定可能です。

    networks:
        - type: "private_network"
          ip: "192.168.10.20"

[ブリッジ](https://www.vagrantup.com/docs/networking/public_network.html)インターフェイスを有効にするには、`bridge`項目を設定し、ネットワークタイプを`public_network`へ変更します。

    networks:
        - type: "public_network"
          ip: "192.168.10.20"
          bridge: "en1: Wi-Fi (AirPort)"

[DHCP](https://www.vagrantup.com/docs/networking/public_network.html)を有効にするには、設定から`ip`オプションを取り除いてください。

    networks:
        - type: "public_network"
          bridge: "en1: Wi-Fi (AirPort)"

<a name="updating-homestead"></a>
## Homesteadの更新

２つの簡単な手順で、Homesteadをアップデートできます。最初に`vagrant box update`コマンドを使い、Vagrant boxを更新してください。

    vagrant box update

次に、Homesteadのソースコードを更新する必要があります。リポジトリをクローンしている場合は、リポジトリをクローンしたもともとの場所で、`git pull origin master`を単に実行するするだけです。

プロジェクトの`composer.json`ファイルにより、Homesteadをインストールしている場合は、`composer.json`ファイルに`"laravel/homestead": "^6"`が含まれていることを確認し、依存パッケージを更新してください。

    composer update

<a name="old-versions"></a>
## 旧バージョン

> {tip} 古いバージョンのPHPを使用したい場合は、Homesteadの古いバージョンを試して見る前に、<a href="#multiple-php-versions">複数PHPバージョン</a>のドキュメントを確認してください。

`Homestead.yaml`ファイルに以下の行を付け加えることで、Homesteadが使用するboxのバージョンを簡単にオーバーライドできます。

    version: 0.6.0

一例：

    box: laravel/homestead
    version: 0.6.0
    ip: "192.168.20.20"
    memory: 2048
    cpus: 4
    provider: virtualbox

古いバージョンのHomestead boxを使用する場合は、互換性のあるバージョンのHomesteadソースコードへ合わせる必要があります。下図はサポートされているboxのバージョンを表し、使用すべきHomesteadソースコードのバージョンとPHPバージョンです。

|   | Homesteadバージョン | Boxバージョン |
|---|---|---|
| PHP 7.0 | 3.1.0 | 0.6.0 |
| PHP 7.1 | 4.0.0 | 1.0.0 |
| PHP 7.1 | 5.0.0 | 2.0.0 |
| PHP 7.1 | 6.0.0 | 3.0.0 |

<a name="provider-specific-settings"></a>
## プロパイダ固有の設定

<a name="provider-specific-virtualbox"></a>
### VirtualBox

デフォルトのHomestead設定は、`natdnshostresolver`設定を`on`にしています。これにより、HomesteadはホストのオペレーティングシステムのDNS設定を利用します。この動作をオーバーライドしたい場合は、`Homestead.yaml`へ以下の行を追加してください。

    provider: virtualbox
    natdnshostresolver: off
