# データベース：マイグレーション

- [イントロダクション](#introduction)
- [マイグレーション生成](#generating-migrations)
- [マイグレーション構造](#migration-structure)
- [マイグレーション実行](#running-migrations)
    - [ロールバック](#rolling-back-migrations)
- [テーブル](#tables)
    - [テーブル作成](#creating-tables)
    - [テーブルリネーム／削除](#renaming-and-dropping-tables)
- [カラム](#columns)
    - [カラム作成](#creating-columns)
    - [カラム修飾子](#column-modifiers)
    - [カラム変更](#modifying-columns)
    - [カラム削除](#dropping-columns)
- [インデックス](#indexes)
    - [インデックス作成](#creating-indexes)
    - [インデックス削除](#dropping-indexes)
    - [外部キー制約](#foreign-key-constraints)

<a name="introduction"></a>
## イントロダクション

マイグレーションとはデータベースのバージョンコントロールのような機能です。アプリケーションデータベースのスキーマの更新をチームで簡単に共有できるようにしてくれます。マイグレーションは基本的にLaravelのスキーマビルダとペアで使い、アプリケーションのデータベーススキーマの作成を楽にしてくれます。もしあなたが今まで、チームメイトに彼らのローカルデータベーススキーマに手作業でカラムを追加するよう依頼したことがあるなら、データベースマイグレーションは、そうした問題を解決してくれます。

Laravelの`Schema`[ファサード](/docs/{{version}}/facades)は、テーブルの作成や操作をサポートしてるデータベースシステム全部に対しサポートします。

<a name="generating-migrations"></a>
## マイグレーション生成

`make:migration` [Artisanコマンド](/docs/{{version}}/artisan)を使いマイグレーションを生成できます。

    php artisan make:migration create_users_table

マイグレーションは`database/migrations`フォルダーに設置されます。マイグレーションの実行順をフレームワークに知らせるため、名前にタイムスタンプが含まれています。

`--table`と`--create`オプションも、テーブル名とマイグレーションで新しいテーブルを生成するかを指定するために使用できます。これらのオプションは生成するマイグレーションスタブの中へ指定したテーブルをただ予め埋め込みます。

    php artisan make:migration create_users_table --create=users

    php artisan make:migration add_votes_to_users_table --table=users

マイグレーションの生成出力先のパスを指定したい場合は、`make:migrate`コマンドの実行時に`--path`オプションを付けてください。パスはアプリケーションのベースパスからの相対位置です。

<a name="migration-structure"></a>
## マイグレーション構造

マイグレーションは`up`と`down`の２メソッドを含んでいます。`up`メソッドは新しいテーブル、カラム、インデックスをデータベースに追加するために使用し、一方の`down`メソッドは`up`メソッドが行った操作を元に戻します。

両方のメソッドでは、記述的にテーブルを作成したり、変更したりできるLaravelスキーマビルダが使えます。`Schema`ビルダで使用できる全メソッドは、[このドキュメント後半](#creating-tables)で確認してください。例として`flights`テーブルを作成するマイグレーションを見てください。

    <?php

    use Illuminate\Support\Facades\Schema;
    use Illuminate\Database\Schema\Blueprint;
    use Illuminate\Database\Migrations\Migration;

    class CreateFlightsTable extends Migration
    {
        /**
         * マイグレーション実行
         *
         * @return void
         */
        public function up()
        {
            Schema::create('flights', function (Blueprint $table) {
                $table->increments('id');
                $table->string('name');
                $table->string('airline');
                $table->timestamps();
            });
        }

        /**
         * マイグレーションを元に戻す
         *
         * @return void
         */
        public function down()
        {
            Schema::drop('flights');
        }
    }


<a name="running-migrations"></a>
## マイグレーション実行

アプリケーションで用意したマイグレーションを全部実行するには、`migrate` Artisanコマンドを使用します。

    php artisan migrate

> {note} [Homestead仮想マシン](/docs/{{version}}/homestead)を使用している場合、このコマンドは仮想マシン内で実行してください。

#### 実働環境でのマイグレーション強制

いくつかのマイグレーション操作は破壊的です。つまりデーターを失う可能性があります。実働環境(production)のデータベースに対し、こうしたコマンドが実行されることから保護するために、コマンド実行前に確認のプロンプトが表示されます。コマンド実行時のプロンプトを出さないためには、`--force`フラグを指定してください。

    php artisan migrate --force

<a name="rolling-back-migrations"></a>
### ロールバック

最後のマイグレーション操作をロールバックしたい場合は、`rollback`コマンドを使います。このロールバックは、最後に「一度に」実行したマイグレーションをまとめて元に戻します。

    php artisan migrate:rollback

`rollback`コマンドに`step`オプションを付けると、巻き戻す数を限定できます。たとえば、次のコマンドは最後の5マイグレーションをロールバックします。

    php artisan migrate:rollback --step=5

`migrate:reset`コマンドはアプリケーション全部のマイグレーションをロールバックします。

    php artisan migrate:reset

#### rollbackとmigrateの１コマンド実行

`migrate:refresh`コマンドは全部のデータベースマイグレーションを最初にロールバックし、それから`migrate`コマンドを実行します。このコマンドはデータベース全体を作り直すために便利です。

    php artisan migrate:refresh

    // データベースをリフレッシュし、全データベースシードを実行
    php artisan migrate:refresh --seed

`refresh`コマンドに`step`オプションを付けると、巻き戻してからマイグレーションを再実行する数を限定できます。たとえば、次のコマンドは最後の5マイグレーションをロールバック後にマイグレートします。

    php artisan migrate:refresh --step=5

<a name="tables"></a>
## テーブル

<a name="creating-tables"></a>
### テーブル作成

新しいデータベーステーブルを作成するには、`Schema`ファサードの`create`メソッドを使用します。`create`メソッドは引数を２つ取ります。最初はテーブルの名前で、２つ目は新しいテーブルを定義するために使用する`Blueprint`オブジェクトを受け取る「クロージャ」です。

    Schema::create('users', function (Blueprint $table) {
        $table->increments('id');
    });

もちろんテーブル作成時には、テーブルのカラムを定義するためにスキーマビルダの[カラムメソッド](#creating-columns)をどれでも利用できます。

#### テーブル／カラムの存在チェック

`hasTable`や`hasColumn`メソッドを使えば、テーブルやカラムの存在を簡単にチェックできます。

    if (Schema::hasTable('users')) {
        //
    }

    if (Schema::hasColumn('users', 'email')) {
        //
    }

#### 接続とストレージエンジン

デフォルト接続以外のデータベース接続でスキーマ操作を行いたい場合は、`connection`メソッドを使ってください。

    Schema::connection('foo')->create('users', function (Blueprint $table) {
        $table->increments('id');
    });

テーブルのストレージエンジンを指定する場合は、スキーマビルダの`engine`プロパティを設定します。

    Schema::create('users', function (Blueprint $table) {
        $table->engine = 'InnoDB';

        $table->increments('id');
    });

<a name="renaming-and-dropping-tables"></a>
### テーブルリネーム／削除

既存のデータベーステーブルの名前を変えたい場合は、`rename`メソッドを使います。

    Schema::rename($from, $to);

存在するテーブルを削除する場合は、`drop`か`dropIfExists`メソッドを使います。

    Schema::drop('users');

    Schema::dropIfExists('users');

#### 外部キーを持つテーブルのリネーム

テーブルのリネームを行う前に、Laravelの規約に基づいた名前の代わりに、マイグレーションファイル中で独自の名前付けた外部キー制約が存在していないか確認してください。そうしないと、外部キー制約名は古いテーブル名を参照してしまいます。

<a name="columns"></a>
## カラム

<a name="creating-columns"></a>
### カラム作成

存在するテーブルを更新するには、`Schema`ファサードの`table`メソッドを使います。`create`メソッドと同様に、`table`メソッドは２つの引数を取ります。テーブルの名前と、テーブルにカラムを追加するために使用する`Blueprint`インスタンスを受け取る「クロージャ」です。

    Schema::table('users', function (Blueprint $table) {
        $table->string('email');
    });

#### 使用できるカラムタイプ

当然ながらスキーマビルダは、テーブルを構築する時に使用する様々なカラムタイプを持っています。

コマンド  | 説明
------------- | -------------
`$table->bigIncrements('id');`  |  「符号なしBIGINT」を使用した自動増分ID（主キー）
`$table->bigInteger('votes');`  |  BIGINTカラム
`$table->binary('data');`  |  BLOBカラム
`$table->boolean('confirmed');`  |  BOOLEANカラム
`$table->char('name', 4);`  |  長さを指定するCHARカラム
`$table->date('created_at');`  |  DATEカラム
`$table->dateTime('created_at');`  |  DATETIMEカラム
`$table->dateTimeTz('created_at');`  |  タイムゾーン付きDATETIMEカラム
`$table->decimal('amount', 5, 2);`  |  有効／小数点以下桁数指定のDECIMALカラム
`$table->double('column', 15, 8);`  |  15桁、小数点以下８桁のDOUBLEカラム
`$table->enum('choices', ['foo', 'bar']);` | ENUMカラム
`$table->float('amount', 8, 2);`  |  8桁、小数点以下2桁のFLOATカラム
`$table->geometry('column');`  | GEOMETRY equivalent for the database.
`$table->geometryCollection('column');`  | GEOMETRYCOLLECTION equivalent for the database.
`$table->increments('id');`  |  「符号なしINT」を使用した自動増分ID（主キー）
`$table->integer('votes');`  |  INTEGERカラム
`$table->ipAddress('visitor');`  |  IPアドレスカラム
`$table->json('options');`  |  JSONフィールド
`$table->jsonb('options');`  |  JSONBフィールド
`$table->lineString('column');`  |  LINESTRING equivalent for the database.
`$table->longText('description');`  |  LONGTEXTカラム
`$table->macAddress('device');`  |  MACアドレスカラム
`$table->mediumIncrements('id');`  |  「符号なしMEDIUMINT」を使用した自動増分ID（主キー）
`$table->mediumInteger('numbers');`  |  MEDIUMINTカラム
`$table->mediumText('description');`  |  MEDIUMTEXTカラム
`$table->morphs('taggable');`  |  符号なしINTERGERの`taggable_id`と文字列の`taggable_type`を追加
`$table->multiLineString('column');`  |  MULTILINESTRING equivalent for the database.
`$table->multiPoint('column');`  |  MULTIPOINT equivalent for the database.
`$table->multiPolygon('column');`  |  MULTIPOLYGON equivalent for the database.
`$table->nullableMorphs('taggable');`  |  Nullableな`morphs()`カラム
`$table->nullableTimestamps();`  |  Nullableな`timestamps()`カラム
`$table->point('column');`  | POINT equivalent for the database.
`$table->polygon('column');`  | POLYGON equivalent for the database.
`$table->rememberToken();`  |  VARCHAR(100) NULLの`remember_token`を追加
`$table->smallIncrements('id');`  |  「符号なしSMALLINT」を使用した自動増分ID（主キー）
`$table->smallInteger('votes');`  |  SMALLINTカラム
`$table->softDeletes();`  |  ソフトデリートのためにNULL値可能な`deleted_at`カラム追加
`$table->string('email');`  |  VARCHARカラム
`$table->string('name', 100);`  |  長さ指定のVARCHARカラム
`$table->text('description');`  |  TEXTカラム
`$table->time('sunrise');`  |  TIMEカラム
`$table->timeTz('sunrise');`  |  タイムゾーン付きTIMEカラム
`$table->tinyInteger('numbers');`  |  TINYINTカラム
`$table->timestamp('added_on');`  |  TIMESTAMPカラム
`$table->timestampTz('added_on');`  |  タイムゾーン付きTIMESTAMPカラム
`$table->timestamps();`  |  NULL値可能な`created_at`と`updated_at`カラム追加
`$table->timestampsTz();`  |  タイムゾーン付きでNULL値可能な`created_at`と`updated_at`カラム追加
`$table->unsignedBigInteger('votes');`  |  符号なしBIGINTカラム
`$table->unsignedInteger('votes');`  |  符号なしINTカラム
`$table->unsignedMediumInteger('votes');`  |  符号なしMEDIUMINTカラム
`$table->unsignedSmallInteger('votes');`  |   符号なしSMALLINTカラム
`$table->unsignedTinyInteger('votes');`  |   符号なしTINYINTカラム
`$table->uuid('id');`  |  データベース向けのUUID類似値

<a name="column-modifiers"></a>
### カラム修飾子

上記のカラムタイプに付け加え、カラムを追加するときに使用できる様々な修飾子もあります。たとえばカラムを「NULL値設定可能(nullable)」にしたい場合は、`nullable`メソッドを使います。

    Schema::table('users', function (Blueprint $table) {
        $table->string('email')->nullable();
    });

下表が使用可能なカラム修飾子の一覧です。[インデックス修飾子](#creating-indexes)は含まれていません。

修飾子  | 説明
------------- | -------------
`->after('column')`  |  指定カラムの次にカラムを設置する(MySQLのみ)
`->comment('my comment')`  |  カラムにコメント追加(MySQLのみ)
`->default($value)`  |  カラムのデフォルト(default)値設定
`->first()`  |  カラムをテーブルの最初(first)に設置する
`->nullable()`  |  カラムにNULL値を許す
`->storedAs($expression)`  |  stored generatedカラムにする(MySQLのみ)
`->unsigned()`  |  整数(integer)を符号無し(unsigned)にする
`->virtualAs($expression)`  |  virtual generatedカラムにする(MySQLのみ)

<a name="changing-columns"></a>
<a name="modifying-columns"></a>
### カラム変更

#### 動作要件

カラムを変更する前に、`composer.json`ファイルで`doctrine/dbal`を確実に追加してください。Doctrine DBALライブラリーは現在のカラムの状態を決め、指定されたカラムに対する修正を行うSQLクエリを生成するために、使用しています。

    composer require doctrine/dbal

#### カラム属性の変更

`change`メソッドは存在するカラムを新しいタイプへ変更するか、カラムの属性を変えます。たとえば文字列の長さを増やしたい場合です。`change`の実例を見てもらうため、`name`カラムのサイズを25から50へ増やしてみます。

    Schema::table('users', function (Blueprint $table) {
        $table->string('name', 50)->change();
    });

さらにカラムをNULL値設定可能にしてみましょう。

    Schema::table('users', function (Blueprint $table) {
        $table->string('name', 50)->nullable()->change();
    });

> {note} 以降のカラムタイプは変更できません：char、double、enum、mediumInteger、timestamp、tinyInteger、ipAddress、json、jsonb、macAddress、mediumIncrements、morphs、nullableMorphs、nullableTimestamps、softDeletes、timeTz、timestampTz、timestamps、timestampsTz、unsignedMediumInteger、unsignedTinyInteger、uuid

<a name="renaming-columns"></a>
#### カラム名変更

カラム名を変更するには、`renameColumn`メソッドをスキーマビルダで使用してください。カラム名を変更する前に、`composer.json`ファイルで`doctrine/dbal`を依存パッケージとして追加してください。

    Schema::table('users', function (Blueprint $table) {
        $table->renameColumn('from', 'to');
    });

> {note} カラムタイプが`enum`のテーブル中のカラム名変更は、現在サポートしていません。

<a name="dropping-columns"></a>
### カラム削除

カラムをドロップするには、スキーマビルダの`dropColumn`メソッドを使用します。SQLiteデータベースからカラムをドロップする場合は、事前に`composer.json`ファイルへ`doctrine/dbal`依存パッケージを追加してください。その後にライブラリーをインストールするため、ターミナルで`composer update`を実行してください。

    Schema::table('users', function (Blueprint $table) {
        $table->dropColumn('votes');
    });

`dropColumn`メソッドにカラム名の配列を渡せば、テーブルから複数のカラムをドロップできます。

    Schema::table('users', function (Blueprint $table) {
        $table->dropColumn(['votes', 'avatar', 'location']);
    });

> {note} SQLite使用時に、一つのマイグレーションによる複数カラム削除／変更はサポートされていません。

<a name="indexes"></a>
## インデックス

<a name="creating-indexes"></a>
### インデックス作成

スキーマビルダは様々なインデックスタイプをサポートしています。まず指定したカラムの値を一意にする例を見てください。インデックスを作成するには、カラム定義に`unique`メソッドをチェーンで付け加えるだけです。

    $table->string('email')->unique();

もしくはカラム定義の後でインデックスを作成することも可能です。例を見てください。

    $table->unique('email');

インデックスメソッドにカラムの配列を渡し、複合インデックスを作成することもできます。

    $table->index(['account_id', 'created_at']);

Laravelは自動的に、わかりやすいインデックス名を付けます。しかしメソッドの第２引数で、名前を指定することもできます。

    $table->index('email', 'my_index_name');

#### 使用可能なインデックスタイプ

コマンド  | 説明
------------- | -------------
`$table->primary('id');`  |  主キー追加
`$table->primary(['first', 'last']);`  |  複合キー追加
`$table->unique('email');`  | uniqueキー追加
`$table->unique('state', 'my_index_name');`  |  インデックス名のカスタマイズ
`$table->unique(['first', 'last']);`  |  uniqueな複合キー追加
`$table->index('state');`  |  基本的なインデックス追加

#### インデックス長とMySQL／MariaDB

Laravelはデータベース中への「絵文字」保存をサポートするため、デフォルトで`utf8mb4`文字セットを使っています。バージョン5.7.7より古いMySQLや、バージョン10.2.2より古いMariaDBを使用している場合、マイグレーションにより生成されるデフォルトのインデックス用文字列長を明示的に設定する必要があります。`AppServiceProvider`中で`Schema::defaultStringLength`を呼び出してください。

    use Illuminate\Support\Facades\Schema;

    /**
     * 全アプリケーションサービスの初期起動処理
     *
     * @return void
     */
    public function boot()
    {
        Schema::defaultStringLength(191);
    }

もしくは、データベースの`innodb_large_prefix`オプションを有効にする方法もあります。このオプションを各自に有効にする方法は、使用するデータベースのドキュメントを参照してください。

<a name="dropping-indexes"></a>
### インデックス削除

インデックスを削除する場合はインデックスの名前を指定します。Laravelはデフォルトで意味が通る名前をインデックスに付けます。シンプルにテーブル名、インデックスしたカラム名、インデックスタイプをつなげたものです。いくつか例をご覧ください。

コマンド  | 説明
------------- | -------------
`$table->dropPrimary('users_id_primary');`  |  "users"テーブルから主キーを削除
`$table->dropUnique('users_email_unique');`  |  "users"テーブルからユニークキーを削除
`$table->dropIndex('geo_state_index');`  |  "geo"テーブルから基本インデックスを削除

カラムの配列をインデックス削除メソッドに渡すと、テーブル、カラム、キータイプに基づき、命名規則に従ったインデックス名が生成されます。

    Schema::table('geo', function (Blueprint $table) {
        $table->dropIndex(['state']); // 'geo_state_index'インデックスを削除
    });

<a name="foreign-key-constraints"></a>
### 外部キー制約

Laravelはデータベースレベルの整合性を強制するために、テーブルに対する外部キー束縛の追加も提供しています。たとえば`users`テーブルの`id`カラムを参照する、`posts`テーブルの`user_id`カラムを定義してみましょう。

    Schema::table('posts', function (Blueprint $table) {
        $table->integer('user_id')->unsigned();

        $table->foreign('user_id')->references('id')->on('users');
    });

さらに束縛に対して「デリート時(on delete)」と「更新時(on update)」に対する処理をオプションとして指定できます。

    $table->foreign('user_id')
          ->references('id')->on('users')
          ->onDelete('cascade');

外部キーを削除するには、`dropForeign`メソッドを使用します。他のインデックスで使用されるものと似た命名規則が、外部キーにも使用されています。つまりテーブル名とカラム名をつなげ、"_foreign"を最後につけた名前になります。

    $table->dropForeign('posts_user_id_foreign');

もしくは配列値を渡せば、削除時に自動的に命名規則に従った名前が使用されます。

    $table->dropForeign(['user_id']);

以下のメソッドにより、マイグレーション中の外部キー制約の有効／無効を変更できます。

    Schema::enableForeignKeyConstraints();

    Schema::disableForeignKeyConstraints();
