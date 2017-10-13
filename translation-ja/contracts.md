# 契約

- [イントロダクション](#introduction)
    - [契約 vs. ファサード](#contracts-vs-facades)
- [いつ契約を使うか](#when-to-use-contracts)
    - [疎結合](#loose-coupling)
    - [単純さ](#simplicity)
- [契約使用法](#how-to-use-contracts)
- [契約リファレンス](#contract-reference)

<a name="introduction"></a>
## イントロダクション

Laravelの契約とはインターフェイスのことで、フレームワークにより提供されているコアサービスを定義したものです。例えば`Illuminate\Contracts\Queue\Queue`契約はジョブをキューするために必要なメソッドを定義しており、一方`Illuminate\Contracts\Mail\Mailer`契約はメール送信に必要なメソッドを定義しています。

Each contract has a corresponding implementation provided by the framework. For example, Laravel provides a queue implementation with a variety of drivers, and a mailer implementation that is powered by [SwiftMailer](https://swiftmailer.symfony.com/).

Laravelの全契約は[GitHubのリポジトリ](https://github.com/illuminate/contracts)で参照できます。これは全契約を素早く参照する方法であり、同時にパッケージ開発者は個別に独立したパッケージを利用する際、参考にできるでしょう。

<a name="contracts-vs-facades"></a>
### 契約 Vs. ファサード

Laravelの[ファサード](/docs/{{version}}/facades)とヘルパ関数は、タイプヒントやサービスコンテナを契約の解決に使用する必要なく、Laravelの機能を活用できる、シンプルな手法を提供しています。ほとんどの場合、各ファサードと同等の契約が用意されています。

クラスのコンストラクタで、タイプヒントを指定する必要がないファサードと異なり、契約はクラスで必要な依存を明確に定義付けることができます。ある開発者はこの方法で、明確に依存を定義することを好みますが、一方で他の開発者はファサードの利便性を楽しんでいます。

> {tip} 大抵のアプリケーションでは、好みがファサードでも、契約でも問題ないでしょう。しかし、パッケージを作成する場合は、パッケージ開発のテストのしやすさという点で、契約を使うことをしっかりと考えるべきでしょう。

<a name="when-to-use-contracts"></a>
## いつ契約を使うか

他でも説明しているように、契約とファサードのどちらを使うかは、個人や開発チームの好みに行き着きます。契約とファサードのどちらでも、堅牢でよくテストされたLaravelアプリケーションを作成できます。クラスの責務に焦点を当てていれば、契約とファサード間の実践上の違いはとても小さいことに気がつくでしょう。

しかし、契約に関して皆さん、まだ多くの疑問をお持ちでしょう。たとえば、なぜすべてにインターフェイスを使うのでしょう？　インターフェイスはより複雑ではないのでしょうか？　インターフェイスを使用する理由は、次の点に集約されます。「疎結合と単純さ」

<a name="loose-coupling"></a>
### 疎結合

最初にキャッシュの実装とがっちり結合したコードをレビューしてみましょう。次のコードをご覧ください。

    <?php

    namespace App\Orders;

    class Repository
    {
        /**
         * キャッシュインスタンス
         */
        protected $cache;

        /**
         * 新しいリポジトリインスタンスの生成
         *
         * @param  \SomePackage\Cache\Memcached  $cache
         * @return void
         */
        public function __construct(\SomePackage\Cache\Memcached $cache)
        {
            $this->cache = $cache;
        }

        /**
         * 注文をIDから取得
         *
         * @param  int  $id
         * @return Order
         */
        public function find($id)
        {
            if ($this->cache->has($id))    {
                //
            }
        }
    }

このクラスのコードは、使用しているキャッシュの実装ときつく結合しています。つまりパッケージベンダーの具象キャッシュクラスに依存しているために、結合が強くなっています。パッケージのAPIが変更されたら、同時にこのコードも変更しなくてはなりません。

キャッシュの裏で動作している技術(Memcached)を別のもの(Redis)へ置き換えたくなれば、リポジトリを修正する必要があるというのは起こり得ます。リポジトリは誰がデータを提供しているかとか、どのように提供しているかという知識を沢山持っていてはいけません。

**このようなアプローチを取る代わりに、ベンダーと関連がないシンプルなインターフェイスへ依存するコードにより向上できます。**

    <?php

    namespace App\Orders;

    use Illuminate\Contracts\Cache\Repository as Cache;

    class Repository
    {
        /**
         * キャッシュインスタンス
         */
        protected $cache;

        /**
         * 新しいリポジトリインスタンスの生成
         *
         * @param  Cache  $cache
         * @return void
         */
        public function __construct(Cache $cache)
        {
            $this->cache = $cache;
        }
    }

これでコードは特定のベンダー、しかもLaravelにさえ依存しなくなりました。契約パッケージは実装も依存も含んでいないため、与えられた契約の異なった実装を簡単に記述できます。キャッシュを使用するコードを変更することなく、キャッシュ実装を置き換えることができるようになりました。

<a name="simplicity"></a>
### 単純さ

Laravelのサービスは全てシンプルなインターフェイスの中で適切に定義されているので、サービスが提供する機能も簡単に定義できています。 **契約はフレームワークの機能の簡単なドキュメントとして使えます。**

それに加え、シンプルなインターフェイスに基づけばあなたのコードは簡単に理解でき、メンテナンスできるようにもなります。大きくて複雑なクラスの中でどのメソッドが使用可能かを探し求めるよりも、シンプルでクリーンなインターフェイスを参照できます。

<a name="how-to-use-contracts"></a>
## 契約使用法

では、契約の実装はどうやって入手するのでしょうか？とてもシンプルです。

Laravelでは多くのタイプのクラスが[サービスコンテナ](/docs/{{version}}/container)を利用して依存解決されています。コントローラを始め、イベントリスナ、フィルター、キュージョブ、それにルートクロージャもそうです。契約の実装を手に入れるには、依存を解決するクラスのコンストラクターで「タイプヒント」を指定するだけです。

例として、次のイベントハンドラをご覧ください。

    <?php

    namespace App\Listeners;

    use App\User;
    use App\Events\OrderWasPlaced;
    use Illuminate\Contracts\Redis\Database;

    class CacheOrderInformation
    {
        /**
         * Redisデータベースの実装
         */
        protected $redis;

        /**
         * 新しいイベントハンドラの生成
         *
         * @param  Database  $redis
         * @return void
         */
        public function __construct(Database $redis)
        {
            $this->redis = $redis;
        }

        /**
         * イベントの処理
         *
         * @param  OrderWasPlaced  $event
         * @return void
         */
        public function handle(OrderWasPlaced $event)
        {
            //
        }
    }

イベントリスナの依存解決時に、サービスコンテナはクラスのコンストラクターで指定されているタイプヒントを読み取り、適切な値を注入します。サービスコンテナへ何かを登録する方法を学ぶには、[ドキュメント](/docs/{{version}}/container)を参照してください。

<a name="contract-reference"></a>
## 契約リファレンス

次の一覧表は、全Laravel契約と、同機能のファサードのクイックリファレンスです。

契約  |  対応するファサード
------------- | -------------
[Illuminate\Contracts\Auth\Factory](https://github.com/illuminate/contracts/blob/{{version}}/Auth/Factory.php)  |  Auth
[Illuminate\Contracts\Auth\PasswordBroker](https://github.com/illuminate/contracts/blob/{{version}}/Auth/PasswordBroker.php)  |  Password
[Illuminate\Contracts\Bus\Dispatcher](https://github.com/illuminate/contracts/blob/{{version}}/Bus/Dispatcher.php)  |  Bus
[Illuminate\Contracts\Broadcasting\Broadcaster](https://github.com/illuminate/contracts/blob/{{version}}/Broadcasting/Broadcaster.php)  | &nbsp;
[Illuminate\Contracts\Cache\Repository](https://github.com/illuminate/contracts/blob/{{version}}/Cache/Repository.php) | Cache
[Illuminate\Contracts\Cache\Factory](https://github.com/illuminate/contracts/blob/{{version}}/Cache/Factory.php) | Cache::driver()
[Illuminate\Contracts\Config\Repository](https://github.com/illuminate/contracts/blob/{{version}}/Config/Repository.php) | Config
[Illuminate\Contracts\Container\Container](https://github.com/illuminate/contracts/blob/{{version}}/Container/Container.php) | App
[Illuminate\Contracts\Cookie\Factory](https://github.com/illuminate/contracts/blob/{{version}}/Cookie/Factory.php) | Cookie
[Illuminate\Contracts\Cookie\QueueingFactory](https://github.com/illuminate/contracts/blob/{{version}}/Cookie/QueueingFactory.php) | Cookie::queue()
[Illuminate\Contracts\Encryption\Encrypter](https://github.com/illuminate/contracts/blob/{{version}}/Encryption/Encrypter.php) | Crypt
[Illuminate\Contracts\Events\Dispatcher](https://github.com/illuminate/contracts/blob/{{version}}/Events/Dispatcher.php) | Event
[Illuminate\Contracts\Filesystem\Cloud](https://github.com/illuminate/contracts/blob/{{version}}/Filesystem/Cloud.php) | &nbsp;
[Illuminate\Contracts\Filesystem\Factory](https://github.com/illuminate/contracts/blob/{{version}}/Filesystem/Factory.php) | File
[Illuminate\Contracts\Filesystem\Filesystem](https://github.com/illuminate/contracts/blob/{{version}}/Filesystem/Filesystem.php) | File
[Illuminate\Contracts\Foundation\Application](https://github.com/illuminate/contracts/blob/{{version}}/Foundation/Application.php) | App
[Illuminate\Contracts\Hashing\Hasher](https://github.com/illuminate/contracts/blob/{{version}}/Hashing/Hasher.php) | Hash
[Illuminate\Contracts\Logging\Log](https://github.com/illuminate/contracts/blob/{{version}}/Logging/Log.php) | Log
[Illuminate\Contracts\Mail\MailQueue](https://github.com/illuminate/contracts/blob/{{version}}/Mail/MailQueue.php) | Mail::queue()
[Illuminate\Contracts\Mail\Mailer](https://github.com/illuminate/contracts/blob/{{version}}/Mail/Mailer.php) | Mail
[Illuminate\Contracts\Queue\Factory](https://github.com/illuminate/contracts/blob/{{version}}/Queue/Factory.php) | Queue::driver()
[Illuminate\Contracts\Queue\Queue](https://github.com/illuminate/contracts/blob/{{version}}/Queue/Queue.php) | Queue
[Illuminate\Contracts\Redis\Factory](https://github.com/illuminate/contracts/blob/{{version}}/Redis/Factory.php) | Redis
[Illuminate\Contracts\Routing\Registrar](https://github.com/illuminate/contracts/blob/{{version}}/Routing/Registrar.php) | Route
[Illuminate\Contracts\Routing\ResponseFactory](https://github.com/illuminate/contracts/blob/{{version}}/Routing/ResponseFactory.php) | Response
[Illuminate\Contracts\Routing\UrlGenerator](https://github.com/illuminate/contracts/blob/{{version}}/Routing/UrlGenerator.php) | URL
[Illuminate\Contracts\Support\Arrayable](https://github.com/illuminate/contracts/blob/{{version}}/Support/Arrayable.php) | &nbsp;
[Illuminate\Contracts\Support\Jsonable](https://github.com/illuminate/contracts/blob/{{version}}/Support/Jsonable.php) | &nbsp;
[Illuminate\Contracts\Support\Renderable](https://github.com/illuminate/contracts/blob/{{version}}/Support/Renderable.php) | &nbsp;
[Illuminate\Contracts\Validation\Factory](https://github.com/illuminate/contracts/blob/{{version}}/Validation/Factory.php) | Validator::make()
[Illuminate\Contracts\Validation\Validator](https://github.com/illuminate/contracts/blob/{{version}}/Validation/Validator.php) | &nbsp;
[Illuminate\Contracts\View\Factory](https://github.com/illuminate/contracts/blob/{{version}}/View/Factory.php) | View::make()
[Illuminate\Contracts\View\View](https://github.com/illuminate/contracts/blob/{{version}}/View/View.php) | &nbsp;
