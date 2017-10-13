# データベース：クエリビルダ

- [イントロダクション](#introduction)
- [結果の取得](#retrieving-results)
    - [結果の分割](#chunking-results)
    - [集計](#aggregates)
- [SELECT](#selects)
- [SQL文](#raw-expressions)
- [JOIN](#joins)
- [UNION](#unions)
- [WHERE節](#where-clauses)
    - [パラメータのグループ分け](#parameter-grouping)
    - [Where Exists節](#where-exists-clauses)
    - [JSON Where節](#json-where-clauses)
- [順序、グループ分け、制限、オフセット](#ordering-grouping-limit-and-offset)
- [条件節](#conditional-clauses)
- [INSERT](#inserts)
- [UPDATE](#updates)
    - [JSONカラムの更新](#updating-json-columns)
    - [増減分](#increment-and-decrement)
- [DELETE](#deletes)
- [排他的ロック](#pessimistic-locking)

<a name="introduction"></a>
## イントロダクション

データベースクエリビルダはスラスラと書ける(fluent)便利なインターフェイスで、クエリを作成し実行するために使用します。アプリケーションで行われるほとんどのデーターベース操作が可能で、サポートしている全データベースシステムに対し使用できます。

LaravelクエリビルダはアプリケーションをSQLインジェクション攻撃から守るために、PDOパラメーターによるバインディングを使用します。バインドする文字列をクリーンにしてから渡す必要はありません。

<a name="retrieving-results"></a>
## 結果の取得

#### 全レコードの取得

クエリを書くには`DB`ファサードの`table`メソッドを使います。`table`メソッドは指定したテーブルに対するクエリビルダインスタンスを返します。これを使いクエリに制約を加え、最終的な結果を取得するチェーンを繋げます。次に、最終的な結果を`get`で取得します。

    <?php

    namespace App\Http\Controllers;

    use Illuminate\Support\Facades\DB;
    use App\Http\Controllers\Controller;

    class UserController extends Controller
    {
        /**
         * アプリケーションの全ユーザーレコード一覧を表示
         *
         * @return Response
         */
        public function index()
        {
            $users = DB::table('users')->get();

            return view('user.index', ['users' => $users]);
        }
    }

`get`メソッドは、PHPの`StdClass`オブジェクトのインスタンスを結果として含む、`Illuminate\Support\Collection`を返します。各カラムの値は、オブジェクトのプロパティとしてアクセスできます。

    foreach ($users as $user) {
        echo $user->name;
    }

#### テーブルから１カラム／１レコード取得

データベーステーブルから１レコードのみ取得する必要がある場合は、`first`メソッドを使います。このメソッドは`StdClass`オブジェクトを返します。

    $user = DB::table('users')->where('name', 'John')->first();

    echo $user->name;

全カラムは必要ない場合、`value`メソッドにより一つの値のみ取得できます。このメソッドはカラムの値を直接返します。

    $email = DB::table('users')->where('name', 'John')->value('email');

#### カラム値をリストで取得

単一カラムの値をコレクションで取得したい場合は`pluck`メソッドを使います。以下の例では役割名(title)をコレクションで取得しています。

    $titles = DB::table('roles')->pluck('title');

    foreach ($titles as $title) {
        echo $title;
    }

取得コレクションのキーカラムを指定することもできます。

    $roles = DB::table('roles')->pluck('title', 'name');

    foreach ($roles as $name => $title) {
        echo $title;
    }

<a name="chunking-results"></a>
### 結果の分割

数千のデータベースレコードを扱う場合は`chunk`メソッドの使用を考慮してください。このメソッドは一度に小さな「かたまり(chunk)」で結果を取得し、このチャンクは処理のために「クロージャ」に渡されます。このメソッドは数千のレコードを処理する[Artisanコマンド](/docs/{{version}}/artisan)を書くときに便利です。`users`レコード全体から一度に１００レコードずつチャンクを処理する例を見てください。

    DB::table('users')->orderBy('id')->chunk(100, function ($users) {
        foreach ($users as $user) {
            //
        }
    });

クロージャから`false`を返すとチャンクの処理を中断できます。

    DB::table('users')->orderBy('id')->chunk(100, function ($users) {
        // レコードの処理…

        return false;
    });

<a name="aggregates"></a>
### 集計

またクエリビルダは`count`、`max`、`min`、`avg`、`sum`など多くの集計メソッドを提供しています。クエリを制約した後にこれらのメソッドを使うことも可能です。

    $users = DB::table('users')->count();

    $price = DB::table('orders')->max('price');

もちろんこれらのメソッドをクエリを構築するために他の節と組み合わせて使用できます。

    $price = DB::table('orders')
                    ->where('finalized', 1)
                    ->avg('price');

<a name="selects"></a>
## SELECT

#### SELECT節の指定

もちろん、いつもデータベースレコードの全カラムが必要ではないでしょう。クエリの`select`節を`select`メソッドで指定できます。

    $users = DB::table('users')->select('name', 'email as user_email')->get();

`distinct`メソッドで重複行をまとめた結果を返すように強制できます。

    $users = DB::table('users')->distinct()->get();

既にクエリビルダインスタンスがあり、select節にカラムを追加したい場合は`addSelect`メソッドを使ってください。

    $query = DB::table('users')->select('name');

    $users = $query->addSelect('age')->get();

<a name="raw-expressions"></a>
## SQL文

たまにクエリの中でSQLを直接使用したいことがあります。このようなSQLでは文字をそのまま埋め込むだけですので、SQLインジェクションをされないように気をつけてください！　エスケープなしのSQLを使用する場合は`DB::raw`メソッドを使用します。

    $users = DB::table('users')
                         ->select(DB::raw('count(*) as user_count, status'))
                         ->where('status', '<>', 1)
                         ->groupBy('status')
                         ->get();

<a name="joins"></a>
## JOIN

#### INNER JOIN文

さらにクエリビルダはJOIN文を書くためにも使用できます。基本的な"INNER JOIN"を実行するには、クエリビルダインスタンスに`join`メソッドを使ってください。`join`メソッドの第１引数は結合したいテーブル名、それ以降の引数にはJOIN時のカラムの制約条件を指定します。もちろん下記のように、一つのクエリで複数のテーブルを結合できます。

    $users = DB::table('users')
                ->join('contacts', 'users.id', '=', 'contacts.user_id')
                ->join('orders', 'users.id', '=', 'orders.user_id')
                ->select('users.*', 'contacts.phone', 'orders.price')
                ->get();

#### LEFT JOIN文

"INNER JOIN"の代わりに"LEFT JOIN"を実行したい場合は`leftJoin`メソッドを使います。`leftJoin`メソッドの使い方は`join`メソッドと同じです。

    $users = DB::table('users')
                ->leftJoin('posts', 'users.id', '=', 'posts.user_id')
                ->get();

#### クロスジョイン文

「クロスジョイン」を実行するときは、接合したいテーブル名を指定し、`crossJoin`メソッドを使ってください。クロスジョインにより、最初のテーブルと指定したテーブルとの、デカルト積を生成します。

    $users = DB::table('sizes')
                ->crossJoin('colours')
                ->get();

#### 上級のJOIN文

さらに上級なJOIN節を指定することもできます。そのためには`join`メソッドの第２引数に「クロージャ」を指定します。その「クロージャ」は`JOIN`節に制約を指定できるようにする`JoinClause`オブジェクトを受け取ります。

    DB::table('users')
            ->join('contacts', function ($join) {
                $join->on('users.id', '=', 'contacts.user_id')->orOn(...);
            })
            ->get();

JOINに"where"節を使用したい場合はjoinの中で`where`や`orWhere`を使用して下さい。２つのカラムを比べる代わりに、これらのメソッドは値とカラムを比較します。

    DB::table('users')
            ->join('contacts', function ($join) {
                $join->on('users.id', '=', 'contacts.user_id')
                     ->where('contacts.user_id', '>', 5);
            })
            ->get();

<a name="unions"></a>
## UNION

クエリビルダは２つのクエリを結合(union)させる手軽な手法を提供します。たとえば最初にクエリを作成し、その後に２つ目のクエリを結合するために`union`メソッドを使います。

    $first = DB::table('users')
                ->whereNull('first_name');

    $users = DB::table('users')
                ->whereNull('last_name')
                ->union($first)
                ->get();

> {tip} `union`と同じ使い方の`unionAll`メソッドも使えます。

<a name="where-clauses"></a>
## WHERE節

#### 単純なWHERE節

`where`節をクエリに追加するには、クエリビルダインスタンスの`where`メソッドを使います。基本的な`where`の呼び出しでは３つの引数を使います。第１引数はカラム名です。第２引数はデータベースがサポートしているオペレーターです。第３引数はカラムに対して比較する値です。

例として、"votes"カラムの値が100と等しいレコードのクエリを見てください。

    $users = DB::table('users')->where('votes', '=', 100)->get();

利便性のため、カラムが指定値と等しいかを比べたい場合は、`where`メソッドの第２引数に値をそのまま指定できます。

    $users = DB::table('users')->where('votes', 100)->get();

もちろん、`where`文を書くときには、その他いろいろなオペレータも使えます。

    $users = DB::table('users')
                    ->where('votes', '>=', 100)
                    ->get();

    $users = DB::table('users')
                    ->where('votes', '<>', 100)
                    ->get();

    $users = DB::table('users')
                    ->where('name', 'like', 'T%')
                    ->get();

`where`に配列で条件を渡すこともできます。

    $users = DB::table('users')->where([
        ['status', '=', '1'],
        ['subscribed', '<>', '1'],
    ])->get();

#### OR節

WHEREの結合にチェーンで`or`節をクエリに追加できます。`orWhere`メソッドは`where`メソッドと同じ引数を受け付けます。

    $users = DB::table('users')
                        ->where('votes', '>', 100)
                        ->orWhere('name', 'John')
                        ->get();

#### その他のWHERE節

**whereBetween**

`whereBetween`メソッドはカラムの値が２つの値の間である条件を加えます。

    $users = DB::table('users')
                        ->whereBetween('votes', [1, 100])->get();

**whereNotBetween**

`whereNotBetween`メソッドは、カラムの値が２つの値の間ではない条件を加えます。

    $users = DB::table('users')
                        ->whereNotBetween('votes', [1, 100])
                        ->get();

**whereIn / whereNotIn**

`whereIn`メソッドは指定した配列の中にカラムの値が含まれている条件を加えます。

    $users = DB::table('users')
                        ->whereIn('id', [1, 2, 3])
                        ->get();

`whereNotIn`メソッドはカラムの値が指定した配列の中に含まれて**いない**条件を加えます。

    $users = DB::table('users')
                        ->whereNotIn('id', [1, 2, 3])
                        ->get();

**whereNull / whereNotNull**

`whereNull`メソッドは指定したカラムの値が`NULL`である条件を加えます。

    $users = DB::table('users')
                        ->whereNull('updated_at')
                        ->get();

`whereNotNull`メソッドは指定したカラムの値が`NULL`でない条件を加えます。

    $users = DB::table('users')
                        ->whereNotNull('updated_at')
                        ->get();

**whereDate / whereMonth / whereDay / whereYear**

`whereDate`メソッドはカラム値を日付と比較する時に使用します。

    $users = DB::table('users')
                    ->whereDate('created_at', '2016-12-31')
                    ->get();

`whereMonth`メソッドはカラム値と、ある年の指定した月とを比較します。

    $users = DB::table('users')
                    ->whereMonth('created_at', '12')
                    ->get();

`whereDay`メソッドはカラム値と、ある月の指定した日とを比べます。

    $users = DB::table('users')
                    ->whereDay('created_at', '31')
                    ->get();

`whereYear`メソッドはカラム値と、指定した年とを比べます。

    $users = DB::table('users')
                    ->whereYear('created_at', '2016')
                    ->get();

**whereColumn**

`whereColumn`メソッドは２つのカラムが同値である確認をするのに使います。

    $users = DB::table('users')
                    ->whereColumn('first_name', 'last_name')
                    ->get();

メソッドに比較演算子を追加指定することもできます。

    $users = DB::table('users')
                    ->whereColumn('updated_at', '>', 'created_at')
                    ->get();

`whereColumn`へ配列により複数の条件を渡すこともできます。各条件は`and`オペレータでつなげられます。

    $users = DB::table('users')
                    ->whereColumn([
                        ['first_name', '=', 'last_name'],
                        ['updated_at', '>', 'created_at']
                    ])->get();

<a name="parameter-grouping"></a>
### パラメータのグループ分け

時には、"WHERE EXISTS"節やグループにまとめたパラーメーターのネストのような、上級のWHERE節を作成する必要が起きます。Laravelクエリビルダはこれらもうまく処理できます。手始めに、カッコで制約をまとめる例を見てください。

    DB::table('users')
                ->where('name', '=', 'John')
                ->orWhere(function ($query) {
                    $query->where('votes', '>', 100)
                          ->where('title', '<>', 'Admin');
                })
                ->get();

ご覧の通り、`orWhere`メソッドに渡している「クロージャ」が、クエリビルダのグルーピングを指示しています。生成するSQLの括弧内で展開される制約を指定できるように、「クロージャ」はクエリビルダのインスタンスを受け取ります。

    select * from users where name = 'John' or (votes > 100 and title <> 'Admin')

<a name="where-exists-clauses"></a>
### Where Exists節

`whereExists`メソッドは`WHERE EXISTS`のSQLを書けるように用意しています。`whereExists`メソッドは引数に「クロージャ」を取り、"EXISTS"節の中に置かれるクエリを定義するためのクエリビルダを受け取ります。

    DB::table('users')
                ->whereExists(function ($query) {
                    $query->select(DB::raw(1))
                          ->from('orders')
                          ->whereRaw('orders.user_id = users.id');
                })
                ->get();

上のクエリは以下のSQLを生成します。

    select * from users
    where exists (
        select 1 from orders where orders.user_id = users.id
    )

<a name="json-where-clauses"></a>
### JSON WHERE節

Laravelはデータベース上のJSONタイプをサポートするカラムに対するクエリに対応しています。現在、MySQL5.7とPostgreSQLに対応しています。JSONカラムをクエリするには`->`オペレータを使ってください。

    $users = DB::table('users')
                    ->where('options->language', 'en')
                    ->get();

    $users = DB::table('users')
                    ->where('preferences->dining->meal', 'salad')
                    ->get();

<a name="ordering-grouping-limit-and-offset"></a>
## 順序、グループ分け、制限、オフセット

#### orderBy

`orderBy`メソッドは指定したカラムでクエリ結果をソートします。`orderBy`メソッドの最初の引数はソート対象のカラムで、第２引数はソートの昇順(`asc`)と降順(`desc`)をコントロールします。

    $users = DB::table('users')
                    ->orderBy('name', 'desc')
                    ->get();

#### latest／oldest

`latest`と`oldest`メソッドにより、データの結果を簡単に整列できます。デフォルトで、結果は`created_at`カラムによりソートされます。ソートキーとしてカラム名を渡すこともできます。

    $user = DB::table('users')
                    ->latest()
                    ->first();

#### inRandomOrder

`inRandomOrder`メソッドはクエリ結果をランダムな順番にしたい時に使用します。たとえば、以下のコードはランダムにユーザーを一人取得します。

    $randomUser = DB::table('users')
                    ->inRandomOrder()
                    ->first();

#### groupBy / having / havingRaw

`groupBy`と`having`メソッドはクエリ結果をグループにまとめるために使用します。`having`メソッドは`where`メソッドと似た使い方です。

    $users = DB::table('users')
                    ->groupBy('account_id')
                    ->having('account_id', '>', 100)
                    ->get();

`havingRaw`メソッドは`haveing`節の値としてSQL文字列をそのまま指定するために使用します。たとえば2,500ドルより多く売り上げている部門(department)を全部見つけましょう。

    $users = DB::table('orders')
                    ->select('department', DB::raw('SUM(price) as total_sales'))
                    ->groupBy('department')
                    ->havingRaw('SUM(price) > 2500')
                    ->get();

#### skip / take

クエリから限られた(`LIMIT`)数のレコードを受け取ったり、結果から指定した件数を飛ばしたりするには、`skip`と`take`メソッドを使います。

    $users = DB::table('users')->skip(10)->take(5)->get();

別の方法として、`limit`と`offset`メソッドも使用できます。

    $users = DB::table('users')
                    ->offset(10)
                    ->limit(5)
                    ->get();

<a name="conditional-clauses"></a>
## 条件節

ある条件がtrueの場合の時のみ、クエリへ特定の文を適用したい場合があります。例えば特定の入力値がリクエストに含まれている場合に、`where`文を適用する場合です。`when`メソッドで実現できます。

    $role = $request->input('role');

    $users = DB::table('users')
                    ->when($role, function ($query) use ($role) {
                        return $query->where('role_id', $role);
                    })
                    ->get();


`when`メソッドは、第１引数が`true`の時のみ、指定されたクロージャを実行します。最初の引数が`false`の場合、クロージャを実行しません。

`when`メソッドの第3引数に別のクロージャを渡せます。このクロージャは、最初の引数の評価が`false`であると実行されます。この機能をどう使うかを確認するため、クエリのデフォルトソートを設定してみましょう。

    $sortBy = null;

    $users = DB::table('users')
                    ->when($sortBy, function ($query) use ($sortBy) {
                        return $query->orderBy($sortBy);
                    }, function ($query) {
                        return $query->orderBy('name');
                    })
                    ->get();


<a name="inserts"></a>
## INSERT

クエリビルダは、データベーステーブルにレコードを挿入するための`insert`メソッドを提供しています。`insert`メソッドは挿入するカラム名と値の配列を引数に取ります。

    DB::table('users')->insert(
        ['email' => 'john@example.com', 'votes' => 0]
    );

配列の配列を`insert`に渡して呼び出すことで、テーブルにたくさんのレコードを一度にまとめて挿入できます。

    DB::table('users')->insert([
        ['email' => 'taylor@example.com', 'votes' => 0],
        ['email' => 'dayle@example.com', 'votes' => 0]
    ]);

#### 自動増分ID

テーブルが自動増分IDを持っている場合、`insertGetId`メソッドを使うとレコードを挿入し、そのレコードのIDを返してくれます。

    $id = DB::table('users')->insertGetId(
        ['email' => 'john@example.com', 'votes' => 0]
    );

> {note} PostgreSQLで`insertGetId`メソッドを使う場合、自動増分カラム名は`id`である必要があります。他の「シーケンス」からIDを取得したい場合は、`insertGetId`メソッドの第２引数にシーケンス名を指定してください。

<a name="updates"></a>
## UPDATE

もちろん、データベースへレコードを挿入するだけでなく、存在しているレコードを`update`メソッドで更新することもできます。`update`メソッドは`insert`メソッドと同様に、更新対象のカラムのカラム名と値の配列を引数に受け取ります。更新するクエリを`where`節を使って制約することもできます。

    DB::table('users')
                ->where('id', 1)
                ->update(['votes' => 1]);

<a name="updating-json-columns"></a>
### JSONカラムの更新

JSONカラムを更新する場合、JSONオブジェクトの中の適切なキーへアクセスするために、`->`記法を使ってください。JSONカラムをサポートしているデータベースでのみ、このオペレータをサポートしています。

    DB::table('users')
                ->where('id', 1)
                ->update(['options->enabled' => true]);

<a name="increment-and-decrement"></a>
### 増減分

クエリビルダは指定したカラムの値を増やしたり、減らしたりするのに便利なメソッドも用意しています。これはシンプルな短縮記法で、`update`分で書くのに比べるとより記述的であり、簡潔なインターフェイスを提供しています。

両方のメソッドともに、最小１つ、更新したいカラムを引数に取ります。オプションの第２引数はそのカラムの増減値を指定します。

    DB::table('users')->increment('votes');

    DB::table('users')->increment('votes', 5);

    DB::table('users')->decrement('votes');

    DB::table('users')->decrement('votes', 5);

さらに増減操作と一緒に更新する追加のカラムを指定することもできます。

    DB::table('users')->increment('votes', 1, ['name' => 'John']);

<a name="deletes"></a>
## DELETE

クエリビルダは`delete`メソッドで、テーブルからレコードを削除するためにも使用できます。 `delete`メソッドを呼び出す前に`where`節を追加し、`delete`文を制約することもできます。

    DB::table('users')->delete();

    DB::table('users')->where('votes', '>', 100)->delete();

全レコードを削除し、自動増分のIDを0にリセットするためにテーブルをTRUNCATEしたい場合は、`truncate`メソッドを使います。

    DB::table('users')->truncate();

<a name="pessimistic-locking"></a>
## 悲観的ロック

クエリビルダは、`SELECT`文で「悲観的ロック」を行うための機能をいくつか持っています。SELECT文を実行する間「共有ロック」をかけたい場合は、`sharedLock`メソッドをクエリに指定して下さい。共有ロックはトランザクションがコミットされるまで、SELECTしている行が更新されることを防ぎます。

    DB::table('users')->where('votes', '>', 100)->sharedLock()->get();

もしくは`lockForUpdate`メソッドが使えます。占有ロックをかけることで、レコードを更新したりSELECTするために他の共有ロックが行われるのを防ぎます。

    DB::table('users')->where('votes', '>', 100)->lockForUpdate()->get();
