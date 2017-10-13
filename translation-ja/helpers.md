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

### 配列とオブジェクト

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
[array_random](#method-array-random)
[array_set](#method-array-set)
[array_sort](#method-array-sort)
[array_sort_recursive](#method-array-sort-recursive)
[array_where](#method-array-where)
[array_wrap](#method-array-wrap)
[data_fill](#method-data-fill)
[data_get](#method-data-get)
[data_set](#method-data-set)
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

[\__](#method-__)
[camel_case](#method-camel-case)
[class_basename](#method-class-basename)
[e](#method-e)
[ends_with](#method-ends-with)
[kebab_case](#method-kebab-case)
[preg_replace_array](#method-preg-replace-array)
[snake_case](#method-snake-case)
[starts_with](#method-starts-with)
[str_after](#method-str-after)
[str_before](#method-str-before)
[str_contains](#method-str-contains)
[str_finish](#method-str-finish)
[str_is](#method-str-is)
[str_limit](#method-str-limit)
[str_plural](#method-str-plural)
[str_random](#method-str-random)
[str_replace_array](#method-str-replace-array)
[str_replace_first](#method-str-replace-first)
[str_replace_last](#method-str-replace-last)
[str_singular](#method-str-singular)
[str_slug](#method-str-slug)
[str_start](#method-str-start)
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
[app](#method-app)
[auth](#method-auth)
[back](#method-back)
[bcrypt](#method-bcrypt)
[blank](#method-blank)
[broadcast](#method-broadcast)
[cache](#method-cache)
[class_uses_recursive](#method-class-uses-recursive)
[collect](#method-collect)
[config](#method-config)
[cookie](#method-cookie)
[csrf_field](#method-csrf-field)
[csrf_token](#method-csrf-token)
[dd](#method-dd)
[decrypt](#method-decrypt)
[dispatch](#method-dispatch)
[dispatch_now](#method-dispatch-now)
[dump](#method-dump)
[encrypt](#method-encrypt)
[env](#method-env)
[event](#method-event)
[factory](#method-factory)
[filled](#method-filled)
[info](#method-info)
[logger](#method-logger)
[method_field](#method-method-field)
[now](#method-now)
[old](#method-old)
[optional](#method-optional)
[policy](#method-policy)
[redirect](#method-redirect)
[report](#method-report)
[request](#method-request)
[rescue](#method-rescue)
[resolve](#method-resolve)
[response](#method-response)
[retry](#method-retry)
[session](#method-session)
[tap](#method-tap)
[today](#method-today)
[throw_if](#method-throw-if)
[throw_unless](#method-throw-unless)
[trait_uses_recursive](#method-trait-uses-recursive)
[transform](#method-transform)
[validator](#method-validator)
[value](#method-value)
[view](#method-view)
[with](#method-with)

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
## Arrays & Objects

<a name="method-array-add"></a>
#### `array_add()` {#collection-method .first-collection-method}

The `array_add` function adds a given key / value pair to an array if the given key doesn't already exist in the array:

    $array = array_add(['name' => 'Desk'], 'price', 100);

    // ['name' => 'Desk', 'price' => 100]

<a name="method-array-collapse"></a>
#### `array_collapse()` {#collection-method}

`array_collapse`関数は配列の配列を一次元の配列へ展開します。

    $array = array_collapse([[1, 2, 3], [4, 5, 6], [7, 8, 9]]);

    // [1, 2, 3, 4, 5, 6, 7, 8, 9]

<a name="method-array-divide"></a>
#### `array_divide()` {#collection-method}

The `array_divide` function returns two arrays, one containing the keys, and the other containing the values of the given array:

    list($keys, $values) = array_divide(['name' => 'Desk']);

    // $keys: ['name']

    // $values: ['Desk']

<a name="method-array-dot"></a>
#### `array_dot()` {#collection-method}

`array_dot`関数は多次元配列を「ドット」記法で深さを表した一次元配列に変換します。

    $array = ['products' => ['desk' => ['price' => 100]]];

    $flattened = array_dot($array);

    // ['products.desk.price' => 100]

<a name="method-array-except"></a>
#### `array_except()` {#collection-method}

The `array_except` function removes the given key / value pairs from an array:

    $array = ['name' => 'Desk', 'price' => 100];

    $filtered = array_except($array, ['price']);

    // ['name' => 'Desk']

<a name="method-array-first"></a>
#### `array_first()` {#collection-method}

`array_first`関数は指定されたテストにパスした最初の要素を返します。

    $array = [100, 200, 300];

    $first = array_first($array, function ($value, $key) {
        return $value >= 150;
    });

    // 200

デフォルト値を３つ目の引数で指定することもできます。この値はテストでどの値もテストにパスしない場合に返されます。

    $first = array_first($array, $callback, $default);

<a name="method-array-flatten"></a>
#### `array_flatten()` {#collection-method}

The `array_flatten` function flattens a multi-dimensional array into a single level array:

    $array = ['name' => 'Joe', 'languages' => ['PHP', 'Ruby']];

    $flattened = array_flatten($array);

    // ['Joe', 'PHP', 'Ruby']

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

    $price = array_get($array, 'products.desk.price');

    // 100

`array_get`関数は、指定したキーが存在しない場合に返されるデフォルト値を指定できます。

    $discount = array_get($array, 'products.desk.discount', 0);

    // 0

<a name="method-array-has"></a>
#### `array_has()` {#collection-method}

The `array_has` function checks whether a given item or items exists in an array using "dot" notation:

    $array = ['product' => ['name' => 'Desk', 'price' => 100]];

    $contains = array_has($array, 'product.name');

    // true

    $contains = array_has($array, ['product.price', 'product.discount']);

    // false

<a name="method-array-last"></a>
#### `array_last()` {#collection-method}

`array_last`関数は、テストでパスした最後の配列要素を返します。

    $array = [100, 200, 300, 110];

    $last = array_last($array, function ($value, $key) {
        return $value >= 150;
    });

    // 300

メソッドの第３引数には、デフォルト値を渡します。テストでパスする値がない場合に、返されます。

    $last = array_last($array, $callback, $default);

<a name="method-array-only"></a>
#### `array_only()` {#collection-method}

The `array_only` function returns only the specified key / value pairs from the given array:

    $array = ['name' => 'Desk', 'price' => 100, 'orders' => 10];

    $slice = array_only($array, ['name', 'price']);

    // ['name' => 'Desk', 'price' => 100]

<a name="method-array-pluck"></a>
#### `array_pluck()` {#collection-method}

The `array_pluck` function retrieves all of the values for a given key from an array:

    $array = [
        ['developer' => ['id' => 1, 'name' => 'Taylor']],
        ['developer' => ['id' => 2, 'name' => 'Abigail']],
    ];

    $names = array_pluck($array, 'developer.name');

    // ['Taylor', 'Abigail']

さらに、結果のリストのキー項目も指定できます。

    $names = array_pluck($array, 'developer.name', 'developer.id');

    // [1 => 'Taylor', 2 => 'Abigail']

<a name="method-array-prepend"></a>
#### `array_prepend()` {#collection-method}

`array_prepend`関数は配列の先頭にアイテムを追加します。

    $array = ['one', 'two', 'three', 'four'];

    $array = array_prepend($array, 'zero');

    // ['zero', 'one', 'two', 'three', 'four']

必要であれば、値に対するキーを指定できます。

    $array = ['price' => 100];

    $array = array_prepend($array, 'Desk', 'name');

    // ['name' => 'Desk', 'price' => 100]

<a name="method-array-pull"></a>
#### `array_pull()` {#collection-method}

The `array_pull` function returns and removes a key / value pair from an array:

    $array = ['name' => 'Desk', 'price' => 100];

    $name = array_pull($array, 'name');

    // $name: Desk

    // $array: ['price' => 100]

メソッドの第３引数として、デフォルト値を渡せます。この値はキーが存在しない場合に返されます。

    $value = array_pull($array, $key, $default);

<a name="method-array-random"></a>
#### `array_random()` {#collection-method}

The `array_random` function returns a random value from an array:

    $array = [1, 2, 3, 4, 5];

    $random = array_random($array);

    // 4 - (ランダムに取得された値)

You may also specify the number of items to return as an optional second argument. Note that providing this argument will return an array, even if only one item is desired:

    $items = array_random($array, 2);

    // [2, 5] - (retrieved randomly)

<a name="method-array-set"></a>
#### `array_set()` {#collection-method}

`array_set`関数は「ドット」記法を使用し、深くネストした配列に値をセットします。

    $array = ['products' => ['desk' => ['price' => 100]]];

    array_set($array, 'products.desk.price', 200);

    // ['products' => ['desk' => ['price' => 200]]]

<a name="method-array-sort"></a>
#### `array_sort()` {#collection-method}

`array_sort`関数は、配列の値に基づきソートします。

    $array = ['Desk', 'Table', 'Chair'];

    $sorted = array_sort($array);

    // ['Chair', 'Desk', 'Table']

指定したクロージャの結果に基づき、ソートすることもできます。

    $array = [
        ['name' => 'Desk'],
        ['name' => 'Table'],
        ['name' => 'Chair'],
    ];

    $sorted = array_values(array_sort($array, function ($value) {
        return $value['name'];
    }));

    /*
        [
            ['name' => 'Chair'],
            ['name' => 'Desk'],
            ['name' => 'Table'],
        ]
    */

<a name="method-array-sort-recursive"></a>
#### `array_sort_recursive()` {#collection-method}

The `array_sort_recursive` function recursively sorts an array using the `sort` function:

    $array = [
        ['Roman', 'Taylor', 'Li'],
        ['PHP', 'Ruby', 'JavaScript'],
    ];

    $sorted = array_sort_recursive($array);

    /*
        [
            ['Li', 'Roman', 'Taylor'],
            ['JavaScript', 'PHP', 'Ruby'],
        ]
    */

<a name="method-array-where"></a>
#### `array_where()` {#collection-method}

The `array_where` function filters an array using the given Closure:

    $array = [100, '200', 300, '400', 500];

    $filtered = array_where($array, function ($value, $key) {
        return is_string($value);
    });

    // [1 => 200, 3 => 400]

<a name="method-array-wrap"></a>
#### `array_wrap()` {#collection-method}

The `array_wrap` function wraps the given value in an array. If the given value is already an array it will not be changed:

    $string = 'Laravel';

    $array = array_wrap($string);

    // ['Laravel']

<a name="method-data-fill"></a>
#### `data_fill()` {#collection-method}

The `data_fill` function sets a missing value within a nested array or object using "dot" notation:

    $data = ['products' => ['desk' => ['price' => 100]]];

    data_fill($data, 'products.desk.price', 200);

    // ['products' => ['desk' => ['price' => 100]]]

    data_fill($data, 'products.desk.discount', 10);

    // ['products' => ['desk' => ['price' => 100, 'discount' => 10]]]

この関数はアスタリスクもワイルドカードとして受け取り、それに応じてターゲットにデータを埋め込みます。

    $data = [
        'products' => [
            ['name' => 'Desk 1' => 'price' => 100],
            ['name' => 'Desk 2'],
        ],
    ];

    data_fill($data, 'products.*.price', 200);

    /*
        [
            'products' => [
                ['name' => 'Desk 1' => 'price' => 100],
                ['name' => 'Desk 2' => 'price' => 200],
            ],
        ]
    */

<a name="method-data-get"></a>
#### `data_get()` {#collection-method}

`data_get`関数は「ドット」記法を使用し、ネストした配列やオブジェクトから値を取得します。

    $data = ['products' => ['desk' => ['price' => 100]]];

    $price = data_get($data, 'products.desk.price');

    // 100

`data_get`関数は、指定したキーが存在しない場合に返す、デフォルト値も指定できます。

    $discount = data_get($data, 'products.desk.discount', 0);

    // 0

<a name="method-data-set"></a>
#### `data_set()` {#collection-method}

`data_set`関数は「ドット」記法を使用し、ネストした配列やオブジェクトに値をセットします。

    $data = ['products' => ['desk' => ['price' => 100]]];

    data_set($data, 'products.desk.price', 200);

    // ['products' => ['desk' => ['price' => 200]]]

この関数はアスタリスクもワイルドカードとして受け取り、それに応じてターゲットにデータを埋め込みます。

    $data = [
        'products' => [
            ['name' => 'Desk 1', 'price' => 100],
            ['name' => 'Desk 2', 'price' => 150],
        ],
    ];

    data_set($data, 'products.*.price', 200);

    /*
        [
            'products' => [
                ['name' => 'Desk 1' => 'price' => 200],
                ['name' => 'Desk 2' => 'price' => 200],
            ],
        ]
    */

By default, any existing values are overwritten. If you wish to only set a value if it doesn't exist, you may pass `false` as the third argument:

    $data = ['products' => ['desk' => ['price' => 100]]];

    data_set($data, 'products.desk.price', 200, false);

    // ['products' => ['desk' => ['price' => 100]]]

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

The `config_path` function returns the fully qualified path to the `config` directory. You may also use the `config_path` function to generate a fully qualified path to a given file within the application's configuration directory:

    $path = config_path();

    $path = config_path('app.php');

<a name="method-database-path"></a>
#### `database_path()` {#collection-method}

The `database_path` function returns the fully qualified path to the `database` directory. You may also use the `database_path` function to generate a fully qualified path to a given file within the database directory:

    $path = database_path();

    $path = database_path('factories/UserFactory.php');

<a name="method-mix"></a>
#### `mix()` {#collection-method}

The `mix` function returns the path to a [versioned Mix file](/docs/{{version}}/mix):

    $path = mix('css/app.css');

<a name="method-public-path"></a>
#### `public_path()` {#collection-method}

The `public_path` function returns the fully qualified path to the `public` directory. You may also use the `public_path` function to generate a fully qualified path to a given file within the public directory:

    $path = public_path();

    $path = public_path('css/app.css');

<a name="method-resource-path"></a>
#### `resource_path()` {#collection-method}

The `resource_path` function returns the fully qualified path to the `resources` directory. You may also use the `resource_path` function to generate a fully qualified path to a given file within the resources directory:

    $path = resource_path();

    $path = resource_path('assets/sass/app.scss');

<a name="method-storage-path"></a>
#### `storage_path()` {#collection-method}

The `storage_path` function returns the fully qualified path to the `storage` directory. You may also use the `storage_path` function to generate a fully qualified path to a given file within the storage directory:

    $path = storage_path();

    $path = storage_path('app/file.txt');

<a name="strings"></a>
## 文字列

<a name="method-__"></a>
#### `__()` {#collection-method}

The `__` function translates the given translation string or translation key using your [localization files](/docs/{{version}}/localization):

    echo __('Welcome to our application');

    echo __('messages.welcome');

If the specified translation string or key does not exist, the `__` function will simply return the given value. So, using the example above, the `__` function would return `messages.welcome` if that translation key does not exist.

<a name="method-camel-case"></a>
#### `camel_case()` {#collection-method}

`camel_case`関数は、文字列をキャメルケース（２つ目以降の単語の先頭は大文字）へ変換します。

    $converted = camel_case('foo_bar');

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

    $result = ends_with('This is my name', 'name');

    // true

<a name="method-kebab-case"></a>
#### `kebab_case()` {#collection-method}

`kebab_case`関数は、指定した文字列を「ケバブ-ケース」に変換します。

    $converted = kebab_case('fooBar');

    // foo-bar

<a name="method-preg-replace-array"></a>
#### `preg_replace_array()` {#collection-method}

The `preg_replace_array` function replaces a given pattern in the string sequentially using an array:

    $string = 'The event will take place between :start and :end';

    $replaced = preg_replace_array('/:[a-z_]+/', ['8:30', '9:00'], $string);

    // The event will take place between 8:30 and 9:00

<a name="method-snake-case"></a>
#### `snake_case()` {#collection-method}

`snake_case`関数は文字列をスネークケース（小文字名下線区切り）に変換します。

    $converted = snake_case('fooBar');

    // foo_bar

<a name="method-starts-with"></a>
#### `starts_with()` {#collection-method}

`starts_with`関数は指定した文字列が、２番めの文字列で始まっているか調べます。

    $result = starts_with('This is my name', 'This');

    // true

<a name="method-str-after"></a>
#### `str_after()` {#collection-method}

`str_after`関数は、指定した値に続く文字列を全て返します。

    $slice = str_after('This is my name', 'This is');

    // ' my name'

<a name="method-str-before"></a>
#### `str_before()` {#collection-method}

`str_before`関数は、文字列中の指定した値より前の文字列を全部返します。

    $slice = str_before('This is my name', 'my name');

    // 'This is '

<a name="method-str-contains"></a>
#### `str_contains()` {#collection-method}

`str_contains`関数は指定した文字列が、２つ目の文字列を含んでいるか調べます。

    $contains = str_contains('This is my name', 'my');

    // true

指定した文字列に値のどれかが含まれているかを判定するために、値の配列を渡すことも可能です。

    $contains = str_contains('This is my name', ['my', 'foo']);

    // true

<a name="method-str-finish"></a>
#### `str_finish()` {#collection-method}

`str_finish`関数は指定した文字列の最後が、２つ目の引数の値で終了していない場合、その値を追加します。

    $adjusted = str_finish('this/string', '/');

    // this/string/

    $adjusted = str_finish('this/string/', '/');

    // this/string/

<a name="method-str-is"></a>
#### `str_is()` {#collection-method}

`str_is`関数は指定した文字列がパターンに一致しているかを判定します。アスタリスクが使用されるとワイルドカードとして利用されます。

    $matches = str_is('foo*', 'foobar');

    // true

    $matches = str_is('baz*', 'foobar');

    // false

<a name="method-str-limit"></a>
#### `str_limit()` {#collection-method}

The `str_limit` function truncates the given string at the specified length:

    $truncated = str_limit('The quick brown fox jumps over the lazy dog', 20);

    // The quick brown fox...

また、第３引数として、最長文字列数を超えた場合に末尾へ追加する、文字列を渡すこともできます。

    $truncated = str_limit('The quick brown fox jumps over the lazy dog', 20, ' (...)');

    // The quick brown fox (...)

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

    $random = str_random(40);

<a name="method-str-replace-array"></a>
#### `str_replace_array()` {#collection-method}

The `str_replace_array` function replaces a given value in the string sequentially using an array:

    $string = 'The event will take place between ? and ?';

    $replaced = str_replace_array('?', ['8:30', '9:00'], $string);

    // The event will take place between 8:30 and 9:00

<a name="method-str-replace-first"></a>
#### `str_replace_first()` {#collection-method}

The `str_replace_first` function replaces the first occurrence of a given value in a string:

    $replaced = str_replace_first('the', 'a', 'the quick brown fox jumps over the lazy dog');

    // a quick brown fox jumps over the lazy dog

<a name="method-str-replace-last"></a>
#### `str_replace_last()` {#collection-method}

The `str_replace_last` function replaces the last occurrence of a given value in a string:

    $replaced = str_replace_last('the', 'a', 'the quick brown fox jumps over the lazy dog');

    // the quick brown fox jumps over a lazy dog

<a name="method-str-singular"></a>
#### `str_singular()` {#collection-method}

`str_singular`関数は複数形を単数形へ変換します。この関数は、現在英語のみサポートしています。

    $singular = str_singular('cars');

    // car

    $singular = str_singular('children');

    // child

<a name="method-str-slug"></a>
#### `str_slug()` {#collection-method}

`str_slug`関数は指定された文字列から、URLフレンドリーな「スラグ」を生成します。

    $slug = str_slug('Laravel 5 Framework', '-');

    // laravel-5-framework

<a name="method-str-start"></a>
#### `str_start()` {#collection-method}

`str_start`関数は文字列が指定値から始まっていない場合、先頭にその文字列を追加します。

    $adjusted = str_start('this/string', '/');

    // /this/string

    $adjusted = str_start('/this/string/', '/');

    // /this/string

<a name="method-studly-case"></a>
#### `studly_case()` {#collection-method}

`studly_case`関数は文字列をアッパーキャメルケース（単語の頭文字を大文字）に変換します。

    $converted = studly_case('foo_bar');

    // FooBar

<a name="method-title-case"></a>
#### `title_case()` {#collection-method}

`title_case`関数は、指定された文字列を「タイトルケース」へ変換します。

    $converted = title_case('a nice title uses the correct case');

    // A Nice Title Uses The Correct Case

<a name="method-trans"></a>
#### `trans()` {#collection-method}

The `trans` function translates the given translation key using your [localization files](/docs/{{version}}/localization):

    echo trans('messages.welcome');

If the specified translation key does not exist, the `trans` function will simply return the given key. So, using the example above, the `trans` function would return `messages.welcome` if the translation key does not exist.

<a name="method-trans-choice"></a>
#### `trans_choice()` {#collection-method}

The `trans_choice` function translates the given translation key with inflection:

    echo trans_choice('messages.notifications', $unreadCount);

If the specified translation key does not exist, the `trans_choice` function will simply return the given key. So, using the example above, the `trans_choice` function would return `messages.notifications` if the translation key does not exist.

<a name="urls"></a>
## URL

<a name="method-action"></a>
#### `action()` {#collection-method}

`action`関数は、指定されたコントローラアクションのURLを生成します。完全修飾コントローラ名は必要ありません。代わりに、`App\Http\Controllers`名前空間からの相対クラス名を指定してください。

    $url = action('HomeController@index');

メソッドがルートパラメーターを受け付ける場合は、第２引数で指定してください。

    $url = action('UserController@profile', ['id' => 1]);

<a name="method-asset"></a>
#### `asset()` {#collection-method}

The `asset` function generates a URL for an asset using the current scheme of the request (HTTP or HTTPS):

    $url = asset('img/photo.jpg');

<a name="method-secure-asset"></a>
#### `secure_asset()` {#collection-method}

The `secure_asset` function generates a URL for an asset using HTTPS:

    $url = secure_asset('img/photo.jpg');

<a name="method-route"></a>
#### `route()` {#collection-method}

`route`関数は指定された名前付きルートへのURLを生成します。

    $url = route('routeName');

ルートにパラメーターを受け付ける場合は第２引数で指定します

    $url = route('routeName', ['id' => 1]);

By default, the `route` function generates an absolute URL. If you wish to generate a relative URL, you may pass `false` as the third argument:

    $url = route('routeName', ['id' => 1], false);

<a name="method-secure-url"></a>
#### `secure_url()` {#collection-method}

`secure_url`関数は、指定したパスへの完全なHTTPS URLを生成します。

    $url = secure_url('user/profile');

    $url = secure_url('user/profile', [1]);

<a name="method-url"></a>
#### `url()` {#collection-method}

`url`関数は指定したパスへの完全なURLを生成します。

    $url = url('user/profile');

    $url = url('user/profile', [1]);

パスを指定しない場合は、`Illuminate\Routing\UrlGenerator`インスタンスを返します。

    $current = url()->current();

    $full = url()->full();

    $previous = url()->previous();

<a name="miscellaneous"></a>
## その他

<a name="method-abort"></a>
#### `abort()` {#collection-method}

The `abort` function throws [an HTTP exception](/docs/{{version}}/errors#http-exceptions) which will be rendered by the [exception handler](/docs/{{version}}/errors#the-exception-handler):

    abort(403);

You may also provide the exception's response text and custom response headers:

    abort(403, 'Unauthorized.', $headers);

<a name="method-abort-if"></a>
#### `abort_if()` {#collection-method}

`abort_if`関数は、指定された論理値が`true`と評価された場合に、HTTP例外を投げます。

    abort_if(! Auth::user()->isAdmin(), 403);

Like the `abort` method, you may also provide the exception's response text as the third argument and an array of custom response headers as the fourth argument.

<a name="method-abort-unless"></a>
#### `abort_unless()` {#collection-method}

`abort_unless`関数は、指定した論理値が`false`と評価された場合に、HTTP例外を投げます。

    abort_unless(Auth::user()->isAdmin(), 403);

Like the `abort` method, you may also provide the exception's response text as the third argument and an array of custom response headers as the fourth argument.

<a name="method-app"></a>
#### `app()` {#collection-method}

`app`関数は、[サービスコンテナ](/docs/{{version}}/container)のインスタンスを返します。

    $container = app();

You may pass a class or interface name to resolve it from the container:

    $api = app('HelpSpot\API');

<a name="method-auth"></a>
#### `auth()` {#collection-method}

`auth`関数は、[authenticator](/docs/{{version}}/authentication)のインスタンスを返します。利便のため、代わりに`Auth`ファサードを使用することもできます。

    $user = auth()->user();

If needed, you may specify which guard instance you would like to access:

    $user = auth('admin')->user();

<a name="method-back"></a>
#### `back()` {#collection-method}

The `back` function generates a [redirect HTTP response](/docs/{{version}}/responses#redirects) to the user's previous location:

    return back($status = 302, $headers = [], $fallback = false);

    return back();

<a name="method-bcrypt"></a>
#### `bcrypt()` {#collection-method}

The `bcrypt` function [hashes](/docs/{{version}}/hashing) the given value using Bcrypt. You may use it as an alternative to the `Hash` facade:

    $password = bcrypt('my-secret-password');

<a name="method-broadcast"></a>
#### `broadcast()` {#collection-method}

`broadcast`関数は、指定した[イベント](/docs/{{version}}/events)をリスナへ[ブロードキャスト](/docs/{{version}}/broadcasting)します。

    broadcast(new UserRegistered($user));

<a name="method-blank"></a>
#### `blank()` {#collection-method}

`blank`関数は指定値が"blank"であるかどうかを返します。

    blank('');
    blank('   ');
    blank(null);
    blank(collect());

    // true

    blank(0);
    blank(true);
    blank(false);

    // false

For the inverse of `blank`, see the [`filled`](#method-filled) method.

<a name="method-cache"></a>
#### `cache()` {#collection-method}

The `cache` function may be used to get values from the [cache](/docs/{{version}}/cache). If the given key does not exist in the cache, an optional default value will be returned:

    $value = cache('key');

    $value = cache('key', 'default');

関数にキー／値ペアの配列を渡すと、アイテムをキャッシュへ追加します。さらに分数、もしくはキャッシュ値が有効であると推定される期限を渡すこともできます。

    cache(['key' => 'value'], 5);

    cache(['key' => 'value'], Carbon::now()->addSeconds(10));

<a name="method-class-uses-recursive"></a>
#### `class_uses_recursive()` {#collection-method}

The `class_uses_recursive` function returns all traits used by a class, including traits used by any subclasses:

    $traits = class_uses_recursive(App\User::class);

<a name="method-collect"></a>
#### `collect()` {#collection-method}

The `collect` function creates a [collection](/docs/{{version}}/collections) instance from the given value:

    $collection = collect(['taylor', 'abigail']);

<a name="method-config"></a>
#### `config()` {#collection-method}

The `config` function gets the value of a [configuration](/docs/{{version}}/configuration) variable. The configuration values may be accessed using "dot" syntax, which includes the name of the file and the option you wish to access. A default value may be specified and is returned if the configuration option does not exist:

    $value = config('app.timezone');

    $value = config('app.timezone', $default);

You may set configuration variables at runtime by passing an array of key / value pairs:

    config(['app.debug' => true]);

<a name="method-cookie"></a>
#### `cookie()` {#collection-method}

`cookie`関数は新しい[クッキー](/docs/{{version}}/requests#cookies)インスタンスを生成します。

    $cookie = cookie('name', 'value', $minutes);

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

If you do not want to halt the execution of your script, use the [`dump`](#method-dump) function instead.

<a name="method-decrypt"></a>
#### `decrypt()` {#collection-method}

The `decrypt` function decrypts the given value using Laravel's [encrypter](/docs/{{version}}/encryption):

    $decrypted = decrypt($encrypted_value);

<a name="method-dispatch"></a>
#### `dispatch()` {#collection-method}

`dispatch`関数は、指定した[ジョブ](/docs/{{version}}/queues#creating-jobs)をLaravelの[ジョブキュー](/docs/{{version}}/queues)へ投入します。

    dispatch(new App\Jobs\SendEmails);

<a name="method-dispatch-now"></a>
#### `dispatch_now()` {#collection-method}

`dispatch_now`関数は、指定した[ジョブ](/docs/{{version}}/queues#creating-jobs)を即時に実行し、`handle`メソッドからの値を返します。

    $result = dispatch_now(new App\Jobs\SendEmails);

<a name="method-dump"></a>
#### `dump()` {#collection-method}

The `dump` function dumps the given variables:

    dump($value);

    dump($value1, $value2, $value3, ...);

If you want to stop executing the script after dumping the variables, use the [`dd`](#method-dd) function instead.

<a name="method-encrypt"></a>
#### `encrypt()` {#collection-method}

The `encrypt` function encrypts the given value using Laravel's [encrypter](/docs/{{version}}/encryption):

    $encrypted = encrypt($unencrypted_value);

<a name="method-env"></a>
#### `env()` {#collection-method}

The `env` function retrieves the value of an [environment variable](/docs/{{version}}/configuration#environment-configuration) or returns a default value:

    $env = env('APP_ENV');

    // Returns 'production' if APP_ENV is not set...
    $env = env('APP_ENV', 'production');

<a name="method-event"></a>
#### `event()` {#collection-method}

`event`関数は指定した[イベント](/docs/{{version}}/events)をリスナに対して発行します

    event(new UserRegistered($user));

<a name="method-factory"></a>
#### `factory()` {#collection-method}

`factory`関数は指定したクラス、名前、個数のモデルファクトリビルダを生成します。これは[テスト](/docs/{{version}}/database-testing#writing-factories)や[シーディング（DB初期値設定）](/docs/{{version}}/seeding#using-model-factories)で使用できます。

    $user = factory(App\User::class)->make();

<a name="method-filled"></a>
#### `filled()` {#collection-method}

`filled`関数は、指定値が"blank"であるかどうかを返します。

    filled(0);
    filled(true);
    filled(false);

    // true

    filled('');
    filled('   ');
    filled(null);
    filled(collect());

    // false

For the inverse of `filled`, see the [`blank`](#method-blank) method.

<a name="method-info"></a>
#### `info()` {#collection-method}

The `info` function will write information to the [log](/docs/{{version}}/errors#logging):

    info('Some helpful information!');

関連情報の配列を関数へ渡すこともできます。

    info('User login attempt failed.', ['id' => $user->id]);

<a name="method-logger"></a>
#### `logger()` {#collection-method}

The `logger` function can be used to write a `debug` level message to the [log](/docs/{{version}}/errors#logging):

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

    $now = now();

<a name="method-old"></a>
#### `old()` {#collection-method}

The `old` function [retrieves](/docs/{{version}}/requests#retrieving-input) an [old input](/docs/{{version}}/requests#old-input) value flashed into the session:

    $value = old('value');

    $value = old('value', 'default');

<a name="method-optional"></a>
#### `optional()` {#collection-method}

`optional`関数は、どんな引数も指定でき、そのオブジェクトのプロパティへアクセスしたり、メソッドを呼び出したりできます。オブジェクトが`null`の場合、プロパティとメソッドはエラーを発生させる代わりに、シンプルに`null`を返します。

    return optional($user->address)->street;

    {!! old('name', optional($user)->name) !!}

<a name="method-policy"></a>
#### `policy()` {#collection-method}

The `policy` method retrieves a [policy](/docs/{{version}}/authorization#creating-policies) instance for a given class:

    $policy = policy(App\User::class);

<a name="method-redirect"></a>
#### `redirect()` {#collection-method}

The `redirect` function returns a [redirect HTTP response](/docs/{{version}}/responses#redirects), or returns the redirector instance if called with no arguments:

    return redirect($to = null, $status = 302, $headers = [], $secure = null);

    return redirect('/home');

    return redirect()->route('route.name');

<a name="method-report"></a>
#### `report()` {#collection-method}

The `report` function will report an exception using your [exception handler](/docs/{{version}}/errors#the-exception-handler)'s `report` method:

    report($e);

<a name="method-request"></a>
#### `request()` {#collection-method}

`request`関数は現在の[リクエスト](/docs/{{version}}/requests)インスタンスを返すか、入力アイテムを取得します。

    $request = request();

    $value = request('key', $default = null);

<a name="method-rescue"></a>
#### `rescue()` {#collection-method}

The `rescue` function executes the given Closure and catches any exceptions that occur during its execution. All exceptions that are caught will be sent to your [exception handler](/docs/{{version}}/errors#the-exception-handler)'s `report` method; however, the request will continue processing:

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

<a name="method-resolve"></a>
#### `resolve()` {#collection-method}

`resolve`関数は[サービスコンテナ](/docs/{{version}}/container)を使い、指定されたクラスやインターフェイスの名前から、そのインスタンス自身を依存解決します。

    $api = resolve('HelpSpot\API');

<a name="method-response"></a>
#### `response()` {#collection-method}

`response`関数は[response](/docs/{{version}}/responses)インスタンスを返すか、レスポンスファクトリのインスタンスを取得します。

    return response('Hello World', 200, $headers);

    return response()->json(['foo' => 'bar'], 200, $headers);

<a name="method-retry"></a>
#### `retry()` {#collection-method}

`retry`関数は指定された最大試行回数を過ぎるまで、指定されたコールバックを実行します。コールバックが例外を投げなければ、返却値が返されます。コールバックが例外を投げた場合は、自動的にリトライされます。最大試行回数を超えると、例外が投げられます。

    return retry(5, function () {
        // 実行間で500ms空け、５回試行する
    }, 100);

<a name="method-session"></a>
#### `session()` {#collection-method}

The `session` function may be used to get or set [session](/docs/{{version}}/session) values:

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
        'email' => $email,
    ]);

<a name="method-today"></a>
#### `today()` {#collection-method}

`today`関数は、現在の日付を表す新しい`Illuminate\Support\Carbon`インスタンスを生成します。

    $today = today();

<a name="method-throw-if"></a>
#### `throw_if()` {#collection-method}

`throw_if`関数は、指定した論理式が`true`と評価された場合に、指定した例外を投げます。

    throw_if(! Auth::user()->isAdmin(), AuthorizationException::class);

    throw_if(
        ! Auth::user()->isAdmin(),
        AuthorizationException::class,
        'You are not allowed to access this page'
    );

<a name="method-throw-unless"></a>
#### `throw_unless()` {#collection-method}

`throw_unless`関数は、指定した論理式が`false`と評価された場合に、指定した例外を投げます。

    throw_unless(Auth::user()->isAdmin(), AuthorizationException::class);

    throw_unless(
        Auth::user()->isAdmin(),
        AuthorizationException::class,
        'You are not allowed to access this page'
    );

<a name="method-trait-uses-recursive"></a>
#### `trait_uses_recursive()` {#collection-method}

The `trait_uses_recursive` function returns all traits used by a trait:

    $traits = trait_uses_recursive(\Illuminate\Notifications\Notifiable::class);

<a name="method-transform"></a>
#### `transform()` {#collection-method}

The `transform` function executes a `Closure` on a given value if the value is not [blank](#method-blank) and returns the result of the `Closure`:

    $callback = function ($value) {
        return $value * 2;
    };

    $result = transform(5, $callback);

    // 10

A default value or `Closure` may also be passed as the third parameter to the method. This value will be returned if the given value is blank:

    $result = transform(null, $callback, 'The value is blank');

    // The value is blank

<a name="method-validator"></a>
#### `validator()` {#collection-method}

`validator`関数は、指定した引数で新しい[バリデータ](/docs/{{version}}/validation)インスタンスを生成します。利便のため、`Validator`ファサードを代わりに使うこともできます。

    $validator = validator($data, $rules, $messages);

<a name="method-value"></a>
#### `value()` {#collection-method}

The `value` function returns the value it is given. However, if you pass a `Closure` to the function, the `Closure` will be executed then its result will be returned:

    $result = value(true);

    // true

    $result = value(function () {
        return false;
    });

    // false

<a name="method-view"></a>
#### `view()` {#collection-method}

`view`関数は[view](/docs/{{version}}/views)インスタンスを返します。

    return view('auth.login');

<a name="method-with"></a>
#### `with()` {#collection-method}

The `with` function returns the value it is given. If a `Closure` is passed as the second argument to the function, the `Closure` will be executed its result will be returned:

    $callback = function ($value) {
        return (is_numeric($value)) ? $value * 2 : 0;
    };

    $result = with(5, $callback);

    // 10

    $result = with(null, $callback);

    // 0

    $result = with(5, null);

    // 5
