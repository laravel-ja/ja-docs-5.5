# 貢献ガイド

- [バグレポート](#bug-reports)
- [コア開発の議論](#core-development-discussion)
- [どのブランチ？](#which-branch)
- [セキュリティ脆弱性](#security-vulnerabilities)
- [コーディングスタイル](#coding-style)
    - [PHPDoc](#phpdoc)
    - [StyleCI](#styleci)

<a name="bug-reports"></a>
## バグレポート

より積極的に援助して頂きたいため、Laravelではただのバグレポートでなく、プルリクエストしてくれることを強く推奨しています。「バグレポート」は、失敗するテストを含めた、プルリクエストの形式で送ってください。

しかし、バグレポートを提出する場合には、その問題をタイトルに含め、明確に内容を記述してください。できる限り関連する情報や、その問題をデモするコードも含めてください。バグレポートの目的はあなた自身、そして他の人でも、簡単にバグが再現でき、修正されるようにすることです。

バグレポートは同じ問題にあった他の人達と、解決するために協力できる望みを生み出すのだということを覚えておいてください。バグレポートにより、自動的に何かが起きたり、誰かがすぐ修正したりすることを期待しないでください。バグレポートの提出は、あなた自身と他の人が、問題を解決する道筋を開始するきっかけなのです。

LaravelのソースコードはGitHubで管理され、各Laravelプロジェクトのリポジトリが存在しています。

<div class="content-list" markdown="1">
- [Laravel Application](https://github.com/laravel/laravel)
- [Laravel Art](https://github.com/laravel/art)
- [Laravel Documentation](https://github.com/laravel/docs)
- [Laravel Cashier](https://github.com/laravel/cashier)
- [Laravel Cashier for Braintree](https://github.com/laravel/cashier-braintree)
- [Laravel Envoy](https://github.com/laravel/envoy)
- [Laravel Framework](https://github.com/laravel/framework)
- [Laravel Homestead](https://github.com/laravel/homestead)
- [Laravel Homestead Build Scripts](https://github.com/laravel/settler)
- [Laravel Horizon](https://github.com/laravel/horizon)
- [Laravel Passport](https://github.com/laravel/passport)
- [Laravel Scout](https://github.com/laravel/scout)
- [Laravel Socialite](https://github.com/laravel/socialite)
- [Laravel Website](https://github.com/laravel/laravel.com)
</div>

<a name="core-development-discussion"></a>
## コア開発の議論

新機能や、現存のLaravelの振る舞いについて改善を提言したい場合は、Laravel内部の[issueボード](https://github.com/laravel/internals/issues)へおねがいします。新機能を提言する場合は自発的に、それを完動させるのに必要な、コードを最低限でも実装してください。

バグ、新機能、既存機能の実装についてのざっくばらんな議論は、Slackの[LaraChat](https://larachat.co)チームにある`#internals`チャンネルで行っています。LaravelのメンテナーであるTaylor Otwellは、通常ウイークエンドの午前８時から５時まで（America/Chicago標準時、UTC-6:00）接続しています。他の時間帯では、時々接続しています。

<a name="which-branch"></a>
## どのブランチ？

**全ての**バグフィックスは、最新の安定ブランチ,もしくは現在のLTSブランチ(5.5)へ送ってください。次のリリースの中にだけ存在している機能に対する修正でない限り、決してバグフィックスを`master`ブランチに送っては**いけません**。

現在のLaravelリリースと**完全な後方コンパティビリティ**を持っている**マイナー**な機能は、最新の安定ブランチへ送ってください。

次のLaravelリリースに含めるべき、**メジャー**な新機能は、常に`master`ブランチへ送ってください。

もし、あなたの新機能がメジャーなのか、マイナーなのかはっきりしなければ、[LaraChat](https://larachat.co) Slackチームの`#internals`チャンネルでTaylor Otwellに尋ねてください。

<a name="security-vulnerabilities"></a>
## セキュリティ脆弱性

Laravelにセキュリティー脆弱性を見つけたときは、メールで[Taylor Otwell(taylorotwell@laravel.com)](mailto:taylor@laravel.com)に連絡してください。全セキュリティー脆弱性は、速やかに対応されるでしょう。

<a name="coding-style"></a>
## コーディングスタイル

Laravelは[PSR-2](https://github.com/php-fig/fig-standards/blob/master/accepted/PSR-2-coding-style-guide.md)コーディング規約と[PSR-4](https://github.com/php-fig/fig-standards/blob/master/accepted/PSR-4-autoloader.md)オートローディング規約に準拠しています。

<a name="phpdoc"></a>
### PHPDoc

次に正しいLaravelのドキュメントブロックの例を示します。`@param`属性に続け２スペース、引数タイプ、２スペース、最後に変数名となっていることに注意してください。

    /**
     * Register a binding with the container.
     *
     * @param  string|array  $abstract
     * @param  \Closure|string|null  $concrete
     * @param  bool  $shared
     * @return void
     */
    public function bind($abstract, $concrete = null, $shared = false)
    {
        //
    }

<a name="styleci"></a>
### StyleCI

コードのスタイルが完璧でなくても心配ありません。プルリクエストがマージされたあとで、[StyleCI](https://styleci.io/)が自動的にスタイルを修正し、Laravelリポジトリへマージします。これにより、コードスタイルではなく、貢献の内容へ集中することができます。
