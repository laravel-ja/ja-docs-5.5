# ヘルパ

- [イントロダクション](#introduction)
- [使用可能な関数](#available-methods)

<a name="introduction"></a>
## イントロダクション

Laravelは様々な、グローバル「ヘルパ」PHP関数を用意しています。これらの多くはフレームワーク自身で使用されています。便利なものが見つかれば、皆さんのアプリケーションでも大いに活用してください。

<a name="available-methods"></a>
## 使用可能な関数

<style>
    .collection-method-list > p {
        column-count: 3; -moz-column-count: 3; -webkit-column-count: 3;
        column-gap: 2em; -moz-column-gap: 2em; -webkit-column-gap: 2em;
    }

    .collection-method-list a {
        display: block;
    }
</style>

### 配列

<div class="collection-method-list" markdown="1">

[array_add](#method-array-add)
[array_collapse](#method-array-collapse)
[array_divide](#method-array-divide)
[array_dot](#method-array-dot)
[array_except](#method-array-except)
[array_first](#method-array-first)
[array_flatten](#method-array-flatten)
[array_forget](#method-array-forget)
[array_get](#method-array-get)
[array_has](#method-array-has)
[array_last](#method-array-last)
[array_only](#method-array-only)
[array_pluck](#method-array-pluck)
[array_prepend](#method-array-prepend)
[array_pull](#method-array-pull)
[array_set](#method-array-set)
[array_sort](#method-array-sort)
[array_sort_recursive](#method-array-sort-recursive)
[array_where](#method-array-where)
[array_wrap](#method-array-wrap)
[head](#method-head)
[last](#method-last)
</div>

### パス

<div class="collection-method-list" markdown="1">

[app_path](#method-app-path)
[base_path](#method-base-path)
[config_path](#method-config-path)
[database_path](#method-database-path)
[mix](#method-mix)
[public_path](#method-public-path)
[resource_path](#method-resource-path)
[storage_path](#method-storage-path)

</div>

### 文字列

<div class="collection-method-list" markdown="1">

[camel_case](#method-camel-case)
[class_basename](#method-class-basename)
[e](#method-e)
[ends_with](#method-ends-with)
[kebab_case](#method-kebab-case)
[snake_case](#method-snake-case)
[str_limit](#method-str-limit)
[starts_with](#method-starts-with)
[str_after](#method-str-after)
[str_before](#method-str-before)
[str_contains](#method-str-contains)
[str_finish](#method-str-finish)
[str_is](#method-str-is)
[str_plural](#method-str-plural)
[str_random](#method-str-random)
[str_singular](#method-str-singular)
[str_slug](#method-str-slug)
[studly_case](#method-studly-case)
[title_case](#method-title-case)
[trans](#method-trans)
[trans_choice](#method-trans-choice)

</div>

### URL

<div class="collection-method-list" markdown="1">

[action](#method-action)
[asset](#method-asset)
[secure_asset](#method-secure-asset)
[route](#method-route)
[secure_url](#method-secure-url)
[url](#method-url)

</div>

### その他

<div class="collection-method-list" markdown="1">

[abort](#method-abort)
[abort_if](#method-abort-if)
[abort_unless](#method-abort-unless)
[auth](#method-auth)
[back](#method-back)
[bcrypt](#method-bcrypt)
[cache](#method-cache)
[collect](#method-collect)
[config](#method-config)
[csrf_field](#method-csrf-field)
[csrf_token](#method-csrf-token)
[dd](#method-dd)
[dispatch](#method-dispatch)
[env](#method-env)
[event](#method-event)
[factory](#method-factory)
[info](#method-info)
[logger](#method-logger)
[method_field](#method-method-field)
[now](#method-now)
[old](#method-old)
[optional](#method-optional)
[redirect](#method-redirect)
[report](#method-report)
[request](#method-request)
[rescue](#method-rescue)
[response](#method-response)
[retry](#method-retry)
[session](#method-session)
[tap](#method-tap)
[today](#method-today)
[value](#method-value)
[view](#method-view)

</div>

<a name="method-listing"></a>
## メソッド一覧

<style>
    #collection-method code {
        font-size: 14px;
    }

    #collection-method:not(.first-collection-method) {
        margin-top: 50px;
    }
</style>

<a name="arrays"></a>
## 配列

<a name="method-array-add"></a>
#### `array_add()` {#collection-method .first-collection-method}

`array_add`関数は指定されたキー／値のペアをそのキーが存在していない場合、配列に追加します。

    $array = array_add(['name' => 'Desk'], 'price', 100);

    // ['name' => 'Desk', 'price' => 100]

<a name="method-array-collapse"></a>
#### `array_collapse()` {#collection-method}

`array_collapse`関数は配列の配列を一次元の配列へ展開します。

    $array = array_collapse([[1, 2, 3], [4, 5, 6], [7, 8, 9]]);

    // [1, 2, 3, 4, 5, 6, 7, 8, 9]

<a name="method-array-divide"></a>
#### `array_divide()` {#collection-method}

`array_divide`関数は２つの配列をリターンします。一つはオリジナル配列のキー、もう一方は値です。

    list($keys, $values) = array_divide(['name' => 'Desk']);

    // $keys: ['name']

    // $values: ['Desk']

<a name="method-array-dot"></a>
#### `array_dot()` {#collection-method}

`array_dot`関数は多次元配列を「ドット」記法で深さを表した一次元配列に変換します。

    $array = array_dot(['foo' => ['bar' => 'baz']]);

    // ['foo.bar' => 'baz'];

<a name="method-array-except"></a>
#### `array_except()` {#collection-method}

`array_except`関数は指定されたキー／値ペアを配列から削除します。

    $array = ['name' => 'Desk', 'price' => 100];

    $array = array_except($array, ['price']);

    // ['name' => 'Desk']

<a name="method-array-first"></a>
#### `array_first()` {#collection-method}

`array_first`関数は指定されたテストにパスした最初の要素を返します。

    $array = [100, 200, 300];

    $value = array_first($array, function ($value, $key) {
        return $value >= 150;
    });

    // 200

デフォルト値を３つ目の引数で指定することもできます。この値はテストでどの値もテストにパスしない場合に返されます。

    $value = array_first($array, $callback, $default);

<a name="method-array-flatten"></a>
#### `array_flatten()` {#collection-method}

`array_flatten`関数は、多次元配列を一次元配列へ変換します。

    $array = ['name' => 'Joe', 'languages' => ['PHP', 'Ruby']];

    $array = array_flatten($array);

    // ['Joe', 'PHP', 'Ruby'];

<a name="method-array-forget"></a>
#### `array_forget()` {#collection-method}

`array_forget`関数は「ドット記法」で指定されたキーと値のペアを深くネストされた配列から取り除きます。

    $array = ['products' => ['desk' => ['price' => 100]]];

    array_forget($array, 'products.desk');

    // ['products' => []]

<a name="method-array-get"></a>
#### `array_get()` {#collection-method}

`array_get`関数は指定された値を「ドット」記法で指定された値を深くネストされた配列から取得します。

    $array = ['products' => ['desk' => ['price' => 100]]];

    $value = array_get($array, 'products.desk');

    // ['price' => 100]

`array_get`関数は、指定したキーが存在しない場合に返されるデフォルト値を指定できます。

    $value = array_get($array, 'names.john', 'default');

<a name="method-array-has"></a>
#### `array_has()` {#collection-method}

`array_has`関数は、「ドット」記法で指定されたアイテムの存在をチェックします。

    $array = ['product' => ['name' => 'desk', 'price' => 100]];

    $hasItem = array_has($array, 'product.name');

    // true

    $hasItems = array_has($array, ['product.price', 'product.discount']);

    // false

<a name="method-array-last"></a>
#### `array_last()` {#collection-method}

`array_last`関数は、テストでパスした最後の配列要素を返します。

    $array = [100, 200, 300, 110];

    $value = array_last($array, function ($value, $key) {
        return $value >= 150;
    });

    // 300

メソッドの第３引数には、デフォルト値を渡します。テストでパスする値がない場合に、返されます。

    $value = array_last($array, $callback, $default);

<a name="method-array-only"></a>
#### `array_only()` {#collection-method}

`array_only`関数は配列中の指定されたキー／値ペアのアイテムのみを返します。

    $array = ['name' => 'Desk', 'price' => 100, 'orders' => 10];

    $array = array_only($array, ['name', 'price']);

    // ['name' => 'Desk', 'price' => 100]

<a name="method-array-pluck"></a>
#### `array_pluck()` {#collection-method}

`array_pluck`関数は配列から指定されたキー／値ペアのリストを取得します。

    $array = [
        ['developer' => ['id' => 1, 'name' => 'Taylor']],
        ['developer' => ['id' => 2, 'name' => 'Abigail']],
    ];

    $array = array_pluck($array, 'developer.name');

    // ['Taylor', 'Abigail'];

さらに、結果のリストのキー項目も指定できます。

    $array = array_pluck($array, 'developer.name', 'developer.id');

    // [1 => 'Taylor', 2 => 'Abigail'];

<a name="method-array-prepend"></a>
#### `array_prepend()` {#collection-method}

`array_prepend`関数は配列の先頭にアイテムを追加します。

    $array = ['one', 'two', 'three', 'four'];

    $array = array_prepend($array, 'zero');

    // $array: ['zero', 'one', 'two', 'three', 'four']

必要であれば、値に対するキーを指定できます。

    $array = ['price' => 100];

    $array = array_prepend($array, 'Desk', 'name');

    // $array: ['name' => 'Desk', 'price' => 100]

<a name="method-array-pull"></a>
#### `array_pull()` {#collection-method}

`array_pull`関数は配列から指定されたキー／値ペアを取得し、同時に削除します。

    $array = ['name' => 'Desk', 'price' => 100];

    $name = array_pull($array, 'name');

    // $name: Desk

    // $array: ['price' => 100]

メソッドの第３引数として、デフォルト値を渡せます。この値はキーが存在しない場合に返されます。

    $value = array_pull($array, $key, $default);

<a name="method-array-set"></a>
#### `array_set()` {#collection-method}

`array_set`関数は「ドット」記法を使用し、深くネストした配列に値をセットします。

    $array = ['products' => ['desk' => ['price' => 100]]];

    array_set($array, 'products.desk.price', 200);

    // ['products' => ['desk' => ['price' => 200]]]

<a name="method-array-sort"></a>
#### `array_sort()` {#collection-method}

The `array_sort` function sorts an array by its values:

    $array = [
        'Desk',
        'Table',
        'Chair',
    ];

    $array = array_sort($array);

    /*
        [
            'Chair',
            'Desk',
            'Table',
        ]
    */

You may also sort the array by the results of the given Closure:

    $array = [
        ['name' => 'Desk'],
        ['name' => 'Chair'],
    ];

    $array = array_values(array_sort($array, function ($value) {
        return $value['name'];
    }));

    /*
        [
            ['name' => 'Chair'],
            ['name' => 'Desk'],
        ]
    */

<a name="method-array-sort-recursive"></a>
#### `array_sort_recursive()` {#collection-method}

`array_sort_recursive`関数は`sort`機能を使い配列を再帰的にソートします。

    $array = [
        [
            'Roman',
            'Taylor',
            'Li',
        ],
        [
            'PHP',
            'Ruby',
            'JavaScript',
        ],
    ];

    $array = array_sort_recursive($array);

    /*
        [
            [
                'Li',
                'Roman',
                'Taylor',
            ],
            [
                'JavaScript',
                'PHP',
                'Ruby',
            ]
        ];
    */

<a name="method-array-where"></a>
#### `array_where()` {#collection-method}

`array_where`は指定されたクロージャで、配列をフィルタリングします。

    $array = [100, '200', 300, '400', 500];

    $array = array_where($array, function ($value, $key) {
        return is_string($value);
    });

    // [1 => 200, 3 => 400]

<a name="method-array-wrap"></a>
#### `array_wrap()` {#collection-method}

`array_wrap`関数は、指定した値を配列中にラップします。指定した値が配列中に存在している場合は、変更されません。

    $string = 'Laravel';

    $array = array_wrap($string);

    // [0 => 'Laravel']

<a name="method-head"></a>
#### `head()` {#collection-method}

`head`関数は、配列の最初の要素を返します。

    $array = [100, 200, 300];

    $first = head($array);

    // 100

<a name="method-last"></a>
#### `last()` {#collection-method}

`last`関数は指定した配列の最後の要素を返します。

    $array = [100, 200, 300];

    $last = last($array);

    // 300

<a name="paths"></a>
## パス

<a name="method-app-path"></a>
#### `app_path()` {#collection-method}

`app_path`関数は、`app`ディレクトリへの完全パスを取得します。また、`app_path`関数は、ファイルパスをアプリケーションディレクトリからの相対位置で渡し、完全なパスを生成することもできます。

    $path = app_path();

    $path = app_path('Http/Controllers/Controller.php');

<a name="method-base-path"></a>
#### `base_path()` {#collection-method}

`base_path`関数は、プロジェクトルートの完全パスを返します。`base_path`関数はさらに、指定されたプロジェクトルートディレクトリからの相対パスから絶対パスを生成します。

    $path = base_path();

    $path = base_path('vendor/bin');

<a name="method-config-path"></a>
#### `config_path()` {#collection-method}

`config_path`関数はアプリケーション設定ディレクトリの完全パスを返します。

    $path = config_path();

<a name="method-database-path"></a>
#### `database_path()` {#collection-method}

`database_path`関数はアプリケーションデータベースディレクトリへの完全パスを返します。

    $path = database_path();

<a name="method-mix"></a>
#### `mix()` {#collection-method}

`mix`関数は、[バージョンつけしたMixファイル](/docs/{{version}}/mix)へのパスを返します。

    mix($file);

<a name="method-public-path"></a>
#### `public_path()` {#collection-method}

`public_path`関数は、`public`ディレクトリの完全パスを返します。

    $path = public_path();

<a name="method-resource-path"></a>
#### `resource_path()` {#collection-method}

`resource_path`関数は、`resource`ディレクトリの完全パスを返します。リソースディレクトリからの相対パスを`resource_path`関数に指定することで、完全パスを生成することもできます。

    $path = resource_path();

    $path = resource_path('assets/sass/app.scss');

<a name="method-storage-path"></a>
#### `storage_path()` {#collection-method}

`storage_path`関数は、`storage`ディレクトリの完全パスを返します。また、`storage_path`関数は、指定されたstorageディレクトリからの相対ファイルパスから、完全パスを生成します。

    $path = storage_path();

    $path = storage_path('app/file.txt');

<a name="strings"></a>
## 文字列

<a name="method-camel-case"></a>
#### `camel_case()` {#collection-method}

`camel_case`関数は、文字列をキャメルケース（２つ目以降の単語の先頭は大文字）へ変換します。

    $camel = camel_case('foo_bar');

    // fooBar

<a name="method-class-basename"></a>
#### `class_basename()` {#collection-method}

`class_basename`関数は指定されたクラス名から名前空間を除いた、クラス名だけを取得します。

    $class = class_basename('Foo\Bar\Baz');

    // Baz

<a name="method-e"></a>
#### `e()` {#collection-method}

`e`関数は、PHPの`htmlspecialchars`関数を`double_encode`オプションに`false`を指定し、実行します。

    echo e('<html>foo</html>');

    // &lt;html&gt;foo&lt;/html&gt;

<a name="method-ends-with"></a>
#### `ends_with()` {#collection-method}

`ends_with`関数は、最初の文字列が２つ目の引数の文字列で終わっているか調べます。

    $value = ends_with('This is my name', 'name');

    // true

<a name="method-kebab-case"></a>
#### `kebab_case()` {#collection-method}

`kebab_case`関数は、指定した文字列を「ケバブ-ケース」に変換します。

    $value = kebab_case('fooBar');

    // foo-bar


<a name="method-snake-case"></a>
#### `snake_case()` {#collection-method}

`snake_case`関数は文字列をスネークケース（小文字名下線区切り）に変換します。

    $snake = snake_case('fooBar');

    // foo_bar

<a name="method-str-limit"></a>
#### `str_limit()` {#collection-method}

`str_limit`関数は文字列を文字数へ短くします。最初の引数に文字列を受付、最終的な文字列の最大文字数が第２引数です。

    $value = str_limit('The PHP framework for web artisans.', 7);

    // The PHP...

<a name="method-starts-with"></a>
#### `starts_with()` {#collection-method}

`starts_with`関数は指定した文字列が、２番めの文字列で始まっているか調べます。

    $value = starts_with('This is my name', 'This');

    // true

<a name="method-str-after"></a>
#### `str_after()` {#collection-method}

`str_after`関数は、指定した値に続く文字列を全て返します。

    $value = str_after('This is: a test', 'This is:');

    // ' a test'

<a name="method-str-before"></a>
#### `str_before()` {#collection-method}

`str_before`関数は、文字列中の指定した値より前の文字列を全部返します。

    $value = str_before('Test :it before', ':it before');

    // 'Test '

<a name="method-str-contains"></a>
#### `str_contains()` {#collection-method}

`str_contains`関数は指定した文字列が、２つ目の文字列を含んでいるか調べます。

    $value = str_contains('This is my name', 'my');

    // true

指定した文字列に値のどれかが含まれているかを判定するために、値の配列を渡すことも可能です。

    $value = str_contains('This is my name', ['my', 'foo']);

    // true

<a name="method-str-finish"></a>
#### `str_finish()` {#collection-method}

`str_finish`関数は指定した文字列の最後が、２つ目の引数で終了していない場合、その値を追加します。

    $string = str_finish('this/string', '/');
    $string2 = str_finish('this/string/', '/');

    // this/string/

<a name="method-str-is"></a>
#### `str_is()` {#collection-method}

`str_is`関数は指定した文字列がパターンに一致しているかを判定します。アスタリスクが使用されるとワイルドカードとして利用されます。

    $value = str_is('foo*', 'foobar');

    // true

    $value = str_is('baz*', 'foobar');

    // false

<a name="method-str-plural"></a>
#### `str_plural()` {#collection-method}

`str_plural`関数は単数形を複数形へ変換します。この関数は現在英語のみサポートしています。

    $plural = str_plural('car');

    // cars

    $plural = str_plural('child');

    // children

整数をこの関数の第２引数に指定することで、文字列の単数形と複数形を切り替えて取得できます。

    $plural = str_plural('child', 2);

    // children

    $plural = str_plural('child', 1);

    // child

<a name="method-str-random"></a>
#### `str_random()` {#collection-method}

`str_random`関数は指定された長さのランダムな文字列を生成します。この関数は、PHPの`random_bytes`関数を使用します。

    $string = str_random(40);

<a name="method-str-singular"></a>
#### `str_singular()` {#collection-method}

`str_singular`関数は複数形を単数形へ変換します。この関数は、現在英語のみサポートしています。

    $singular = str_singular('cars');

    // car

<a name="method-str-slug"></a>
#### `str_slug()` {#collection-method}

`str_slug`関数は指定された文字列から、URLフレンドリーな「スラグ」を生成します。

    $title = str_slug('Laravel 5 Framework', '-');

    // laravel-5-framework

<a name="method-studly-case"></a>
#### `studly_case()` {#collection-method}

`studly_case`関数は文字列をアッパーキャメルケース（単語の頭文字を大文字）に変換します。

    $value = studly_case('foo_bar');

    // FooBar

<a name="method-title-case"></a>
#### `title_case()` {#collection-method}

`title_case`関数は、指定された文字列を「タイトルケース」へ変換します。

    $title = title_case('a nice title uses the correct case');

    // A Nice Title Uses The Correct Case

<a name="method-trans"></a>
#### `trans()` {#collection-method}

`trans`関数は指定した言語行を[言語ファイル](/docs/{{version}}/localization)を使用して翻訳します。

    echo trans('validation.required'):

<a name="method-trans-choice"></a>
#### `trans_choice()` {#collection-method}

`trans_choice`関数は指定された言語行を数値をもとに翻訳します。`Lang::choice`のエイリアスです。

    $value = trans_choice('foo.bar', $count);

<a name="urls"></a>
## URL

<a name="method-action"></a>
#### `action()` {#collection-method}

`action`関数は、指定されたコントローラアクションのURLを生成します。完全修飾コントローラー名は必要ありません。代わりに、`App\Http\Controllers`名前空間からの相対クラス名を指定してください。

    $url = action('HomeController@index');

メソッドがルートパラメーターを受け付ける場合は、第２引数で指定してください。

    $url = action('UserController@profile', ['id' => 1]);

<a name="method-asset"></a>
#### `asset()` {#collection-method}

現在のリクエストのスキーマ(HTTPかHTTPS)を使い、アセットへのURLを生成します。

    $url = asset('img/photo.jpg');

<a name="method-secure-asset"></a>
#### `secure_asset()` {#collection-method}

HTTPSを使い、アセットへのURLを生成します。

    echo secure_asset('foo/bar.zip');

<a name="method-route"></a>
#### `route()` {#collection-method}

`route`関数は指定された名前付きルートへのURLを生成します。

    $url = route('routeName');

ルートにパラメーターを受け付ける場合は第２引数で指定します

    $url = route('routeName', ['id' => 1]);

`route`関数はデフォルトとして絶対URLを生成します。相対URLを生成したい場合は、第３引数に`false`を渡してください。

    $url = route('routeName', ['id' => 1], false);

<a name="method-secure-url"></a>
#### `secure_url()` {#collection-method}

`secure_url`関数は、指定したパスへの完全なHTTPS URLを生成します。

    echo secure_url('user/profile');

    echo secure_url('user/profile', [1]);

<a name="method-url"></a>
#### `url()` {#collection-method}

`url`関数は指定したパスへの完全なURLを生成します。

    echo url('user/profile');

    echo url('user/profile', [1]);

パスを指定しない場合は、`Illuminate\Routing\UrlGenerator`インスタンスを返します。

    echo url()->current();
    echo url()->full();
    echo url()->previous();

<a name="miscellaneous"></a>
## その他

<a name="method-abort"></a>
#### `abort()` {#collection-method}

`abort`関数は、例外ハンドラによりレンダーされるであろう、HTTP例外を投げます。

    abort(401);

例外のレスポンステキストを指定することもできます。

    abort(401, 'Unauthorized.');

<a name="method-abort-if"></a>
#### `abort_if()` {#collection-method}

`abort_if`関数は、指定された論理値が`true`と評価された場合に、HTTP例外を投げます。

    abort_if(! Auth::user()->isAdmin(), 403);

<a name="method-abort-unless"></a>
#### `abort_unless()` {#collection-method}

`abort_unless`関数は、指定した論理値が`false`と評価された場合に、HTTP例外を投げます。

    abort_unless(Auth::user()->isAdmin(), 403);

<a name="method-auth"></a>
#### `auth()` {#collection-method}

`auth`関数はAuthenticatorインスタンスを返します。`Auth`ファサードの代わりに便利に使用できます。

    $user = auth()->user();

<a name="method-back"></a>
#### `back()` {#collection-method}

`back`関数はユーザーの直前のロケーションへのリダイレクトレスポンスを生成します。

    return back();

<a name="method-bcrypt"></a>
#### `bcrypt()` {#collection-method}

`bcrypt`関数は指定した値をBcryptを使用しハッシュ化します。`Hash`ファサードの代用として便利に使用できます。

    $password = bcrypt('my-secret-password');

<a name="method-cache"></a>
#### `cache()` {#collection-method}

`cache`関数はキャッシュから値を取得するために使用します。キャッシュに指定したキーが存在しない場合、オプション値が返されます。

    $value = cache('key');

    $value = cache('key', 'default');

関数にキー／値ペアの配列を渡すと、アイテムをキャッシュへ追加します。さらに分数、もしくはキャッシュ値が有効であると推定される期限を渡すこともできます。

    cache(['key' => 'value'], 5);

    cache(['key' => 'value'], Carbon::now()->addSeconds(10));

<a name="method-collect"></a>
#### `collect()` {#collection-method}

`collect`関数は指定したアイテムから、[コレクション](/docs/{{version}}/collections)インスタンスを生成します。

    $collection = collect(['taylor', 'abigail']);

<a name="method-config"></a>
#### `config()` {#collection-method}

`config`関数は設定変数の値を取得します。設定値はファイル名とアクセスしたいオプションを「ドット」記法で指定します。デフォルト値が指定でき、設定オプションが存在しない時に返されます。

    $value = config('app.timezone');

    $value = config('app.timezone', $default);

`config`ヘルパにキー／値ペアの配列を渡せば、実行時の設定変数をセットすることもできます。

    config(['app.debug' => true]);

<a name="method-csrf-field"></a>
#### `csrf_field()` {#collection-method}

`csrf_field`関数は、CSRFトークン値を持つHTML「隠し」入力フィールドを生成します。[ブレード記法](/docs/{{version}}/blade)を使用した例です。

    {{ csrf_field() }}

<a name="method-csrf-token"></a>
#### `csrf_token()` {#collection-method}

`csrf_field`関数は、CSRFトークン値を持つHTML「隠し」入力フィールドを生成します。[ブレード記法](/docs/{{version}}/blade)を使用した例です。

    $token = csrf_token();

<a name="method-dd"></a>
#### `dd()` {#collection-method}

`dd`関数は指定された変数の内容を表示し、スクリプトの実行を停止します。

    dd($value);

    dd($value1, $value2, $value3, ...);

スクリプトの実行を停止したくない場合は、代わりに`dump`関数を使ってください。

    dump($value);

<a name="method-dispatch"></a>
#### `dispatch()` {#collection-method}

`dispatch`関数は、Laravelの[ジョブキュー](/docs/{{version}}/queues)へ、新しいジョブを投入します。

    dispatch(new App\Jobs\SendEmails);

<a name="method-env"></a>
#### `env()` {#collection-method}

`env`関数は環境変数の値を取得します。取得できない場合はデフォルト値を返します。

    $env = env('APP_ENV');

    // 変数が存在しない場合にデフォルト値を返す
    $env = env('APP_ENV', 'production');

<a name="method-event"></a>
#### `event()` {#collection-method}

`event`関数は指定した[イベント](/docs/{{version}}/events)をリスナに対して発行します

    event(new UserRegistered($user));

<a name="method-factory"></a>
#### `factory()` {#collection-method}

`factory`関数は指定したクラス、名前、個数のモデルファクトリビルダを生成します。これは[テスト](/docs/{{version}}/database-testing#writing-factories)や[シーディング（DB初期値設定）](/docs/{{version}}/seeding#using-model-factories)で使用できます。

    $user = factory(App\User::class)->make();

<a name="method-info"></a>
#### `info()` {#collection-method}

`info`関数はログへ情報(information)を書き出します。

    info('Some helpful information!');

関連情報の配列を関数へ渡すこともできます。

    info('User login attempt failed.', ['id' => $user->id]);

<a name="method-logger"></a>
#### `logger()` {#collection-method}

`logger`関数は、`debug`レベルのメッセージをログへ書き出します。

    logger('Debug message');

関連情報の配列を関数へ渡すこともできます。

    logger('User has logged in.', ['id' => $user->id]);

関数に値を渡さない場合は、[ロガー](/docs/{{version}}/errors#logging)インスタンスが返されます。

    logger()->error('You are not allowed here.');

<a name="method-method-field"></a>
#### `method_field()` {#collection-method}

`method_field`関数はフォームのHTTP動詞の見せかけの値を保持する「隠し」HTTP入力フィールドを生成します。[Blade記法](/docs/{{version}}/blade)を使う例です。

    <form method="POST">
        {{ method_field('DELETE') }}
    </form>

<a name="method-now"></a>
#### `now()` {#collection-method}

`now`関数は、現時点を表す新しい`Illuminate\Support\Carbon`インスタンスを生成します。

    return now();

<a name="method-old"></a>
#### `old()` {#collection-method}

`old`関数はセッションにフラッシュデーターとして保存されている直前の入力値を[取得](/docs/{{version}}/requests#retrieving-input)します。

    $value = old('value');

    $value = old('value', 'default');

<a name="method-optional"></a>
#### `optional()` {#collection-method}

`optional`関数は、どんな引数も指定でき、そのオブジェクトのプロパティへアクセスしたり、メソッドを呼び出したりできます。オブジェクトが`null`の場合、プロパティとメソッドはエラーを発生させる代わりに、シンプルに`null`を返します。

    return optional($user->address)->street;

    {!! old('name', optional($user)->name) !!}

<a name="method-redirect"></a>
#### `redirect()` {#collection-method}

`redirect`関数は、リダイレクトHTTPレスポンスを返します。引数無しで呼び出した場合は、リダイレクタインスタンスを返します。

    return redirect('/home');

    return redirect()->route('route.name');

<a name="method-report"></a>
#### `report()` {#collection-method}

`report`関数は、例外ハンドラの`report`メソッドを利用し、例外をレポートします。

    report($e);

<a name="method-request"></a>
#### `request()` {#collection-method}

`request`関数は現在の[リクエスト](/docs/{{version}}/requests)インスタンスを返すか、入力アイテムを取得します。

    $request = request();

    $value = request('key', $default = null)

<a name="method-rescue"></a>
#### `rescue()` {#collection-method}

`rescue`関数は指定されたクロージャを実行し、実行時に発生する例外をキャッチします。キャッチされた例外は、すべて例外ハンドラーの`report`メソッドに送られます。しかし、リクエストは引き続き処理されます。

    return rescue(function () {
        return $this->method();
    });

`rescue`関数には第2引数を渡すことができます。クロージャ実行時に例外が発生した場合、第2引数に渡した値が返されるデフォルトの値になります。

    return rescue(function () {
        return $this->method();
    }, false);

    return rescue(function () {
        return $this->method();
    }, function () {
        return $this->failure();
    });

<a name="method-response"></a>
#### `response()` {#collection-method}

`response`関数は[response](/docs/{{version}}/responses)インスタンスを返すか、レスポンスファクトリのインスタンスを取得します。

    return response('Hello World', 200, $headers);

    return response()->json(['foo' => 'bar'], 200, $headers);

<a name="method-retry"></a>
#### `retry()` {#collection-method}

The `retry` function attempts to execute the given callback until the given maximum attempt threshold is met. If the callback does not throw an exception, its return value will be returned. If the callback throws an exception, it will automatically be retried. If the maximum attempt count is exceeded, the exception will be thrown:

    return retry(5, function () {
        // 実行間で500ms空け、５回試行する
    }, 100);

<a name="method-session"></a>
#### `session()` {#collection-method}

`session`関数はセッションへ値を設定、もしくは取得するために使用します。

    $value = session('key');

キー／値ペアの配列を渡し、値を設定することができます。

    session(['chairs' => 7, 'instruments' => 3]);

関数に引数を渡さない場合は、セッション保存域が返されます。

    $value = session()->get('key');

    session()->put('key', $value);

<a name="method-tap"></a>
#### `tap()` {#collection-method}

The `tap` function accepts two arguments: an arbitrary `$value` and a Closure. The `$value` will be passed to the Closure and then be returned by the `tap` function. The return value of the Closure is irrelevant:　`tap`関数は引数を２つ取ります。アビリティの`$value`とクロージャです。`$value`はクロージャに渡され、それから`tap`関数により返されます。クロージャから返す値は無視されます。

    $user = tap(User::first(), function ($user) {
        $user->name = 'taylor';

        $user->save();
    });

`tap`関数でクロージャを指定しない場合、渡した`$value`のメソッドを呼び出せます。メソッド呼び出しの戻り値は常に`$value`になり、メソッドが実際に返す値とは無関係です。たとえば、Eloquentの`update`メソッドは、通常整数値を返します。しかし、`update`メソッドを`tap`関数にチェーンして呼び出すことで、メソッドにモデル自身を返すように強制できます。

    $user = tap($user)->update([
        'name' => $name,
        'email' => $email
    ]);

<a name="method-today"></a>
#### `today()` {#collection-method}

`today`関数は、現在の日付を表す新しい`Illuminate\Support\Carbon`インスタンスを生成します。

    return today();

<a name="method-value"></a>
#### `value()` {#collection-method}

`value`関数は値が指定された場合は、シンプルにその値を返します。しかし「クロージャ」が関数に渡されると、そのクロージャを実行し結果を返します。

    $value = value(function () {
        return 'bar';
    });

<a name="method-view"></a>
#### `view()` {#collection-method}

`view`関数は[view](/docs/{{version}}/views)インスタンスを返します。

    return view('auth.login');
