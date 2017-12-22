# Laravel Scout

- [イントロダクション](#introduction)
- [インストール](#installation)
    - [キュー](#queueing)
    - [ドライバの事前要件](#driver-prerequisites)
- [設定](#configuration)
    - [モデルインデックスの設定](#configuring-model-indexes)
    - [検索可能データの設定](#configuring-searchable-data)
- [インデックス](#indexing)
    - [バッチ取り込み](#batch-import)
    - [レコード追加](#adding-records)
    - [レコード更新](#updating-records)
    - [レコード削除](#removing-records)
    - [インデックスの一時停止](#pausing-indexing)
- [検索](#searching)
    - [Where節](#where-clauses)
    - [ペジネーション](#pagination)
- [カスタムエンジン](#custom-engines)

<a name="introduction"></a>
## イントロダクション

Laravel Scout（スカウト、斥候）は、[Eloquentモデル](/docs/{{version}}/eloquent)へ、シンプルなドライバベースのフルテキストサーチを提供します。モデルオブサーバを使い、Scoutは検索インデックスを自動的にEloquentレコードと同期します。

現在、Scoutは[Algolia](https://www.algolia.com/)ドライバを用意しています。カスタムドライバは簡単に書けますので、独自の検索を実装し、Scoutを拡張できます。

<a name="installation"></a>
## インストール

最初に、Composerパッケージマネージャを使い、Scoutをインストールします。

    composer require laravel/scout

Scoutをインストールしたら、`vendor:publish` Artisanコマンドを使用し、Scout設定ファイルを公開します。このコマンドは、`config`ディレクトリ下に`scout.php`設定ファイルを公開します。

    php artisan vendor:publish --provider="Laravel\Scout\ScoutServiceProvider"

最後に、検索可能にしたいモデルへ、`Laravel\Scout\Searchable`トレイトを追加します。このトレイトはモデルオブザーバを登録し、サーチドライバとモデルの同期を取り続けます。

    <?php

    namespace App;

    use Laravel\Scout\Searchable;
    use Illuminate\Database\Eloquent\Model;

    class Post extends Model
    {
        use Searchable;
    }

<a name="queueing"></a>
### キュー

Scoutを厳格（リアルタイム）に利用する必要が無いのであれば、このライブラリを使用する前に[キュードライバ](/docs/{{version}}/queues)の設定を考えてみるべきでしょう。キューワーカの実行により、モデルの情報を検索インデックスに同期する全操作をキューイングでき、アプリケーションのWebインターフェイスのレスポンス時間を改善できるでしょう。

キュードライバを設定したら、`config/scout.php`設定ファイルの`queue`オプション値を`true`に設定してください。

    'queue' => true,

<a name="driver-prerequisites"></a>
### ドライバの事前要件

#### Algolia

Algoliaドライバを使用する場合、Algolia `id`と`secret`接続情報を`config/scout.php`設定ファイルで設定する必要があります。接続情報を設定し終えたら、Algolia PHP SDKをComposerパッケージマネージャで、インストールする必要があります。

    composer require algolia/algoliasearch-client-php

<a name="configuration"></a>
## 設定

<a name="configuring-model-indexes"></a>
### モデルインデックスの設定

各Eloquentモデルは、検索可能レコードすべてを含む、指定された検索「インデックス」と同期されます。言い換えれば、各インデックスはMySQLテーブルのようなものであると、考えられます。デフォルトで、各モデルはそのモデルの典型的な「テーブル」名に一致するインデックスへ保存されます。通常、モデルの複数形ですが、モデルの`searchableAs`メソッドをオーバーライドすることで、このモデルのインデックスを自由にカスタマイズ可能です。

    <?php

    namespace App;

    use Laravel\Scout\Searchable;
    use Illuminate\Database\Eloquent\Model;

    class Post extends Model
    {
        use Searchable;

        /**
         * モデルのインデックス名取得
         *
         * @return string
         */
        public function searchableAs()
        {
            return 'posts_index';
        }
    }

<a name="configuring-searchable-data"></a>
### 検索可能データの設定

デフォルトでは、指定されたモデルの`toArray`形態全体が、検索インデックスへ保存されます。検索インデックスと同期するデータをカスタマイズしたい場合は、そのモデルの`toSearchableArray`メソッドをオーバーライドできます。

    <?php

    namespace App;

    use Laravel\Scout\Searchable;
    use Illuminate\Database\Eloquent\Model;

    class Post extends Model
    {
        use Searchable;

        /**
         * モデルのインデックス可能なデータ配列の取得
         *
         * @return array
         */
        public function toSearchableArray()
        {
            $array = $this->toArray();

            // 配列のカスタマイズ…

            return $array;
        }
    }

<a name="indexing"></a>
## インデックス

<a name="batch-import"></a>
### バッチ取り込み

既存プロジェクトにScoutをインストールする場合、検索ドライバへ取り込むために必要なデータベースレコードは、既に存在しています。Scoutは既存の全レコードを検索インデックスへ取り込むために使用する、`import` Artisanコマンドを提供しています。

    php artisan scout:import "App\Post"

<a name="adding-records"></a>
### レコード追加

モデルに`Laravel\Scout\Searchable`トレイトを追加したら、必要なのはモデルインスタンスを`save`することです。これにより自動的に検索インデックスへ追加されます。Scoutで[キューを使用する](#queueing)設定にしている場合は、この操作はキューワーカにより、バックグランドで実行されます。

    $order = new App\Order;

    // ...

    $order->save();

#### クエリによる追加

Eloquentクエリにより、検索インデックスへモデルのコレクションを追加したい場合は、Eloquentクエリに`searchable`メソッドをチェーンします。`searchable`メソッドは、クエリの[結果をチャンクへ分割](/docs/{{version}}/eloquent#chunking-results)し、レコードを検索エンジンへ追加します。この場合も、Scoutでキューを使用する設定をしていれば、キューワーカが全チャンクをバックグランドで追加します。

    // Eloquentクエリにより追加
    App\Order::where('price', '>', 100)->searchable();

    // リレーションにより、レコードを追加することもできる
    $user->orders()->searchable();

    // コレクションにより、追加することもできる
    $orders->searchable();

`searchable`メソッドは"upsert(update+insert)"操作と考えられます。言い換えれば、モデルレコードがインデックスへ既に存在していれば、更新されます。検索エンジンに存在していなければ、インデックスへ追加されます。

<a name="updating-records"></a>
### レコード更新

検索可能モデルを更新するには、モデルインスタンスのプロパティを更新し、`save`でモデルをデータベースへ保存します。Scoutは自動的に変更を検索インデックスへ保存します。

    $order = App\Order::find(1);

    // 注文を更新…

    $order->save();

モデルのコレクションを更新するためにも、Eloquentクエリの`searchable`メソッドを使用します。検索エンジンにモデルが存在していない場合は、作成します。

    // Eloquentクエリによる更新
    App\Order::where('price', '>', 100)->searchable();

    // リレーションによる更新も可能
    $user->orders()->searchable();

    // コレクションによる更新も可能
    $orders->searchable();

<a name="removing-records"></a>
### レコード削除

インデックスからレコードを削除するには、データベースからモデルを`delete`で削除するだけです。この形態による削除は、モデルの[ソフト削除](/docs/{{version}}/eloquent#soft-deleting)と互換性があります。

    $order = App\Order::find(1);

    $order->delete();

レコードを削除する前に、モデルを取得したくない場合は、Eloquentクエリインスタンスかコレクションに対し、`unsearchable`メソッドを使用します。

    // Eloquentクエリによる削除
    App\Order::where('price', '>', 100)->unsearchable();

    // リレーションによる削除も可能
    $user->orders()->unsearchable();

    // コレクションによる削除も可能
    $orders->unsearchable();

<a name="pausing-indexing"></a>
### インデックスの一時停止

Eloquentモデルをバッチ処理するが、検索インデックスへモデルデータを同期したくない場合も時々あります。`withoutSyncingToSearch`メソッドを使用することで可能です。このメソッドは、即時に実行されるコールバックを１つ引数に取ります。コールバック中のモデル操作は、インデックスへ同期されることはありません。

    App\Order::withoutSyncingToSearch(function () {
        // モデルアクションの実行…
    });

<a name="searching"></a>
## 検索

`search`メソッドにより、モデルの検索を開始しましょう。`search`メソッドはモデルを検索するために使用する文字列だけを引数に指定します。`get`メソッドを検索クエリにチェーンし、指定した検索クエリに一致するEloquentモデルを取得できます。

    $orders = App\Order::search('Star Trek')->get();

Scoutの検索ではEloquentモデルのコレクションが返されるため、ルートやコントローラから直接結果を返せば、自動的にJSONへ変換されます。

    use Illuminate\Http\Request;

    Route::get('/search', function (Request $request) {
        return App\Order::search($request->search)->get();
    });

Eloquentモデルにより変換される前の、結果をそのまま取得したい場合は、`raw`メソッドを使用してください。

    $orders = App\Order::search('Star Trek')->raw();

検索クエリは通常、モデルの[`searchableAs`](#configuring-model-indexes)メソッドに指定されたインデックスを使い実行されます。しかし、その代わりに検索に使用するカスタムインデックスを`within`メソッドで使用することもできます。

    $orders = App\Order::search('Star Trek')
        ->within('tv_shows_popularity_desc')
        ->get();

<a name="where-clauses"></a>
### Where節

Scoutは検索クエリに対して"WHERE"節を単に追加する方法も提供しています。現在、この節としてサポートしているのは、基本的な数値の一致を確認することだけで、主にIDにより検索クエリを絞り込むために使用します。検索インデックスはリレーショナル・データベースではないため、より上級の"WHERE"節は現在サポートしていません。

    $orders = App\Order::search('Star Trek')->where('user_id', 1)->get();

<a name="pagination"></a>
### ペジネーション

コレクションの取得に付け加え、検索結果を`paginate`メソッドでページづけできます。このメソッドは、`Paginator`インスタンスを返しますので、[Eloquentクエリのペジネーション](/docs/{{version}}/pagination)と同様に取り扱えます。

    $orders = App\Order::search('Star Trek')->paginate();

`paginate`メソッドの第１引数として、各ページごとに取得したいモデル数を指定します。

    $orders = App\Order::search('Star Trek')->paginate(15);

結果が取得できたら、通常のEloquentクエリのペジネーションと同様に、結果を表示し、[Blade](/docs/{{version}}/blade)を使用してページリンクをレンダーできます。

    <div class="container">
        @foreach ($orders as $order)
            {{ $order->price }}
        @endforeach
    </div>

    {{ $orders->links() }}

<a name="custom-engines"></a>
## カスタムエンジン

#### エンジンのプログラミング

組み込みのScout検索エンジンがニーズに合わない場合、独自のカスタムエンジンを書き、Scoutへ登録してください。エンジンは、`Laravel\Scout\Engines\Engine`抽象クラスを拡張してください。この抽象クラスは、カスタムエンジンが実装する必要のある、７つのメソッドを持っています。

    use Laravel\Scout\Builder;

    abstract public function update($models);
    abstract public function delete($models);
    abstract public function search(Builder $builder);
    abstract public function paginate(Builder $builder, $perPage, $page);
    abstract public function mapIds($results);
    abstract public function map($results, $model);
    abstract public function getTotalCount($results);

これらのメソッドの実装をレビューするために、`Laravel\Scout\Engines\AlgoliaEngine`クラスが役に立つでしょう。このクラスは独自エンジンで、各メソッドをどのように実装すればよいかの、良い取り掛かりになるでしょう。

#### エンジンの登録

カスタムエンジンを書いたら、Scoutエンジンマネージャの`extend`メソッドを使用し、Scoutへ登録します。`AppServiceProvider`かアプリケーションで使用している他のサービスプロバイダの`boot`メソッドで、`extend`メソッドを呼び出してください。たとえば、`MySqlSearchEngine`を書いた場合、次のように登録します。

    use Laravel\Scout\EngineManager;

    /**
     * 全アプリケーションサービスの登録
     *
     * @return void
     */
    public function boot()
    {
        resolve(EngineManager::class)->extend('mysql', function () {
            return new MySqlSearchEngine;
        });
    }

エンジンが登録できたら、Scoutのデフォルト`driver`として、`config/scout.php`設定ファイルで設定します。

    'driver' => 'mysql',
