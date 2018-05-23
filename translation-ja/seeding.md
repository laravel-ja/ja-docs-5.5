# データベース：シーディング

- [イントロダクション](#introduction)
- [シーダクラス定義](#writing-seeders)
    - [モデルファクトリの使用](#using-model-factories)
    - [追加のシーダ呼び出し](#calling-additional-seeders)
- [シーダの実行](#running-seeders)

<a name="introduction"></a>
## イントロダクション

シーダ（初期値設定）クラスを使用し、テストデーターをデーターベースに設定するシンプルな方法もLaravelには備わっています。全シーダクラスは`database/seeds`に保存します。シーダクラスには好きな名前を付けられます。しかし`UsersTableSeeder`などのような分かりやすい規則に従ったほうが良いでしょう。デフォルトとして`DatabaseSeeder`クラスが定義されています。このクラスから`call`メソッドを使い他の初期値設定クラスを呼び出すことで、シーディングの順番をコントロールできます。

<a name="writing-seeders"></a>
## シーダクラス定義

シーダーを生成するには、`make:seeder` [Artisanコマンド](/docs/{{version}}/artisan)を実行します。フレームワークが生成するシーダーは全て`database/seeds`ディレクトリに設置されます。

    php artisan make:seeder UsersTableSeeder

シーダクラスはデフォルトで`run`メソッドだけを含んでいます。このメソッドは`db:seed` [Artisanコマンド](/docs/{{version}}/artisan)が実行された時に呼びだされます。`run`メソッドの中でデータベースに何でも好きなデーターを挿入できます。[クエリビルダ](/docs/{{version}}/queries)でデータを挿入することも、もしくは[Eloquentモデルファクトリ](/docs/{{version}}/database-testing#writing-factories)を使うこともできます。

> {tip} データベースシーディング時、[複数代入](/docs/{{version}}/eloquent#mass-assignment)は自動的に無効になります。

例として、Laravelのインストール時にデフォルトで用意されている`DatabaseSeeder`クラスを変更してみましょう。`run`メソッドにデータベースINSERT文を追加します。

    <?php

    use Illuminate\Database\Seeder;
    use Illuminate\Support\Facades\DB;

    class DatabaseSeeder extends Seeder
    {
        /**
         * データベース初期値設定の実行
         *
         * @return void
         */
        public function run()
        {
            DB::table('users')->insert([
                'name' => str_random(10),
                'email' => str_random(10).'@gmail.com',
                'password' => bcrypt('secret'),
            ]);
        }
    }

<a name="using-model-factories"></a>
### モデルファクトリの利用

もちろんそれぞれのモデルシーダで属性をいちいち指定するのは面倒です。代わりに大量のデータベースレコードを生成するのに便利な[モデルファクトリ](/docs/{{version}}/database-testing#writing-factories)が使用できます。最初に[モデルファクトリのドキュメント](/docs/{{version}}/database-testing#writing-factories)を読んで、どのように定義するのかを学んでください。ファクトリが定義できれば、データベースにレコードを挿入する`factory`ヘルパ関数が利用できます。

例として50件のレコードを生成し、それぞれのユーザーへリレーションを付加してみましょう。

    /**
     * データベース初期値設定の実行
     *
     * @return void
     */
    public function run()
    {
        factory(App\User::class, 50)->create()->each(function ($u) {
            $u->posts()->save(factory(App\Post::class)->make());
        });
    }

<a name="calling-additional-seeders"></a>
### 追加のシーダ呼び出し

`DatabaseSeeder`クラスの中で追加のシーダクラスを呼び出す`call`メソッドが使えます。`call`メソッドを使うことで、圧倒されるぐらい大きな１ファイルを使う代わりに、データベースシーディングを複数のファイルへ分割できます。実行したいシーダクラス名を渡します。

    /**
     * データベース初期値設定の実行
     *
     * @return void
     */
    public function run()
    {
        $this->call([
            UsersTableSeeder::class,
            PostsTableSeeder::class,
            CommentsTableSeeder::class,
        ]);
    }

<a name="running-seeders"></a>
## シーダの実行

シーダクラスを書き上げたら、Composerのオートローダを再生成するために、`dump-autoload`コマンドを実行する必要があります。

    composer dump-autoload

データベースへ初期値を設定するために`db:seed` Artisanコマンドを使用します。デフォルトで`db:seed`コマンドは、他のシーダクラスを呼び出す`DatabaseSeeder`クラスを実行します。しかし特定のファイルを個別に実行したい場合は、`--class`オプションを使いシーダを指定してください。

    php artisan db:seed

    php artisan db:seed --class=UsersTableSeeder

もしくはマイグレーションをロールバックし再実行する`migrate:refresh`コマンドを使っても、データベースに初期値を設定できます。このコマンドはデータベースを完全に作成し直したい場合に便利です。

    php artisan migrate:refresh --seed
