# Eloquent: APIリソース

- [イントロダクション](#introduction)
- [リソース生成](#generating-resources)
- [概略](#concept-overview)
- [リソース記述](#writing-resources)
    - [データラップ](#data-wrapping)
    - [Pagination](#pagination)
    - [条件付き属性](#conditional-attributes)
    - [条件付きリレーション](#conditional-relationships)
    - [メタデータ追加](#adding-meta-data)
- [リソースレスポンス](#responding-with-resources)

<a name="introduction"></a>
## イントロダクション

API構築時には、Eloquentモデルとアプリケーションユーザーに対して実際に返信するJSONリスポンスとの間に、トランスレーション層を設置することが必要となります。Laravelのリソースクラスは、モデルやモデルコレクションを記述的で簡単にJSONへと変換してくれます。

<a name="generating-resources"></a>
## リソース生成

リソースクラスを生成するに、`make:resource` Artisanコマンドが使用できます。デフォルトでリソースは、アプリケーションの`app/Http/Resources`ディレクトリに設置されます。リソースは、`Illuminate\Http\Resources\Json\Resource`クラスを拡張します。

    php artisan make:resource User

#### コレクションのリソース

個別のモデルのリソースに加え、モデルのコレクションを変換し、返信できるリソースを生成することも可能です。これにより、レスポンスにリンクと、指定したリソースコレクション全体を表す他のメタ情報を含めることができるようになります。

リソースコレクションを生成するには、リソース生成時に`--collection`フラグを使用してください。もしくは、シンプルにリソース名へ`Collection`を含めることで、Laravelへコレクションリソースを生成するように指示できます。コレクションリソースは、`Illuminate\Http\Resources\Json\ResourceCollection`クラスを拡張します。

    php artisan make:resource Users --collection

    php artisan make:resource UserCollection

<a name="concept-overview"></a>
## 概略

> {tip} このセクションは、リソースとリソースコレクションに関する大雑把な概略です。リソースで実現可能な機能とカスタマイズについて深く理解するために、このドキュメントの他の部分も読んでください。

リソースを書く時に指定可能な全オプションの説明の前に、最初はLaravelでリソースがどのように使われるかという点を高度なレベルで確認しておきましょう。リソースクラスは、JSON構造へ変換する必要のある、一つのモデルを表します。たとえば、シンプルな`User`リソースクラスです。

    <?php

    namespace App\Http\Resources;

    use Illuminate\Http\Resources\Json\Resource;

    class User extends Resource
    {
        /**
         * リソースを配列へ変換する
         *
         * @param  \Illuminate\Http\Request
         * @return array
         */
        public function toArray($request)
        {
            return [
                'id' => $this->id,
                'name' => $this->name,
                'email' => $this->email,
                'created_at' => $this->created_at,
                'updated_at' => $this->updated_at,
            ];
        }
    }

レスポンスを送り返す時に、JSONへ変換する必要のある属性の配列を返す、`toArray`メソッドは全リソースクラスで定義します。`$this`変数を使用し、直接モデルのプロパティへアクセスできる点に注目です。これはリソースクラスが、変換するためにアクセスするモデルのプロパティとメソッドを自動的に仲介するからです。リソースが定義できたら、ルートかコントローラから返せます。

    use App\User;
    use App\Http\Resources\User as UserResource;

    Route::get('/user', function () {
        return new UserResource(User::find(1));
    });

### リソースコレクション

ページ付けしたリソースやコレクションを返す場合は、ルートかコントローラの中で、リソースインスタンスを生成する時に、`collection`メソッドを使用します。

    use App\User;
    use App\Http\Resources\User as UserResource;

    Route::get('/user', function () {
        return UserResource::collection(User::all());
    });

当然ながら、これにより返送するコレクションに付加する必要のあるメタデータが、追加されるわけではありません。リソースコレクションレスポンスをカスタマイズしたい場合は、そのコレクションを表すために、専用のリソースを生成してください。

    php artisan make:resource UserCollection

リソースコレクションを生成したら、レスポンスに含めたいメタデータを簡単に定義できます。

    <?php

    namespace App\Http\Resources;

    use Illuminate\Http\Resources\Json\ResourceCollection;

    class UserCollection extends ResourceCollection
    {
        /**
         * リソースコレクションを配列へ変換
         *
         * @param  \Illuminate\Http\Request
         * @return array
         */
        public function toArray($request)
        {
            return [
                'data' => $this->collection,
                'links' => [
                    'self' => 'link-value',
                ],
            ];
        }
    }

リソースコレクションを定義したら、ルートかコントローラから返してください。

    use App\User;
    use App\Http\Resources\UserCollection;

    Route::get('/users', function () {
        return new UserCollection(User::all());
    });

<a name="writing-resources"></a>
## リソース記述

> {tip} [概略](#concept-overview)をまだ読んでいないのなら、ドキュメントを読み進める前に目を通しておくことを強くおすすめします。

リソースの本質は、シンプルです。特定のモデルを配列に変換する必要があるだけです。そのため、APIフレンドリーな配列としてユーザーへ送り返せるように、モデルの属性を変換するための`toArray`メソッドをリソースは持っています。

    <?php

    namespace App\Http\Resources;

    use Illuminate\Http\Resources\Json\Resource;

    class User extends Resource
    {
        /**
         * リソースを配列へ変換
         *
         * @param  \Illuminate\Http\Request
         * @return array
         */
        public function toArray($request)
        {
            return [
                'id' => $this->id,
                'name' => $this->name,
                'email' => $this->email,
                'created_at' => $this->created_at,
                'updated_at' => $this->updated_at,
            ];
        }
    }

リソースを定義したら、ルートかコントローラから、直接返してください。

    use App\User;
    use App\Http\Resources\User as UserResource;

    Route::get('/user', function () {
        return new UserResource(User::find(1));
    });

#### リレーション

レスポンスへ関連するリソースを含めたい場合は、`toArray`メソッドで返す配列に追加するだけです。以下の例では、`Post`リソースの`collection`メソッドを使用し、ユーザーのブログポストをリソースレスポンスへ追加しています。

    /**
     * リソースを配列へ変換
     *
     * @param  \Illuminate\Http\Request
     * @return array
     */
    public function toArray($request)
    {
        return [
            'id' => $this->id,
            'name' => $this->name,
            'email' => $this->email,
            'posts' => Post::collection($this->posts),
            'created_at' => $this->created_at,
            'updated_at' => $this->updated_at,
        ];
    }

> {tip} 既にロードされている場合のみ、リレーションを含めたい場合は、[条件付きリレーション](#conditional-relationships)のドキュメントを参照してください。

#### リソースコレクション

リソースは一つのモデルを配列へ変換するのに対し、リソースコレクションはモデルのコレクションを配列へ変換します。モデルタイプそれぞれに対し、リソースコレクションを絶対に定義する必要があるわけではありません。すべてのリソースは、簡単に「アドホック」なリソースコレクションを生成するために、`collection`メソッドを提供しています。

    use App\User;
    use App\Http\Resources\User as UserResource;

    Route::get('/user', function () {
        return UserResource::collection(User::all());
    });

しかしながら、コレクションと一緒に返すメタデータをカスタマイズする必要がある場合は、リソースコレクションを定義する必要があります。

    <?php

    namespace App\Http\Resources;

    use Illuminate\Http\Resources\Json\ResourceCollection;

    class UserCollection extends ResourceCollection
    {
        /**
         * リソースコレクションを配列へ変換
         *
         * @param  \Illuminate\Http\Request
         * @return array
         */
        public function toArray($request)
        {
            return [
                'data' => $this->collection,
                'links' => [
                    'self' => 'link-value',
                ],
            ];
        }
    }

１モデルを扱うリソースと同様に、リソースコレクションもルートやコントローラから直接返してください。

    use App\User;
    use App\Http\Resources\UserCollection;

    Route::get('/users', function () {
        return new UserCollection(User::all());
    });

<a name="data-wrapping"></a>
### データラップ

デフォルトではリソースレスポンスがJSONに変換される際、一番外側のリソースが`data`キー下にラップされます。たとえば、典型的なリソースコレクションのレスポンスは、次のようになるでしょう。

    {
        "data": [
            {
                "id": 1,
                "name": "Eladio Schroeder Sr.",
                "email": "therese28@example.com",
            },
            {
                "id": 2,
                "name": "Liliana Mayert",
                "email": "evandervort@example.com",
            }
        ]
    }

一番外部のリソースをラップしないようにしたい場合は、ベースのリソースクラスに対し、`withoutWrapping`メソッドを使用してください。通常、このメソッドはアプリケーションに対するリクエストごとにロードされる、`AppServiceProvider`か、もしくは他の[サービスプロバイダ](/docs/{{version}}/providers)から呼び出します。

    <?php

    namespace App\Providers;

    use Illuminate\Support\ServiceProvider;
    use Illuminate\Http\Resources\Json\Resource;

    class AppServiceProvider extends ServiceProvider
    {
        /**
         * サービスの初期起動後に、登録内容を処理
         *
         * @return void
         */
        public function boot()
        {
            Resource::withoutWrapping();
        }

        /**
         * コンテナに結合を登録
         *
         * @return void
         */
        public function register()
        {
            //
        }
    }

> {note} `withoutWrapping`メソッドは、最も外側のレスポンスだけに影響を与えます。リソースコレクションに皆さんが自分で追加した`data`キーは、削除しません。

### ネストしたリソースのラップ

リソースのリレーションをどのようにラップするかは、完全に自由です。ネスト状態に拘わらず、`data`キーの中に全リソースコレクションをラップしたい場合は、リソースそれぞれに対するリソースコレクションを定義し、`data`キーにコレクションを含めて返す必要があります。

それにより、二重の`data`キーで最も外側のリソースがラップされてしまうのではないかと、疑うのは当然です。心配ありません。Laravelは決してリソースを間違って二重にラップしたりしません。変換するリソースコレクションのネストレベルについて、心配する必要はありません。

    <?php

    namespace App\Http\Resources;

    use Illuminate\Http\Resources\Json\ResourceCollection;

    class CommentsCollection extends ResourceCollection
    {
        /**
         * リソースコレクションを配列へ変換
         *
         * @param  \Illuminate\Http\Request
         * @return array
         */
        public function toArray($request)
        {
            return ['data' => $this->collection];
        }
    }

### データラップとペジネーション

リソースレスポンスの中で、ページ付けしたコレクションを返す場合、`withoutWrapping`メソッドが呼び出されていても、Laravelはリソースデータを`data`キーでラップします。なぜなら、ページ付けしたレスポンスは、ペジネータの状態を含めた`meta`と`links`キーを常に含めるからです。

    {
        "data": [
            {
                "id": 1,
                "name": "Eladio Schroeder Sr.",
                "email": "therese28@example.com",
            },
            {
                "id": 2,
                "name": "Liliana Mayert",
                "email": "evandervort@example.com",
            }
        ],
        "links":{
            "first": "http://example.com/pagination?page=1",
            "last": "http://example.com/pagination?page=1",
            "prev": null,
            "next": null
        },
        "meta":{
            "current_page": 1,
            "from": 1,
            "last_page": 1,
            "path": "http://example.com/pagination",
            "per_page": 15,
            "to": 10,
            "total": 10
        }
    }

<a name="pagination"></a>
### ペジネーション

常にペジネータインスタンスをリソースの`collection`メソッドや、カスタムリソースコレクションへ渡せます。

    use App\User;
    use App\Http\Resources\UserCollection;

    Route::get('/users', function () {
        return new UserCollection(User::all());
    });

ページ付けしたレスポンスは常に、ペジネータの状態を含む`meta`と`links`キーを持っています。

    {
        "data": [
            {
                "id": 1,
                "name": "Eladio Schroeder Sr.",
                "email": "therese28@example.com",
            },
            {
                "id": 2,
                "name": "Liliana Mayert",
                "email": "evandervort@example.com",
            }
        ],
        "links":{
            "first": "http://example.com/pagination?page=1",
            "last": "http://example.com/pagination?page=1",
            "prev": null,
            "next": null
        },
        "meta":{
            "current_page": 1,
            "from": 1,
            "last_page": 1,
            "path": "http://example.com/pagination",
            "per_page": 15,
            "to": 10,
            "total": 10
        }
    }

<a name="conditional-attributes"></a>
### 条件付き属性

条件が一致する場合のみ、リソースレスポンスへ属性を含めたいこともあります。たとえば、現在のユーザーが"administrator"の場合のみ、ある値を含めたいときです。こうした状況で役に立つ様々なヘルパメソッドをLaravelは提供しています。`when`メソッドは条件により、リソースレスポンスへ属性を追加する場合に使用します。

    /**
     * リソースを配列へ変換
     *
     * @param  \Illuminate\Http\Request
     * @return array
     */
    public function toArray($request)
    {
        return [
            'id' => $this->id,
            'name' => $this->name,
            'email' => $this->email,
            'secret' => $this->when($this->isAdmin(), 'secret-value'),
            'created_at' => $this->created_at,
            'updated_at' => $this->updated_at,
        ];
    }

この場合、`$this->isAdmin()`メソッドが`true`を返す場合のみ、最終的なリソースレスポンスに`secret`キーが返されます。メソッドが`false`の場合、クライアントへ送り返される前に、リソースレスポンスから`secret`キーは完全に削除されます。`when`メソッドにより、配列の構築時に条件文に頼らずとも、リソースを記述的に定義できます。

`when`メソッドは、第２引数にクロージャを引き受け、指定した条件が`ture`の場合のみ、結果の値を算出することもできます。

    'secret' => $this->when($this->isAdmin(), function () {
        return 'secret-value';
    }),

> {tip} リソースで呼び出されるメソッドは、元のモデルインスタンスへの呼び出しを仲介することを思い出してください。ですからこの場合、`isAdmin`メソッドはリソースを提供するオリジナルのEloquentモデルへと仲介されます。

#### 条件付き属性のマージ

リソースレスポンスへ同じ条件にもとづいて、多くの属性を含めたい場合もあります。この場合、指定した条件が`ture`の場合のみ、レスポンスへ属性を組み入れる`mergeWhen`メソッドを使用します。

    /**
     * リソースを配列へ変換
     *
     * @param  \Illuminate\Http\Request
     * @return array
     */
    public function toArray($request)
    {
        return [
            'id' => $this->id,
            'name' => $this->name,
            'email' => $this->email,
            $this->mergeWhen($this->isAdmin(), [
                'first-secret' => 'value',
                'second-secret' => 'value',
            ]),
            'created_at' => $this->created_at,
            'updated_at' => $this->updated_at,
        ];
    }

このメソッドでも、指定した条件が`false`の場合、利用者へ送り返される前に、リソースレスポンスから属性は完全に取り除かれます。

> {note} `mergeWhen`メソッドは、文字列と数値のキーが混ざっている配列の中では、使用しないでください。さらに、順番に並んでいない数値キーの配列でも、使用しないでください。

<a name="conditional-relationships"></a>
### 条件付きリレーション

条件によりロードする属性に付け加え、リレーションがモデルにロードされているかに基づいて、リソースレスポンスへリレーションを条件付きで含めることもできます。これにより、どのリレーションをモデルにロードさせるかをコントローラで決め、実際にロードしている場合のみ、リソースで含めることが簡単に実現できます。

最終的に、これによりリソースでの「N＋１」問題を防ぐことができます。リレーションを条件付きでロードするには、`whenLoaded`メソッドを使います。不必要なリレーションのロードを防ぐために、このメソッドはリレーション自身の代わりに、リレーションの名前を引数に取ります。

    /**
     * リソースを配列へ変換
     *
     * @param  \Illuminate\Http\Request
     * @return array
     */
    public function toArray($request)
    {
        return [
            'id' => $this->id,
            'name' => $this->name,
            'email' => $this->email,
            'posts' => Post::collection($this->whenLoaded('posts')),
            'created_at' => $this->created_at,
            'updated_at' => $this->updated_at,
        ];
    }

この例の場合、リレーションがロードされていない場合、`posts`キーは利用者へ送り返される前に、レスポンスから完全に取り除かれます。

#### 条件付きピボット情報

リソースレスポンスへ条件付きでリレーション情報を含める機能に付け加え、`whenPivotLoaded`メソッドを使用し、多対多リレーションの中間テーブルからデータを含めることもできます。`whenPivotLoaded`メソッドは、第１引数に中間テーブルの名前を引き受けます。第２引数には、ピボット情報がそのモデルに対し利用可能な場合の返却値を定義するクロージャを指定します。

    /**
     * リソースを配列へ変換
     *
     * @param  \Illuminate\Http\Request
     * @return array
     */
    public function toArray($request)
    {
        return [
            'id' => $this->id,
            'name' => $this->name,
            'expires_at' => $this->whenPivotLoaded('role_users', function () {
                return $this->pivot->expires_at;
            }),
        ];
    }

<a name="adding-meta-data"></a>
### メタデータ追加

いつくかのJSON API基準では、リソースとリソースコレクションレスポンスへ、追加のメタデータを要求しています。これらには、リソースへの`link`と考えられる情報や、関連するリソース、リソース自体のメタデータなどがよく含まれます。リソースに関する追加のメタデータを返す必要がある場合は、`toArray`メソッドに含めるだけです。たとえば、リソースコレクションを変換する時に、`link`情報を含めるには次のようにします。

    /**
     * リソースを配列へ変換
     *
     * @param  \Illuminate\Http\Request
     * @return array
     */
    public function toArray($request)
    {
        return [
            'data' => $this->collection,
            'links' => [
                'self' => 'link-value',
            ],
        ];
    }

追加のメタデータをリソースから返す場合、ページ付けレスポンスを返す場合に、Laravelが自動的に付け加える`links`や`meta`キーを意図せずオーバーライドしてしまう心配はありません。追加の`links`定義は、ペジネータが提供するリンク情報にマージされるだけです。

#### トップレベルメタデータ

一番外側のリソースが返される場合にのみ、特定のメタデータをリソースレスポンスへ含めたい場合があります。典型的な例は、レスポンス全体のメタ情報です。こうしたメタデータを定義するには、リソースクラスへ`with`メソッドを追加します。このメソッドは、一番外側のリソースを返す場合のみ、リソースレスポンスへ含めるメタデータの配列を返します。

    <?php

    namespace App\Http\Resources;

    use Illuminate\Http\Resources\Json\ResourceCollection;

    class UserCollection extends ResourceCollection
    {
        /**
         * リソースコレクションを配列へ変換
         *
         * @param  \Illuminate\Http\Request
         * @return array
         */
        public function toArray($request)
        {
            return parent::toArray($request);
        }

        /**
         * リソース配列と共に返すべき、追加データの取得
         *
         * @param \Illuminate\Http\Request  $request
         * @return array
         */
        public function with($request)
        {
            return [
                'meta' => [
                    'key' => 'value',
                ],
            ];
        }
    }

#### リソース構築時のメタデータ追加

ルートやコントローラの中で、リソースインスタンスを構築する時に、トップレベルのデータを追加することもできます。全リソースの中で利用可能な`additional`メソッドは、リソースレスポンスへ含めるべき追加データの配列を引数に取ります。

    return (new UserCollection(User::all()->load('roles')))
                    ->additional(['meta' => [
                        'key' => 'value',
                    ]]);

<a name="resource-responses"></a>
## リソースレスポンス

既に目にしてきた通りに、リソースはルートかコントローラから直接返されます。

    use App\User;
    use App\Http\Resources\User as UserResource;

    Route::get('/user', function () {
        return new UserResource(User::find(1));
    });

しかし、利用者へ送信する前に、HTTPレスポンスをカスタマイズする必要が時々あります。最初に、リソースに対して`response`メソッドをチェーンしてください。このメソッドは、`Illuminate\Http\Response`インスタンスを返しますので、レスポンスヘッダーを完全にコントロールできます。

    use App\User;
    use App\Http\Resources\User as UserResource;

    Route::get('/user', function () {
        return (new UserResource(User::find(1)))
                    ->response()
                    ->header('X-Value', 'True');
    });

もしくは、`withResponse`メソッドをレスポンス自身の中で定義することもできます。このメソッドはレスポンスの中で一番外側のリソースとして返す場合に呼び出されます。

    <?php

    namespace App\Http\Resources;

    use Illuminate\Http\Resources\Json\Resource;

    class User extends Resource
    {
        /**
         * リソースを配列へ変換
         *
         * @param  \Illuminate\Http\Request
         * @return array
         */
        public function toArray($request)
        {
            return [
                'id' => $this->id,
            ];
        }

        /**
         * リソースに対して送信するレスポンスのカスタマイズ
         *
         * @param  \Illuminate\Http\Request
         * @param  \Illuminate\Http\Response
         * @return void
         */
        public function withResponse($request, $response)
        {
            $response->header('X-Value', 'True');
        }
    }
