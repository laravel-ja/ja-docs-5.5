# デプロイ

- [イントロダクション](#introduction)
- [サーバ設定](#server-configuration)
    - [Nginx](#nginx)
- [最適化](#optimization)
    - [オートローダー最適化](#autoloader-optimization)
    - [設定ロードの最適化](#optimizing-configuration-loading)
    - [ルートロードの最適化](#optimizing-route-loading)
- [Forgeによるデプロイ](#deploying-with-forge)

<a name="introduction"></a>
## イントロダクション

Laravelアプリケーションをプロダクションとしてデプロイする準備ができたら、アプリケーションをできるだけ確実かつ、効率的な実行を行うには、いくつか重要な手順を行う必要があります。このドキュメントでは、アプリケーションを確実にデプロイするため、重要なポイントを説明します。

<a name="server-configuration"></a>
## サーバ設定

<a name="nginx"></a>
### Nginx

Nginxを実行しているサーバにアプリケーションをデプロイするには、Webサーバの設定として以下の設定ファイルが最初の参考となるでしょう。ほとんどの設定と同様に、このファイルはサーバの設定に合わせてカスタマイズする必要が起きるでしょう。サーバ管理のアシスタントが欲しい場合は、[Laravel Forge](https://forge.laravel.com)のようなサービスの使用を考慮してください。

    server {
        listen 80;
        server_name example.com;
        root /example.com/public;

        add_header X-Frame-Options "SAMEORIGIN";
        add_header X-XSS-Protection "1; mode=block";
        add_header X-Content-Type-Options "nosniff";

        index index.html index.htm index.php;

        charset utf-8;

        location / {
            try_files $uri $uri/ /index.php?$query_string;
        }

        location = /favicon.ico { access_log off; log_not_found off; }
        location = /robots.txt  { access_log off; log_not_found off; }

        error_page 404 /index.php;

        location ~ \.php$ {
            fastcgi_split_path_info ^(.+\.php)(/.+)$;
            fastcgi_pass unix:/var/run/php/php7.1-fpm.sock;
            fastcgi_index index.php;
            include fastcgi_params;
        }

        location ~ /\.(?!well-known).* {
            deny all;
        }
    }

<a name="optimization"></a>
## 最適化

<a name="autoloader-optimization"></a>
### オートローダー最適化

プロダクションへデプロイする場合、Composerのクラスオートローダマップを最適し、Composerが素早く指定されたクラスのファイルを確実に見つけ、ロードできるようにします。

    composer install --optimize-autoloader

> {tip} オートローダを最適化することに加え、プロジェクトのソースコントロールリポジトリへ、`composer.lock`ファイルをいつも確実に含めましょう。`composer.lock`ファイルが存在すると、プロジェクトの依存パッケージのインストールが、より早くなります。

<a name="optimizing-configuration-loading"></a>
### 設定ロードの最適化

アプリケーションをプロダクションへデプロイする場合、デプロイプロセスの中で、確実に`config:cache` Artisanコマンドを実行してください。

    php artisan config:cache

このコマンドは、Laravelの全設定ファイルをキャッシュされる一つのファイルへまとめるため、設定値をロードする場合に、フレームワークがファイルシステムを数多くアクセスする手間を大いに減らします。

<a name="optimizing-route-loading"></a>
### ルートロードの最適化

多くのルートを持つ大きなアプリケーションを構築した場合、デプロイプロセス中に、`route:cache` Artisanコマンドを確実に実行すべきでしょう。

    php artisan route:cache

このコマンドはキャッシュファイルの中の、一つのメソッド呼び出しへ全ルート登録をまとめるため、数百のルートを登録する場合、ルート登録のパフォーマンスを向上します。

> {note} この機能はPHPのシリアライゼーション機能を使用するため、アプリケーションの全ルートをキャッシュするには、コントローラベースのルート定義だけを使用してください。PHPはクロージャをシリアライズできません。

<a name="deploying-with-forge"></a>
## Forgeによるデプロイ

自分のサーバ設定管理に準備不足であったり、堅牢なLaravelアプリケーション実行に必要な数多くのサービス全ての設定について慣れていなければ、[Laravel Forge](https://forge.laravel.com)は素晴らしい代替案です。

Laravel ForgeはDigitalOcean、Linode、AWSなど数多くのインフラプロバイダー上に、サーバを作成できます。それに加え、ForgeはNginx、MySQL、Redis、Memcached、Beanstalkなどのような、堅牢なLaravelアプリケーションを構築するために必要なツールを全てインストールし、管理します。
