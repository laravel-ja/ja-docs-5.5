# データベースのテスト

- [イントロダクション](#introduction)
- [各テスト後のデータベースリセット](#resetting-the-database-after-each-test)
    - [マイグレーションの使用](#using-migrations)
    - [トランザクションの使用](#using-transactions)
- [ファクトリの記述](#writing-factories)
    - [ファクトリステート](#factory-states)
- [ファクトリの使用](#using-factories)
    - [モデルの生成](#creating-models)
    - [モデルの保存](#persisting-models)
    - [リレーション](#relationships)
- [使用可能なアサーション](#available-assertions)

<a name="introduction"></a>
## イントロダクション

Laravelでは、データベースを駆動するアプリケーションのテストを簡単にできる、便利で様々なツールを用意しています。その一つは、指定した抽出条件に一致するデータがデータベース中に存在するかをアサートする、`assertDatabaseHas`ヘルパです。たとえば、`user`テーブルの中に`email`フィールドが`sally@example.com`の値のレコードが存在するかを確認したいとしましょう。次のようにテストできます。

    public function testDatabase()
    {
        // アプリケーションを呼び出す…

        $this->assertDatabaseHas('users', [
            'email' => 'sally@example.com'
        ]);
    }

データベースにデータが存在しないことをアサートする、`assertDatabaseMissing`ヘルパを使うこともできます。

もちろん、`assertDatabaseHas`メソッドやその他のヘルパは、便利に使ってもらうため用意しています。PHPUnitの組み込みアサートメソッドは、テストで自由に使用できます。

<a name="resetting-the-database-after-each-test"></a>
## 各テスト後のデータベースリセット

前のテストデータが、それに引き続くテストへ影響をあたえないように、各テストが終了するごとにデータベースをリセットするのは、時に便利です。

<a name="using-migrations"></a>
### マイグレーションの使用

一つのアプローチは、それぞれのテストの後にデータベースをロールバックし、次のテストの前にマイグレーションする方法です。Laravelはこれを自動的に処理するために、シンプルな`DatabaseMigrations`トレイトを用意しています。テストクラスでこのトレイトを使用するだけで、すべて処理されます。

    <?php

    namespace Tests\Feature;

    use Tests\TestCase;
    use Illuminate\Foundation\Testing\WithoutMiddleware;
    use Illuminate\Foundation\Testing\DatabaseMigrations;
    use Illuminate\Foundation\Testing\DatabaseTransactions;

    class ExampleTest extends TestCase
    {
        use DatabaseMigrations;

        /**
         * 基本的な機能テストの例
         *
         * @return void
         */
        public function testBasicExample()
        {
            $response = $this->get('/');

            // ...
        }
    }

<a name="using-transactions"></a>
### トランザクションの使用

データベースをリセットするもう一つのアプローチは、各テストケースをデータベーストランザクションでラップしてしまうことです。Laravelはこれを自動的に処理する、便利な`DatabaseTransactions`トレイトを用意してます。

    <?php

    namespace Tests\Feature;

    use Tests\TestCase;
    use Illuminate\Foundation\Testing\WithoutMiddleware;
    use Illuminate\Foundation\Testing\DatabaseMigrations;
    use Illuminate\Foundation\Testing\DatabaseTransactions;

    class ExampleTest extends TestCase
    {
        use DatabaseTransactions;

        /**
         * 基本的な機能テストの例
         *
         * @return void
         */
        public function testBasicExample()
        {
            $response = $this->get('/');

            // ...
        }
    }

> {note} デフォルトでは、このトレイトは、トランザクション中のデフォルトデータベース接続をラップしているだけです。もし、アプリケーションで複数のデータベース接続を使用している場合は、テストクラスの`$connectionsToTransact`プロパティを定義する必要があります。このプロパティはトランザクションで実行する接続名の配列を指定します。

<a name="writing-factories"></a>
## ファクトリの記述

テスト実行前に、何件かのレコードをデータベースに挿入する必要があります。こうしたテストデータを作る時に、手動でそれぞれのカラムへ値を指定する代わりに、Laravelではモデルファクトリを使用し、[Eloquentモデル](/docs/{{version}}/eloquent)の各属性にデフォルト値を設定できます。手始めに、アプリケーションの`database/factories/UserFactory.php`ファイルを見てください。このファイルには初めからファクトリの定義が一つ含まれています。

    $factory->define(App\User::class, function (Faker\Generator $faker) {
        static $password;

        return [
            'name' => $faker->name,
            'email' => $faker->unique()->safeEmail,
            'password' => $password ?: $password = bcrypt('secret'),
            'remember_token' => str_random(10),
        ];
    });

ファクトリで定義しているクロージャの中から、モデルの全属性に対するデフォルトのテスト値を返しています。このクロージャは[Faker](https://github.com/fzaninotto/Faker) PHPライブラリーのインスタンスを受け取っており、これはテストのための多用なランダムデータを生成するのに便利です。

より組織立てるために、各モデルごとに追加のファクトリファイルを作成することもできます。たとえば、`database/factories`ディレクトリに`UserFactory.php`と`CommentFactory.php`ファイルを作成できます。`factories`ディレクトリ下の全ファイルは、Laravelにより自動的にロードされます。

<a name="factory-states"></a>
### ファクトリステート

ステートにより、モデルファクトリのどんな組み合わせに対しても適用できる、個別の調整を定義できます。たとえば、`User`モデルは、デフォルト属性値の一つを変更する、`delinquent`ステートを持つとしましょう。`state`メソッドを使い、状態変換を定義します。単純なステートでは、属性の変換の配列を渡します。

    $factory->state(App\User::class, 'delinquent', [
        'account_status' => 'delinquent',
    ]);

ステートで計算や`$faker`インスタンスが必要な場合は、ステートの属性変換を計算するために、クロージャを使います。

    $factory->state(App\User::class, 'address', function ($faker) {
        return [
            'address' => $faker->address,
        ];
    });

<a name="using-factories"></a>
## ファクトリの使用

<a name="creating-models"></a>
### モデルの生成

ファクトリを定義し終えたら、テストかデータベースのシーディング（初期値設定）ファイルの中で、グローバルな`factory`関数を使用してモデルインスタンスを生成できます。では、モデル生成の例をいくつか見てみましょう。最初は`make`メソッドでモデルを生成し、データベースには保存しない方法です。

    public function testDatabase()
    {
        $user = factory(App\User::class)->make();

        // モデルをテストで使用…
    }

さらにモデルのコレクションや指定したタイプのモデルも生成できます。

    // App\Userインスタンスを３つ生成
    $users = factory(App\User::class, 'admin', 3)->make();

#### ステートの適用

    $users = factory(App\User::class, 5)->states('premium', 'deliquent')->make();

    $users = factory(App\User::class, 5)->states('premium', 'deliquent')->make();

    $users = factory(App\User::class, 5)->states('premium', 'deliquent')->make();

#### 属性のオーバーライド

モデルのデフォルト値をオーバーライドしたい場合は、`make`メソッドに配列で値を渡してください。指定した値のみ置き換わり、残りの値はファクトリで指定したデフォルト値のまま残ります。

    $user = factory(App\User::class)->make([
        'name' => 'Abigail',
    ]);

<a name="persisting-models"></a>
### モデルの保存

`create`メソッドはモデルインスタンスを生成するだけでなく、Eloquentの`save`メソッドを使用しデータベースへ保存します。

    public function testDatabase()
    {
        // 一つのApp\Userインスタンスを作成
        $user = factory(App\User::class)->create();

        // App\Userインスタンスを３つ生成
        $users = factory(App\User::class, 3)->create();

        // モデルをテストで使用…
    }

`create`メソッドに配列で値を渡すことで、モデルの属性をオーバーライドできます。

    $user = factory(App\User::class)->create([
        'name' => 'Abigail',
    ]);

<a name="relationships"></a>
### リレーション

以下の例では、生成したモデルにリレーションを付けています。複数モデルの生成に`create`メソッドを使用する場合、[インスタンスのコレクション](/docs/{{version}}/eloquent-collections)が返されます。そのため、コレクションで使用できる`each`などの便利な関数が利用できます。

    $users = factory(App\User::class, 3)
               ->create()
               ->each(function ($u) {
                    $u->posts()->save(factory(App\Post::class)->make());
                });

#### リレーションと属性クロージャ

クロージャ属性をファクトリ定義の中で使い、モデルとのリレーションを追加することもできます。たとえば、`Post`を作成する時に、新しい`User`インスタンスも作成したい場合は、以下のようになります。

    $factory->define(App\Post::class, function ($faker) {
        return [
            'title' => $faker->title,
            'content' => $faker->paragraph,
            'user_id' => function () {
                return factory(App\User::class)->create()->id;
            }
        ];
    });

さらに、クロージャは評価済みのファクトリーの属性配列を受け取ることもできます。

    $factory->define(App\Post::class, function ($faker) {
        return [
            'title' => $faker->title,
            'content' => $faker->paragraph,
            'user_id' => function () {
                return factory(App\User::class)->create()->id;
            },
            'user_type' => function (array $post) {
                return App\User::find($post['user_id'])->type;
            }
        ];
    });

<a name="available-assertions"></a>
## 使用可能なアサーション

Laravelは、多くのデータベースアサーションを[PHPUnit](https://phpunit.de/)テスト向けに提供しています。

メッソッド  | 説明
------------- | -------------
`$this->assertDatabaseHas($table, array $data);`  |  指定したデータが、テーブルに存在することをアサート
`$this->assertDatabaseMissing($table, array $data);`  |  指定したデータが、テーブルに含まれないことをアサート
`$this->assertSoftDeleted($table, array $data);`  |  指定したレコードがソフトデリートされていることをアサート
