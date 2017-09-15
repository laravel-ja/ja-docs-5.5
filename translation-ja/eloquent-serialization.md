# Eloquent：シリアライズ

- [イントロダクション](#introduction)
- [モデルとコレクションのシリアライズ](#serializing-models-and-collections)
    - [配列へのシリアライズ](#serializing-to-arrays)
    - [JSONへのシリアライズ](#serializing-to-json)
- [JSONに含めない属性](#hiding-attributes-from-json)
- [JSONへ値を追加](#appending-values-to-json)
- [Date Serialization](#date-serialization)

<a name="introduction"></a>
## イントロダクション

JSONでAPIを作成する場合にはモデルとリレーションを配列やJSONに変換する必要が良く起きます。そのためEloquentはシリアライズ結果にどの属性を含むかをコントロールしながら、変換を行う便利なメソッドを含んでいます

<a name="serializing-models-and-collections"></a>
## モデルとコレクションのシリアライズ

<a name="serializing-to-arrays"></a>
### 配列へのシリアライズ

モデルとロード済みの[リレーション](/docs/{{version}}/eloquent-relationships)を配列に変換する場合、`toArray`メソッドを使います。このメソッドは再帰的に動作しますので、全属性と全リレーション（リレーションのリレーションも含む）は配列へ変換されます。

    $user = App\User::with('roles')->first();

    return $user->toArray();

モデルの[コレクション](/docs/{{version}}/eloquent-collections)を配列に変換することもできます。

    $users = App\User::all();

    return $users->toArray();

<a name="serializing-to-json"></a>
### JSONへのシリアライズ

モデルをJSONへ変換するには`toJson`メソッドを使います。`toArray`と同様に`toJson`メソッドは再帰的に動作し、全属性と全リレーションをJSONへ変換します。

    $user = App\User::find(1);

    return $user->toJson();

もしくはモデルやコレクションが文字列へキャストされる場合、自動的に`toJson`メソッドが呼び出されます。

    $user = App\User::find(1);

    return (string) $user;

文字列にキャストする場合、モデルやコレクションはJSONに変換されますので、アプリケーションのルートやコントローラーから直接Eloquentオブジェクトを返すことができます。

    Route::get('users', function () {
        return App\User::all();
    });

<a name="hiding-attributes-from-json"></a>
## JSONに含めない属性

モデルから変換する配列やJSONに、パスワードのような属性を含めたくない場合があります。それにはモデルの`$hidden`プロパティーに定義を追加してください。

    <?php

    namespace App;

    use Illuminate\Database\Eloquent\Model;

    class User extends Model
    {
        /**
         * 配列に含めない属性
         *
         * @var array
         */
        protected $hidden = ['password'];
    }

> {note} リレーションを含めない場合は、メソッド名を指定してください。

もしくはモデルを変換後の配列やJSONに含めるべき属性のホワイトリストを定義する、`visible`プロパティーを使用してください。モデルが配列やJSONへ変換される場合、その他の属性はすべて、変換結果に含まれません。

    <?php

    namespace App;

    use Illuminate\Database\Eloquent\Model;

    class User extends Model
    {
        /**
         * 配列中に含める属性
         *
         * @var array
         */
        protected $visible = ['first_name', 'last_name'];
    }

#### プロパティ配列出力管理の一時的変更

特定のモデルインスタンスにおいて、通常は配列に含めない属性を含めたい場合は、`makeVisible`メソッドを使います。このメソッドは、メソッドチェーンしやすいようにモデルインスタンスを返します。

    return $user->makeVisible('attribute')->toArray();

同様に、通常は含める属性を特定のインスタンスで隠したい場合は、`makeHidden`メソッドを使います。

    return $user->makeHidden('attribute')->toArray();

<a name="appending-values-to-json"></a>
## JSONへ値を追加

モデルを配列やJSONへキャストするとき、データベースに対応するカラムがない属性の配列を追加する必要がある場合も時々あります。これを行うには、最初にその値の[アクセサ](/docs/{{version}}/eloquent-mutators)を定義します。

    <?php

    namespace App;

    use Illuminate\Database\Eloquent\Model;

    class User extends Model
    {
        /**
         * ユーザーの管理者フラグを取得
         *
         * @return bool
         */
        public function getIsAdminAttribute()
        {
            return $this->attributes['admin'] == 'yes';
        }
    }

アクセサができたらモデルの`appends`プロパティーへ属性名を追加します。アクセサが「キャメルケース」で定義されていても、属性名は通常通り「スネークケース」でアクセスされることに注目してください。

    <?php

    namespace App;

    use Illuminate\Database\Eloquent\Model;

    class User extends Model
    {
        /**
         * モデルの配列形態に追加するアクセサ
         *
         * @var array
         */
        protected $appends = ['is_admin'];
    }

`appends`リストに属性を追加すれば、モデルの配列とJSON形式両方に含まれるようになります。`appends`配列の属性もモデルの`visible`と`hidden`の設定に従い動作します。

<a name="date-serialization"></a>
## Date Serialization

Laravel extends the [Carbon](https://github.com/briannesbitt/Carbon) date library in order to provide convenient customization of Carbon's JSON serialization format. To customize how all Carbon dates throughout your application are serialized, use the `Carbon::serializeUsing` method. The `serializeUsing` method accepts a Closure which returns a string representation of the date for JSON serialization:

    <?php

    namespace App\Providers;

    use Illuminate\Support\Carbon;
    use Illuminate\Support\ServiceProvider;

    class AppServiceProvider extends ServiceProvider
    {
        /**
         * Perform post-registration booting of services.
         *
         * @return void
         */
        public function boot()
        {
            Carbon::serializeUsing(function ($carbon) {
                return $carbon->format('U');
            });
        }

        /**
         * Register bindings in the container.
         *
         * @return void
         */
        public function register()
        {
            //
        }
    }
