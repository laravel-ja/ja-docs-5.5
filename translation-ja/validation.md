# バリデーション

- [イントロダクション](#introduction)
- [クイックスタート](#validation-quickstart)
    - [ルート定義](#quick-defining-the-routes)
    - [コントローラ作成](#quick-creating-the-controller)
    - [バリデーションロジック作成](#quick-writing-the-validation-logic)
    - [バリデーションエラー表示](#quick-displaying-the-validation-errors)
    - [オプションフィールドに対する注意](#a-note-on-optional-fields)
- [フォームリクエストバリデーション](#form-request-validation)
    - [フォームリクエスト作成](#creating-form-requests)
    - [フォームリクエストの認可](#authorizing-form-requests)
    - [エラーメッセージのカスタマイズ](#customizing-the-error-messages)
- [バリデータの生成](#manually-creating-validators)
    - [自動リダイレクト](#automatic-redirection)
    - [名前付きエラーバッグ](#named-error-bags)
    - [バリデーション後フック](#after-validation-hook)
- [エラーメッセージ操作](#working-with-error-messages)
    - [カスタムエラーメッセージ](#custom-error-messages)
- [使用可能なバリデーションルール](#available-validation-rules)
- [条件付きの追加ルール](#conditionally-adding-rules)
- [配列のバリデーション](#validating-arrays)
- [カスタムバリデーションルール](#custom-validation-rules)
    - [ルールオブジェクトの使用](#using-rule-objects)
    - [拡張の使用](#using-extensions)

<a name="introduction"></a>
## イントロダクション

Laravelは入力されたデータに対するバリデーションの様々なアプローチを提供しています。Laravelの基本コントローラークラスはパワフルでバラエティー豊かなバリデーションルールを使いHTTPリクエストをバリデーションするために便利な手法を提供している、`ValidatesRequests`トレイトをデフォルトで使用しています。

<a name="validation-quickstart"></a>
## クイックスタート

パワフルなバリデーション機能を学ぶために、フォームバリデーションとユーザーにエラーメッセージを表示する完全な例を見てください。

<a name="quick-defining-the-routes"></a>
### ルート定義

まず、`routes/web.php`ファイルに以下のルートを定義してあるとしましょう。

    Route::get('post/create', 'PostController@create');

    Route::post('post', 'PostController@store');

もちろん、`GET`のルートは新しいブログポストを作成するフォームをユーザーへ表示し、`POST`ルートで新しいブログポストをデータベースへ保存します。

<a name="quick-creating-the-controller"></a>
### コントローラー作成

次に、これらのルートを処理する簡単なコントローラーを見てみましょう。今のところ`store`メソッドは空のままです。

    <?php

    namespace App\Http\Controllers;

    use Illuminate\Http\Request;
    use App\Http\Controllers\Controller;

    class PostController extends Controller
    {
        /**
         * 新ブログポスト作成フォームの表示
         *
         * @return Response
         */
        public function create()
        {
            return view('post.create');
        }

        /**
         * 新しいブログポストの保存
         *
         * @param  Request  $request
         * @return Response
         */
        public function store(Request $request)
        {
            // ブログポストのバリデーションと保存コード…
        }
    }

<a name="quick-writing-the-validation-logic"></a>
### バリデーションロジック

これで新しいブログポストに対するバリデーションロジックを`store`メソッドに埋め込む準備ができました。そのためには、`Illuminate\Http\Request`オブジェクトが提供する、`validate`メソッドを使います。バリデーションルールに成功すると、コードは通常通り続けて実行されます。逆にバリデーションに失敗すると、例外が投げられ、ユーザーに対し自動的に適切なエラーレスポンスが返されます。伝統的なHTTPリクエストの場合は、リダイレクトレスポンスが生成され、一方でAJAXリクエストにはJSONレスポンスが返されます。

`validate`メソッドをもっとよく理解するため、`store`メソッドに取り掛かりましょう。

    /**
     * 新ブログポストの保存
     *
     * @param  Request  $request
     * @return Response
     */
    public function store(Request $request)
    {
        $request->validate([
            'title' => 'required|unique:posts|max:255',
            'body' => 'required',
        ]);

        // ブログポストは有効なので、データベースに保存する処理…
    }

ご覧のとおりに、実行したいバリデーションルールを`validate`メソッドへ渡すだけです。繰り返しますが、バリデーションに失敗すれば、適切なレスポンスが自動的に生成されます。バリデーションに成功すれば、コントローラは続けて通常通り実行されます。

#### 最初のバリデーション失敗時に停止

最初のバリデーションに失敗したら、残りのバリデーションルールの判定を停止したいことも、時々あります。このためには、`bail`ルールを使ってください。

    $request->validate([
        'title' => 'bail|required|unique:posts|max:255',
        'body' => 'required',
    ]);

この例の場合、`title`属性の`unique`ルールに失敗すると、`max`ルールはチェックされません。ルールは指定した順番にバリデートされます。

#### ネストした属性の注意点

HTTPリクエストに「ネスト」したパラメーターが含まれている場合、バリデーションルールは「ドット」記法により指定します。

    $request->validate([
        'title' => 'required|unique:posts|max:255',
        'author.name' => 'required',
        'author.description' => 'required',
    ]);

<a name="quick-displaying-the-validation-errors"></a>
### バリデーションエラー表示

ではやって来たリクエストの入力が指定したバリデーションルールに当てはまらなかった場合はどうなるんでしょう？　既に説明した通り、Laravelは自動的にユーザーを以前のページヘリダイレクトします。付け加えて、バリデーションエラーは全部自動的に[フラッシュデータとしてセッション](/docs/{{version}}/session#flash-data)へ保存されます。

`GET`ルートのビューへエラーメッセージを明示的に結合する必要がないことに注目してください。これはつまり、Laravelはいつもセッションデータの中にエラーの存在をチェックしており、見つけた場合は自動的に結合しているからです。`$errors`変数は`Illuminate\Support\MessageBag`のインスタンスです。このオブジェクトの詳細は、[ドキュメント](#working-with-error-messages)を参照してください。

> {tip} `$errors`変数は`web`ミドルウェアグループに所属する、`Illuminate\View\Middleware\ShareErrorsFromSession`ミドルウェアによりビューに結合されます。**このミドルウェアが適用される場合は、いつでもビューの中で`$errors`変数が使えます。**`$errors`変数はいつでも定義済みであると想定でき、安心して使えます。

この例では、バリデーションに失敗すると、エラーメッセージをビューで表示できるように、コントローラーの`create`メソッドにリダイレクトされることになります。

    <!-- /resources/views/post/create.blade.php -->

    <h1>ポスト作成</h1>

    @if ($errors->any())
        <div class="alert alert-danger">
            <ul>
                @foreach ($errors->all() as $error)
                    <li>{{ $error }}</li>
                @endforeach
            </ul>
        </div>
    @endif

    <!-- ポスト作成フォーム -->

<a name="a-note-on-optional-fields"></a>
### オプションフィールドに対する注意

Laravelは`TrimStrings`と`ConvertEmptyStringsToNull`ミドルウェアをアプリケーションのデフォルトグローバルミドルウェアスタックに含んでいます。これらのミドルウェアは`App\Http\Kernel`クラスでリストされています。このため、バリデータが`null`値が無効であると判定されないように、オプションフィールドへ`nullable`を付ける必要がたびたび起きるでしょう。

    $request->validate([
        'title' => 'required|unique:posts|max:255',
        'body' => 'required',
        'publish_at' => 'nullable|date',
    ]);

上記の例の場合、`publish_at`フィールドが`null`か、有効な日付表現であることを指定しています。ルール定義に`nullable`が追加されないと、バリデータは`null`を無効な日付として判定します。

<a name="quick-ajax-requests-and-validation"></a>
#### AJAXリクエストとバリデーション

この例ではアプリケーションにデータを送るために伝統的なフォームを使いました。しかし、多くのアプリケーションでAJAXリクエストが使用されています。AJAXリクエストに`validate`メソッドを使う場合、Laravelはリダイレクトレスポンスを生成しません。代わりにバリデーションエラーを全部含んだJSONレスポンスを生成します。このJSONレスポンスは422 HTTPステータスコードで送られます。

<a name="form-request-validation"></a>
## フォームリクエストバリデーション

<a name="creating-form-requests"></a>
### フォームリクエスト作成

より複雑なバリデーションのシナリオでは、「フォームリクエスト」を生成したほうが良いでしょう。フォームリクエストは、バリデーションロジックを含んだカスタムリクエストクラスです。フォームリクエストクラスを作成するには、`make:request` Artisan CLIコマンドを使用します。

    php artisan make:request StoreBlogPost

生成されたクラスは、`app/Http/Request`ディレクトリへ設置されます。このディレクトリが存在しなくても、`make:request`コマンドを実行すれば作成されます。では、バリデーションルールを少し`rules`メソッドへ追加してみましょう。

    /**
     * リクエストに適用するバリデーションルールを取得
     *
     * @return array
     */
    public function rules()
    {
        return [
            'title' => 'required|unique:posts|max:255',
            'body' => 'required',
        ];
    }

では、どのようにバリデーションルールを実行するのでしょうか？必要なのは、コントローラーのメソッドで、このリクエストをタイプヒントで指定することです。やって来たフォームリクエストはコントローラーメソッドが呼び出される前にバリデーションを行います。つまり、コントローラーにバリデーションロジックを取っ散らかす必要はありません。

    /**
     * ブログポストの保存
     *
     * @param  StoreBlogPost  $request
     * @return Response
     */
    public function store(StoreBlogPost $request)
    {
        // 送信されたリクエストは正しい
    }

バリデーションに失敗すると、前のアドレスにユーザーを戻すために、リダイレクトレスポンスが生成されます。エラーも表示できるように、フラッシュデーターとしてセッションに保存されます。もしリクエストがAJAXリクエストであれば、バリデーションエラーを表現するJSONを含んだ、422ステータスコードのHTTPレスポンスがユーザーに返されます。

#### フォームリクエストへのAfterフックを追加

フォームリクエストへ"after"フックを追加したい場合は、`withValidator`メソッドを使用します。このメソッドは完全に構築されたバリデータを受け取るため、バリデーションルールが実際に評価される前に、バリデータのどんなメソッドも呼び出すことができます。

    /**
     * バリデータインスタンスの設定
     *
     * @param  \Illuminate\Validation\Validator  $validator
     * @return void
     */
    public function withValidator($validator)
    {
        $validator->after(function ($validator) {
            if ($this->somethingElseIsInvalid()) {
                $validator->errors()->add('field', 'Something is wrong with this field!');
            }
        });
    }

<a name="authorizing-form-requests"></a>
### フォームリクエストの認可

フォームリクエストクラスは`authorize`メソッドも用意しています。このメソッドでは認証されているユーザーが、指定されたリソースを更新する権限を実際に持っているのかを確認します。たとえば、ユーザーが更新しようとしているブログコメントを実際に所有しているかを判断するとしましょう。

    /**
     * ユーザーがこのリクエストの権限を持っているかを判断する
     *
     * @return bool
     */
    public function authorize()
    {
        $comment = Comment::find($this->route('comment'));

        return $comment && $this->user()->can('update', $comment);
    }

全リフォームリクエストはLaravelのベースリクエストクラスを拡張していますので、現在認証されているユーザーへアクセスする、`user`メソッドが使えます。また、上記例中の`route`メソッドの呼び出しにも、注目してください。例えば`{comment}`パラメーターのような、呼び出しているルートで定義されているURIパラメータにもアクセスできます。

    Route::post('comment/{comment}');

`authorize`メソッドが`false`を返すと、403ステータスコードのHTTPレスポンスが自動的に返され、コントローラーメソッドは実行されません。

アプリケーションの他の場所で認証のロジックを行おうと設計しているのでしたら、シンプルに`authorize`メソッドから`true`を返してください。

    /**
     * ユーザーがこのリクエストの権限を持っているかを判断する
     *
     * @return bool
     */
    public function authorize()
    {
        return true;
    }

<a name="customizing-the-error-messages"></a>
### エラーメッセージのカスタマイズ

フォームリクエストにより使用されているメッセージは`message`メソッドをオーバーライドすることによりカスタマイズできます。このメソッドから属性／ルールと対応するエラーメッセージのペアを配列で返してください。

    /**
     * 定義済みバリデーションルールのエラーメッセージ取得
     *
     * @return array
     */
    public function messages()
    {
        return [
            'title.required' => 'A title is required',
            'body.required'  => 'A message is required',
        ];
    }

<a name="manually-creating-validators"></a>
## バリデータの生成

リクエストの`validate`メソッドを使用したくない場合は、`Validator`[ファサード](/docs/{{version}}/facades)を使用し、バリデタインスタンスを生成する必要があります。 このファサードの`make`メソッドで、新しいバリデタインスタンスを生成します。

    <?php

    namespace App\Http\Controllers;

    use Validator;
    use Illuminate\Http\Request;
    use App\Http\Controllers\Controller;

    class PostController extends Controller
    {
        /**
         * 新しいブログポストの保存
         *
         * @param  Request  $request
         * @return Response
         */
        public function store(Request $request)
        {
            $validator = Validator::make($request->all(), [
                'title' => 'required|unique:posts|max:255',
                'body' => 'required',
            ]);

            if ($validator->fails()) {
                return redirect('post/create')
                            ->withErrors($validator)
                            ->withInput();
            }

            // ブログポストの保存処理…
        }
    }

`make`メソッドの第１引数は、バリデーションを行うデータです。第２引数はそのデータに適用するバリデーションルールです。

リクエストのバリデーションが失敗するかを確認した後、セッションにエラーメッセージをフラッシュデータとして保存するために`withErrors`メソッドが利用できます。このメソッドを使うと、簡単にユーザーに情報を表示できるようにするため、リダイレクトの後でビューに対し`$errors`変数を自動的に共有します。`withErrors`メソッドはバリデーターか`MessageBag`、PHPの配列を受け取ります。

<a name="automatic-redirection"></a>
### 自動リダイレクト

バリデタインスタンスを自分で生成したいが、リクエストの`validate`メソッドが提供する自動リダイレクト機能を使用したい場合は、生成したバリデタインスタンスの`validate`メソッドを呼び出すこともできます。バリデーションに失敗すると、ユーザーは自動的にリダイレクトされるか、AJAXリクエストの場合は、JSONリクエストが返されます。

    Validator::make($request->all(), [
        'title' => 'required|unique:posts|max:255',
        'body' => 'required',
    ])->validate();

<a name="named-error-bags"></a>
### 名前付きエラーバッグ

１ページの中に複数のフォームを入れている場合は、特定のフォームのエラーメッセージを受け取れるように、`MessageBag`へ名前を付けてください。`withErrors`の第２引数に名前を渡すだけです。

    return redirect('register')
                ->withErrors($validator, 'login');

`$errors`変数を使い、名前を付けた`MessageBag`インスタンスへアクセスできます。

    {{ $errors->login->first('email') }}

<a name="after-validation-hook"></a>
### バリデーション後のフック

バリデータにはさらに、バリデーションが終了した時点で実行するコールバックを付け加えられます。これにより、追加のバリデーションを行い、さらにエラーメッセージコレクションにエラーメッセージを追加することが簡単にできます。バリデータインスタンスの`after`メソッドを使ってみましょう。

    $validator = Validator::make(...);

    $validator->after(function ($validator) {
        if ($this->somethingElseIsInvalid()) {
            $validator->errors()->add('field', 'Something is wrong with this field!');
        }
    });

    if ($validator->fails()) {
        //
    }

<a name="working-with-error-messages"></a>
## エラーメッセージの操作

`Validator`インスタンスの`errors`メソッドを呼び出すと、エラーメッセージを操作する便利なメソッドを数揃えた、`Illuminate\Support\MessageBag`インスタンスを受け取ります。自動的に作成され、全てのビューで使用できる`$errors`変数も、`MessageBag`クラスのインスタンスです。

#### 指定フィールドの最初のエラーメッセージ取得

指定したフィールドの最初のエラーメッセージを取得するには、`first`メソッドを使います。

    $errors = $validator->errors();

    echo $errors->first('email');

#### 指定フィールドの全エラーメッセージ取得

指定したフィールドの全エラーメッセージを配列で取得したい場合は、`get`メソッドを使います。

    foreach ($errors->get('email') as $message) {
        //
    }

配列形式のフィールドをバリデーションする場合は、`*`文字を使用し、各配列要素の全メッセージを取得できます。

    foreach ($errors->get('attachments.*') as $message) {
        //
    }

#### 全フィールドの全エラーメッセージ取得

全フィールドの全メッセージの配列を取得したい場合は、`all`メソッドを使います。

    foreach ($errors->all() as $message) {
        //
    }

#### 指定フィールドのメッセージ存在確認

`has`メソッドは、指定したフィールドのエラーメッセージが存在しているかを判定するために使います。

    if ($errors->has('email')) {
        //
    }

<a name="custom-error-messages"></a>
### カスタムエラーメッセージ

必要であればバリデーションでデフォルトのメッセージの代わりに、カスタムエラーメッセージを使うことができます。カスタムメッセージを指定するにはいくつか方法があります。最初の方法は`Validator::make`メソッドの第３引数として、カスタムメッセージを渡す方法です。

    $messages = [
        'required' => 'The :attribute field is required.',
    ];

    $validator = Validator::make($input, $rules, $messages);

この例中の`attribute`プレースホルダーはバリデーション対象のフィールドの名前に置き換えられます。バリデーションメッセージ中で他のプレースホルダーを使うこともできます。例を見てください。

    $messages = [
        'same'    => 'The :attribute and :other must match.',
        'size'    => 'The :attribute must be exactly :size.',
        'between' => 'The :attribute value :input is not between :min - :max.',
        'in'      => 'The :attribute must be one of the following types: :values',
    ];

#### 指定フィールドにカスタムメッセージ指定

時々特定のフィールドに対するカスタムエラーメッセージを指定したい場合があります。「ドット」記法を使用し行います。属性名が最初で、続いてルールをつなげます。

    $messages = [
        'email.required' => 'We need to know your e-mail address!',
    ];

<a name="localization"></a>
#### 言語ファイルでカスタムメッセージ指定

多くの場合、`Validator`に直接カスタムメッセージを渡すよりは言語ファイルに指定したいですよね。ならば`resources/lang/xx/validation.php`言語ファイルの`custom`配列にメッセージを追加してください。

    'custom' => [
        'email' => [
            'required' => 'We need to know your e-mail address!',
        ],
    ],

#### 言語ファイル中のカスタム属性名の指定

バリデーションメッセージの`:attribute`部分をカスタムアトリビュート名で置き換えたい場合は、`resources/lang/xx/validation.php`言語ファイルの`attributes`配列でカスタム名を指定してください。

    'attributes' => [
        'email' => 'email address',
    ],

<a name="available-validation-rules"></a>
## 使用可能なバリデーションルール

使用可能な全バリデーションルールとその機能の一覧です。

<style>
    .collection-method-list > p {
        column-count: 3; -moz-column-count: 3; -webkit-column-count: 3;
        column-gap: 2em; -moz-column-gap: 2em; -webkit-column-gap: 2em;
    }

    .collection-method-list a {
        display: block;
    }
</style>

<div class="collection-method-list" markdown="1">

[受け入れ](#rule-accepted)
[アクティブなURL](#rule-active-url)
[（日付）より後](#rule-after)
[（日付）以降](#rule-after-or-equal)
[アルファベット](#rule-alpha)
[アルファベット記号](#rule-alpha-dash)
[アルファベット数字](#rule-alpha-num)
[配列](#rule-array)
[（日付）より前](#rule-before)
[（日付）以前](#rule-before-or-equal)
[範囲](#rule-between)
[論理](#rule-boolean)
[確認](#rule-confirmed)
[日付](#rule-date)
[同一日付](#rule-date-equals)
[日付形式](#rule-date-format)
[相違](#rule-different)
[桁指定数値](#rule-digits)
[桁範囲指定数値](#rule-digits-between)
[寸法(画像ファイル)](#rule-dimensions)
[別々](#rule-distinct)
[メールアドレス](#rule-email)
[存在（データベース）](#rule-exists)
[ファイル](#rule-file)
[充満](#rule-filled)
[画像（ファイル)](#rule-image)
[内包](#rule-in)
[配列内](#rule-in-array)
[整数](#rule-integer)
[IPアドレス](#rule-ip)
[JSON](#rule-json)
[最大値](#rule-max)
[MIMEタイプ](#rule-mimetypes)
[MIMEタイプ(ファイル拡張子)](#rule-mimes)
[最小値](#rule-min)
[NULL可能](#rule-nullable)
[非内包](#rule-not-in)
[数値](#rule-numeric)
[存在](#rule-present)
[正規表現](#rule-regex)
[必須](#rule-required)
[指定フィールド値一致時必須](#rule-required-if)
[指定フィールド値非一致時必須](#rule-required-unless)
[指定フィールド存在時必須](#rule-required-with)
[全指定フィールド存在時必須](#rule-required-with-all)
[指定フィールド非存在時必須](#rule-required-without)
[全指定フィールド非存在時必須](#rule-required-without-all)
[同一](#rule-same)
[サイズ](#rule-size)
[文字列](#rule-string)
[タイムゾーン](#rule-timezone)
[一意（データベース）](#rule-unique)
[URL](#rule-url)

</div>

<a name="rule-accepted"></a>
#### accepted

そのフィールドが**yes**、**on**、**1**、**true**であることをバリデートします。これは「サービス利用規約」同意のバリデーションに便利です。

<a name="rule-active-url"></a>
#### active_url

フィールドが、`dns_get_record` PHP関数により、有効なAかAAAAレコードであることをバリデートします。

<a name="rule-after"></a>
#### after:_日付_

フィールドの値が与えられた日付より後であるかバリデーションします。日付はPHPの`strtotime`関数で処理されます。

    'start_date' => 'required|date|after:tomorrow'

`strtotime`により評価される日付文字列を渡す代わりに、その日付と比較する他のフィールドを指定することもできます。

    'finish_date' => 'required|date|after:start_date'

<a name="rule-after-or-equal"></a>
#### after\_or\_equal:_日付_

フィールドが指定した日付以降であることをバリデートします。詳細は[after](#rule-after)ルールを参照してください。

<a name="rule-alpha"></a>
#### alpha

フィールドが全部アルファベット文字であることをバリデートします。

<a name="rule-alpha-dash"></a>
#### alpha_dash

フィールドが全部アルファベット文字と数字、ダッシュ(-)、下線(_)であることをバリデートします。

<a name="rule-alpha-num"></a>
#### alpha_num

フィールドが全部アルファベット文字と数字であることをバリデートします。

<a name="rule-array"></a>
#### array

フィールドが配列タイプであることをバリデートします。

<a name="rule-before"></a>
#### before:_日付_

フィールドが指定された日付より前であることをバリデートします。日付はPHPの`strtotime`関数で処理されます。

<a name="rule-before-or-equal"></a>
#### before\_or\_equal:_日付_

フィールドが指定した日付以前であることをバリデートします。日付はPHPの`strtotime`関数で処理されます。

<a name="rule-between"></a>
#### between:_min_,_max_

フィールドが指定された*最小値*と*最大値*の間のサイズであることをバリデートします。[`size`](#rule-size)ルールと同様の判定方法で、文字列、数値、配列、ファイルが評価されます。

<a name="rule-boolean"></a>
#### boolean

フィールドが論理値として有効であることをバリデートします。受け入れられる入力は、`true`、`false`、`1`、`0`、`"1"`、`"0"`です。

<a name="rule-confirmed"></a>
#### confirmed

フィールドがそのフィールド名＋`_confirmation`フィールドと同じ値であることをバリデートします。例えば、バリデーションするフィールドが`password`であれば、同じ値の`password_confirmation`フィールドが入力に存在していなければなりません。

<a name="rule-date"></a>
#### date

パリデーションされる値はPHP関数の`strtotime`を使用し確認されます。

<a name="rule-date-equals"></a>
#### date_equals:_日付_

バリデーションされる値が、指定した日付と同じことをバリデートします。日付は、PHPの`strtotime`関数へ渡されます。

<a name="rule-date-format"></a>
#### date\_format:_フォーマット_

バリデーションされる値が**フォーマット**定義と一致するか確認します。バリデーション時には`date`か`date_format`の**どちらか**を使用しなくてはならず、両方はできません。

<a name="rule-different"></a>
#### different:_フィールド_

フィールドが指定された**フィールド**と異なった値を指定されていることをバリデートします。

<a name="rule-digits"></a>
#### digits:_値_

フィールドが**数値**で、**値**の桁数であることをバリデートします。

<a name="rule-digits-between"></a>
#### digits_between:_最小値_,_最大値_

フィールドが**整数で**、桁数が**最小値**から**最大値**の間であることをバリデートします。

<a name="rule-dimensions"></a>
#### dimensions

バリデーション対象のファイルが、パラメータにより指定されたサイズに合致することをバリデートします。

    'avatar' => 'dimensions:min_width=100,min_height=200'

使用可能なパラメータは、_min\_width_、_max\_width_、_min\_height_、_max\_height_、_width_、_height_、_ratio_です。

_ratio_制約は、横／縦比を表します。`3/2`という指定も、`1.5`のようにfloatでの指定も可能です。

    'avatar' => 'dimensions:ratio=3/2'

このルールは多くの引数を要求するので、`Rule::dimensions`メソッドを使い、記述的にこのルールを構築してください。

    use Illuminate\Validation\Rule;

    Validator::make($data, [
        'avatar' => [
            'required',
            Rule::dimensions()->maxWidth(1000)->maxHeight(500)->ratio(3 / 2),
        ],
    ]);

<a name="rule-distinct"></a>
#### distinct

対象が配列の時、フィールドに重複した値がないことをバリデートします。

    'foo.*.id' => 'distinct'

<a name="rule-email"></a>
#### email

フィールドがメールアドレスとして正しいことをバリデートします。

<a name="rule-exists"></a>
#### exists:_テーブル_,_カラム_

フィールドの値が、指定されたデータベーステーブルに存在することをバリデートします。

#### 基本的なExistsルールの使用法

    'state' => 'exists:states'

#### カスタムカラム名の指定

    'state' => 'exists:states,abbreviation'

`exists`クエリにデータベース接続を指定する必要があることも多いでしょう。「ドット」記法を用い、テーブル名の前に接続名を付けることで、指定可能です。

    'email' => 'exists:connection.staff,email'

バリデーションルールで実行されるクエリをカスタマイズしたい場合は、ルールをスラスラと定義できる`Rule`クラスを使ってください。下の例では、`|`文字を区切りとして使用する代わりに、バリデーションルールを配列として指定しています。

    use Illuminate\Validation\Rule;

    Validator::make($data, [
        'email' => [
            'required',
            Rule::exists('staff')->where(function ($query) {
                $query->where('account_id', 1);
            }),
        ],
    ]);

<a name="rule-file"></a>
#### file

フィールドがアップロードに成功したファイルであることをバリデートします。

<a name="rule-filled"></a>
#### filled

フィールドが存在する場合、空でないことをバリデートします。

<a name="rule-image"></a>
#### image

フィールドで指定されたファイルが画像(jpg、png、bmp、gif、svg)であることをバリデートします。

<a name="rule-in"></a>
#### in:foo,bar...

フィールドが指定したリストの中の値に含まれていることをバリデートします。このルールを使用するために配列を`implode`する必要が多くなりますので、ルールを記述的に構築するには、`Rule::in`メソッドを使ってください。

    use Illuminate\Validation\Rule;

    Validator::make($data, [
        'zones' => [
            'required',
            Rule::in(['first-zone', 'second-zone']),
        ],
    ]);

<a name="rule-in-array"></a>
#### in_array:_別のフィールド_

フィールドが、*他のフィールド*の値のどれかであることをバリデートします。

<a name="rule-integer"></a>
#### integer

フィールドが整数値であることをバリデートします。

<a name="rule-ip"></a>
#### ip

フィールドがIPアドレスの形式として正しいことをバリデートします。

#### ipv4

フィールドがIPv4アドレスの形式として正しいことをバリデートします。

#### ipv6

フィールドがIPv6アドレスの形式として正しいことをバリデートします。

<a name="rule-json"></a>
#### json

フィールドが有効なJSON文字列であることをバリデートします。

<a name="rule-max"></a>
#### max:_値_

フィールドが最大値として指定された**値**以下であることをバリデートします。[`size`](#rule-size)ルールと同様の判定方法で、文字列、数値、配列、ファイルが評価されます。

<a name="rule-mimetypes"></a>
#### mimetypes:*text/plain*,...

フィールドが指定されたMIMEタイプのどれかであることをバリデートします。

    'video' => 'mimetypes:video/avi,video/mpeg,video/quicktime'

アップロードされたファイルのMIMEタイプを決定するために、フレームワークはその内容を読み込み、MIMEタイプを推測します。クライアントが提供するMIMEタイプとは異なります。

<a name="rule-mimes"></a>
#### mimes:_foo_,_bar_,...

フィールドで指定されたファイルが拡張子のリストの中のMIMEタイプのどれかと一致することをバリデートします。

#### mimesルールの基本的な使用法

    'photo' => 'mimes:jpeg,bmp,png'

拡張子だけを限定する必要があるとしても、このルールはファイルのMIMEタイプに基づき、ファイルの内容を読み、MIMEタイプを推測することでバリデーションを行います。

MIMEタイプと対応する拡張子の完全なリストは、[https://svn.apache.org/repos/asf/httpd/httpd/trunk/docs/conf/mime.types](http://svn.apache.org/repos/asf/httpd/httpd/trunk/docs/conf/mime.types)で確認できます。

<a name="rule-min"></a>
#### min:_値_

フィールドが最小値として指定された**値**以上であることをバリデートします。[`size`](#rule-size)ルールと同様の判定方法で、文字列、数値、配列、ファイルが評価されます。

<a name="rule-nullable"></a>
#### nullable

フィールドが`null`であることをバリデートします。これは`null`値を含無ことができる文字列や整数のようなプリミティブをバリデーションするときに特に便利です。

<a name="rule-not-in"></a>
#### not_in:_foo_,_bar_,...

フィールドが指定された値のリスト中に含まれていないことをバリデートします。`Rule::notIn`メソッドのほうが、ルールの構成が読み書きしやすいでしょう。

    use Illuminate\Validation\Rule;

    Validator::make($data, [
        'toppings' => [
            'required',
            Rule::notIn(['sprinkles', 'cherries']),
        ],
    ]);

<a name="rule-numeric"></a>
#### numeric

フィールドは数値であることをバリデートします。

<a name="rule-present"></a>
#### present

フィールドが存在していることをバリデートしますが、存在していれば空を許します。

<a name="rule-regex"></a>
#### regex:_正規表現_

フィールドが指定された正規表現にマッチすることをバリデートします。

**注目：** `regex`パターンを使用する場合はルールをパイプ（縦棒）で区切らず、配列で指定する必要があります。特に正規表現に縦棒を含んでいる場合に該当します。

<a name="rule-required"></a>
#### required

フィールドが入力データに存在しており、かつ空でないことをバリデートします。フィールドは以下の条件の場合、「空」であると判断されます。

<div class="content-list" markdown="1">

- 値が`null`である。
- 値が空文字列である。
- 値が空の配列か、空の`Countable`オブジェクトである。
- 値がパスのないアップロード済みファイルである。

</div>

<a name="rule-required-if"></a>
#### required\_if:_他のフィールド_,_値_,...

**他のフィールド**が**値**のどれかと一致している場合、このフィールドが存在し、かつ空でないことをバリデートします。

<a name="rule-required-unless"></a>
#### required\_unless:_他のフィールド_,_値_,...

**他のフィールド**が**値**のどれとも一致していない場合、このフィールドが存在し、かつ空でないことをバリデートします。

<a name="rule-required-with"></a>
#### required\_with:_foo_,_bar_,...

指定した他のフィールドが**一つでも存在している**場合、このフィールドが存在し、かつ空でないことをバリデートします。

<a name="rule-required-with-all"></a>
#### required\_with_all:_foo_,_bar_,...

指定した他のフィールドが**すべて存在している**場合、このフィールドが存在し、かつ空でないことをバリデートします。

<a name="rule-required-without"></a>
#### required\_without:_foo_,_bar_,...

指定した他のフィールドの**どれか一つでも存在していない**場合、このフィールドが存在し、かつ空でないことをバリデートします。

<a name="rule-required-without-all"></a>
#### required\_without_all:_foo_,_bar_,...

指定した他のフィールドが**すべて存在していない**場合、このフィールドが存在し、かつ空でないことをバリデートします。

<a name="rule-same"></a>
#### same:_フィールド_

フィールドが、指定されたフィールドと同じ値であることをバリデートします。

<a name="rule-size"></a>
#### size:_値_

フィールドは指定された*値*と同じサイズであることをバリデートします。文字列の場合、*値*は文字長です。数値項目の場合、*値*は整数値です。配列の場合、*値*は配列の個数(`count`)です。ファイルの場合、*値*はキロバイトのサイズです。

<a name="rule-string"></a>
#### string

フィルードは文字列タイプであることをバリデートします。フィールドが`null`であることも許す場合は、そのフィールドに`nullable`ルールも指定してください。

<a name="rule-timezone"></a>
#### timezone

`timezone_identifiers_list` PHP関数の値に基づき、フィールドがタイムゾーンとして識別されることをバリデートします。

<a name="rule-unique"></a>
#### unique:_テーブル_,_カラム_,_除外ID_,_IDカラム_

フィールドは指定されたデータベーステーブルで一意であることをバリデートします。`column`オプションが指定されない場合、フィールド名が使用されます。

**カスタムカラム名の指定**

    'email' => 'unique:users,email_address'

**カスタムデータベース接続**

場合により、バリデーターにより生成されるデータベースクエリに、カスタム接続を設定する必要があるかもしれません。上記のバリデーションルール、`unique:users`ではクエリに対し、デフォルトデータベース接続が使用されます。これをオーバーライドするにはドット記法で、接続に続けテーブル名を指定してください。

    'email' => 'unique:connection.users,email_address'

**指定されたIDのuniqueルールを無視する**

uniqueチェックで指定したIDを除外したい場合があります。たとえばユーザー名、メールアドレス、それと住所の「プロフィール更新」の状況を考えてください。もちろん、メールアドレスは一意であることを確認したいと思います。しかし、もしユーザーが名前フィールドだけ変更し、メールフィールドを変更しなければ、そのユーザーが既にそのメールアドレスの所有者として登録されているために起きるバリデーションエラーを避けたいと思うでしょう。

バリデータにユーザーIDを無視するように指示するには、ルールをスラスラと定義できる`Rule`クラスを使います。以下の例の場合、さらにルールを`|`文字を区切りとして使用する代わりに、バリデーションルールを配列として指定しています。

    use Illuminate\Validation\Rule;

    Validator::make($data, [
        'email' => [
            'required',
            Rule::unique('users')->ignore($user->id),
        ],
    ]);

もしテーブルの主キーとして、`id`以外のカラム名を使用している場合、`ignore`メソッドを呼び出す時に、カラムの名前を指定してください。

    'email' => Rule::unique('users')->ignore($user->id, 'user_id')

**追加のWHERE節を付け加える**

`where`メソッドを使用し、クエリをカスタマイズすることにより、追加のクエリ制約を指定することも可能です。例として、`account_id`が`1`であることを確認する制約を追加してみましょう。

    'email' => Rule::unique('users')->where(function ($query) {
        return $query->where('account_id', 1);
    })

<a name="rule-url"></a>
#### url

フィールドが有効なURLであることをバリデートします。

<a name="conditionally-adding-rules"></a>
## 条件付きでルールを追加する

#### 項目存在時のバリデーション

ある状況では、そのフィールドが入力配列の中に存在する場合**のみ**、バリデーションを実行したいことがあると思います。これを簡単に行うには、`sometimes`ルールを追加してください。

    $v = Validator::make($data, [
        'email' => 'sometimes|required|email',
    ]);

上の例では`email`フィールドが、`$data`配列の中に存在している場合のみバリデーションが実行されます。

> {tip} フィールドが常に存在しているが、空であることをバリデートする場合は、[この追加フィールドに対する注意事項](#a-note-on-optional-fields)を確認してください。

#### 複雑な条件のバリデーション

時々もっと複雑な条件のロジックによりバリデーションルールを追加したい場合もあります。たとえば他のフィールドが１００より大きい場合のみ、指定したフィールドが入力されているかをバリデートしたいときなどです。もしくは２つのフィールドのどちらか一方が存在する場合は、両方共に値を指定する必要がある場合です。こうしたルールを付け加えるのも面倒ではありません。最初に`Validator`インスタンスを生成するのは、**固定ルール**の場合と同じです。

    $v = Validator::make($data, [
        'email' => 'required|email',
        'games' => 'required|numeric',
    ]);

ゲームコレクターのためのWebアプリケーションだと仮定しましょう。ゲームコレクターがアプリケーションに登録する時に、100ゲーム以上所有しているのであれば、なぜそんなに多く持っているのか理由を説明してもらいます。たとえば販売店を運営しているのかも知れませんし、ただ収集家なのかも知れません。この条件付きの要求を追加するために`Validator`インスタンスへ、`sometimes`メソッドを使用してください。

    $v->sometimes('reason', 'required|max:500', function ($input) {
        return $input->games >= 100;
    });

`sometimes`メソッドの最初の引数は条件付きでバリデーションを行うフィールドの名前です。2つ目の引数は追加したいルールです。3つ目の引数にクロージャが渡され、`true`を返したらそのルールは追加されます。このメソッドにより複雑な条件付きのバリデーションが簡単に作成できます。一度に多くのフィールドに、条件付きバリデーションを追加することもできます。

    $v->sometimes(['reason', 'cost'], 'required', function ($input) {
        return $input->games >= 100;
    });

> {tip} クロージャに渡される`$input`パラメーターは`Illuminate\Support\Fluent`のインスタンスで、フィールドと入力値にアクセスするためのオブジェクトです。

<a name="validating-arrays"></a>
## 配列のバリデーション

フォーム入力フィールドの配列をバリデーションするのに苦労する必要はありません。配列中の属性をバリデーションするために「ドット記法」が使えます。たとえば、送信されたHTTPリクエストに、`photos[profile]`フィールドが含まれているかをバリデーションするには、次のように行います。

    $validator = Validator::make($request->all(), [
        'photos.profile' => 'required|image',
    ]);

配列の各要素をバリデーションすることもできます。たとえば、配列中の各メールアドレスの入力フィールドが、一位であることを確認するには、以下のように行います。

    $validator = Validator::make($request->all(), [
        'person.*.email' => 'email|unique:users',
        'person.*.first_name' => 'required_with:person.*.last_name',
    ]);

言語ファイルで配列ベースのフィールドバリデーションメッセージを指定するのも、同様に`*`文字を使えば簡単です。

    'custom' => [
        'person.*.email' => [
            'unique' => 'Each person must have a unique e-mail address',
        ]
    ],

<a name="custom-validation-rules"></a>
## カスタムバリデーションルール

<a name="using-rule-objects"></a>
### ルールオブジェクトの使用

Laravelは様々な便利なバリデーションルールを提供しています。しかし、独自のバリデーションも利用したいでしょう。カスタムバリデーションルールを登録する一つ目の方法は、ルールオブジェクトを使うやり方です。新しいルールオブジェクトを生成するには、`make:rule` Artisanコマンドを使用します。このコマンドを使用し、文字列が大文字であることをバリデートするルールを生成してみましょう。Laravelの新しいルールは、`app/Rules`ディレクトリに設置されます。

    php artisan make:rule Uppercase

ルールを生成したら、動作を定義する準備ができました。ルールオブジェクトは２つのメソッドを含みます。`passes`と`message`です。`passes`メソッドは属性の値と名前を受け取り、その属性値が有効であれば`true`、無効であれば`false`を返します。`message`メソッドは、バリデーション失敗時に使用する、バリデーションエラーメッセージを返します。

    <?php

    namespace App\Rules;

    use Illuminate\Contracts\Validation\Rule;

    class Uppercase implements Rule
    {
        /**
         * バリデーションの成功を判定
         *
         * @param  string  $attribute
         * @param  mixed  $value
         * @return bool
         */
        public function passes($attribute, $value)
        {
            return strtoupper($value) === $value;
        }

        /**
         * バリデーションエラーメッセージの取得
         *
         * @return string
         */
        public function message()
        {
            return 'The :attribute must be uppercase.';
        }
    }

もちろん、翻訳ファイルのエラーメッセージを返したい場合は、`message`メソッドから`trans`ヘルパを呼び出せます。

    /**
     * バリデーションエラーメッセージの取得
     *
     * @return string
     */
    public function message()
    {
        return trans('validation.uppercase');
    }

ルールが定義できたら、他のバリデーションルールと一緒に、ルールオブジェクトのインスタンスをバリデタへ渡し、指定します。

    use App\Rules\Uppercase;

    $request->validate([
        'name' => ['required', new Uppercase],
    ]);

<a name="using-extensions"></a>
### 拡張の使用

カスタムバリデーションルールを登録するもう一つの方法では、`Validator`[ファサード](/docs/{{version}}/facades)の`extend`メソッドを使用します。カスタムバリデーションルールを登録するために、 [サービスプロバイダ](/docs/{{version}}/providers)の中で、このメソッドを使ってみましょう。

    <?php

    namespace App\Providers;

    use Illuminate\Support\ServiceProvider;
    use Illuminate\Support\Facades\Validator;

    class AppServiceProvider extends ServiceProvider
    {
        /**
         * アプリケーションサービスの初期処理
         *
         * @return void
         */
        public function boot()
        {
            Validator::extend('foo', function ($attribute, $value, $parameters, $validator) {
                return $value == 'foo';
            });
        }

        /**
         * サービスプロバイダ登録
         *
         * @return void
         */
        public function register()
        {
            //
        }
    }

カスタムバリデーターのクロージャは４つの引数を取ります。`$attribute`はバリデーションをしているフィールド、`$value`はその値、`$parameters`はルールに渡された引数、最後は`Validator`インスタンスです。

クロージャの代わりに`extend`メソッドへクラスとメソッドを渡すこともできます。

    Validator::extend('foo', 'FooValidator@validate');

#### エラーメッセージの定義

カスタムルールに対するエラーメッセージを定義する必要もあります。インラインでカスタムエラーの配列を使うか、バリデーション言語ファイルにエントリーを追加するどちらかで行えます。このメッセージは属性とエラーメッセージを指定するだけの一次配列で、「カスタマイズ」した配列を入れてはいけません。

    "foo" => "Your input was invalid!",

    "accepted" => "The :attribute must be accepted.",

    // 残りのバリデーションエラーメッセージ…

カスタムバリデーションルールを作成する場合、エラーメッセージのカスタムプレースフォルダーも定義したいことがあります。前記の方法でカスタムバリデータを作成し、それから`Validator`ファサードの`replacer`メソッドを呼びだしてください。これは[サービスプロバイダ](/docs/{{version}}/providers)の`boot`メソッドの中で行います。

    /**
     * 全アプリケーションサービスの初期処理
     *
     * @return void
     */
    public function boot()
    {
        Validator::extend(...);

        Validator::replacer('foo', function ($message, $attribute, $rule, $parameters) {
            return str_replace(...);
        });
    }

#### 暗黙の拡張

バリデートする属性が存在していない場合か、[`required`](#rule-required)ルールで定義している「空」の場合、カスタム拡張したものも含め、通常のバリデーションルールは実行されません。たとえば[`unique`](#rule-unique)ルールは`null`値に対して実行されません。

    $rules = ['name' => 'unique'];

    $input = ['name' => null];

    Validator::make($input, $rules)->passes(); // true

属性が空であってもルールを実行するということは、その属性が必須であることを暗黙のうちに示しています。このような「暗黙の」拡張を作成するには、`Validator::extendImplicit()`メソッドを使います。

    Validator::extendImplicit('foo', function ($attribute, $value, $parameters, $validator) {
        return $value == 'foo';
    });

> {note} 「暗黙の」拡張は、単にその属性が必須であると**ほのめかしている**だけです。属性が存在しない場合や空のときに、実際にバリデーションを失敗と判断するかどうかは、みなさん次第です。
