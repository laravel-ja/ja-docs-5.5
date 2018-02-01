# Eloquent：ミューテタ

- [イントロダクション](#introduction)
- [アクセサとミューテタ](#accessors-and-mutators)
    - [アクセサの定義](#defining-an-accessor)
    - [ミューテタの定義](#defining-a-mutator)
- [日付ミューテタ](#date-mutators)
- [属性キャスト](#attribute-casting)
    - [配列とJSONのキャスト](#array-and-json-casting)

<a name="introduction"></a>
## イントロダクション

アクセサとミューテタはモデルの取得や値を設定するときに、Eloquent属性のフォーマットを可能にします。たとえば[Laravelの暗号化](/docs/{{version}}/encryption)を使いデータベース保存時に値を暗号化し、Eloquentモデルでアクセスする時には自動的にその属性を復元するように設定できます。

カスタムのアクセサやミューテタに加え、Eloquentは日付フールドを自動的に[Carbon](https://github.com/briannesbitt/Carbon)インスタンスにキャストしますし、[テキストフィールドをJSONにキャスト](#attribute-casting)することもできます。

<a name="accessors-and-mutators"></a>
## アクセサとミューテタ

<a name="defining-an-accessor"></a>
### アクセサの定義

アクセサを定義するには、アクセスしたいカラム名が「studlyケース（Upper Camel Case）」で`Foo`の場合、`getFooAttribute`メソッドをモデルに作成します。以下の例では、`first_name`属性のアクセサを定義しています。`first_name`属性の値にアクセスが起きると、Eloquentは自動的にこのアクセサを呼び出します。

    <?php

    namespace App;

    use Illuminate\Database\Eloquent\Model;

    class User extends Model
    {
        /**
         * ユーザーのファーストネームを取得
         *
         * @param  string  $value
         * @return string
         */
        public function getFirstNameAttribute($value)
        {
            return ucfirst($value);
        }
    }

ご覧の通り、アクセサにはそのカラムのオリジナルの値が渡されますので、それを加工し値を返します。アクセサの値にアクセスするには、モデルインスタンスの`first_name`属性へアクセスしてください。

    $user = App\User::find(1);

    $firstName = $user->first_name;

もちろん、既存の属性を元に算出した、新しい値をアクセサを使用し返すことも可能です。

    /**
     * ユーザーのフルネーム取得
     *
     * @return string
     */
    public function getFullNameAttribute()
    {
        return "{$this->first_name} {$this->last_name}";
    }

<a name="defining-a-mutator"></a>
### ミューテタの定義

ミューテタを定義するにはアクセスしたいカラム名が`Foo`の場合、モデルに「ローワーキャメルケース」で`setFooAttribute`メソッドを作成します。今回も`first_name`属性を取り上げ、ミューテタを定義しましょう。このミューテタはモデルの`first_name`属性へ値を設定する時に自動的に呼びだされます。

    <?php

    namespace App;

    use Illuminate\Database\Eloquent\Model;

    class User extends Model
    {
        /**
         * ユーザーのファーストネームを設定
         *
         * @param  string  $value
         * @return void
         */
        public function setFirstNameAttribute($value)
        {
            $this->attributes['first_name'] = strtolower($value);
        }
    }

ミューテタは属性に設定しようとしている値を受け取りますのでこれを加工し、Eloquentモデルの`$attributes`内部プロパティへ加工済みの値を設定します。では`Sally`を`first_name`属性へ設定してみましょう。

    $user = App\User::find(1);

    $user->first_name = 'Sally';

上記の場合、`setFirstNameAttribute`メソッドが呼び出され、`Sallay`の値が渡されます。このミューテタはそれから名前に`strtolower`を適用し、その値を`$attributes`内部配列へ設定します。

<a name="date-mutators"></a>
## 日付ミューテタ

デフォルトでEloquentは`created_at`と`updated_at`カラムを[Carbon](https://github.com/briannesbitt/Carbon)インスタンスへ変換します。CarbonはPHPネイティブの`DateTime`クラスを拡張しており、便利なメソッドを色々と提供しています。モデルの`$dates`プロパティをオーバーライドすることで、どのフィールドを自動的に変形するのか、逆にこのミューテタを適用しないのかをカスタマイズできます。

    <?php

    namespace App;

    use Illuminate\Database\Eloquent\Model;

    class User extends Model
    {
        /**
         * 日付を変形する属性
         *
         * @var array
         */
        protected $dates = [
            'created_at',
            'updated_at',
            'deleted_at'
        ];
    }

カラムが日付だと推定される場合、値はUnixタイムスタンプ、日付文字列(`Y-m-d`)、日付時間文字列、それにもちろん`DateTime`や`Carbon`インスタンスを値としてセットでき、日付は自動的に正しくデータベースへ保存されます。

    $user = App\User::find(1);

    $user->deleted_at = now();

    $user->save();

前記の通り`$dates`プロパティにリストした属性を取得する場合、自動的に[Carbon](https://github.com/briannesbitt/Carbon)インスタンスへキャストされますので、その属性でCarbonのメソッドがどれでも使用できます。

    $user = App\User::find(1);

    return $user->deleted_at->getTimestamp();

#### Dateフォーマット

デフォルトのタイムスタンプフォーマットは`'Y-m-d H:i:s'`です。タイムスタンプフォーマットをカスタマイズする必要があるなら、モデルの`$dateFormat`プロパティを設定してください。このプロパティは日付属性がデータベースにどのように保存されるかと同時に、モデルが配列やJSONにシリアライズされる時のフォーマットを決定します。

    <?php

    namespace App;

    use Illuminate\Database\Eloquent\Model;

    class Flight extends Model
    {
        /**
         * モデルの日付カラムの保存形式
         *
         * @var string
         */
        protected $dateFormat = 'U';
    }

<a name="attribute-casting"></a>
## 属性キャスト

モデルの`$casts`プロパティは属性を一般的なデータタイプへキャストする便利な手法を提供します。`$casts`プロパティは配列で、キーにはキャストする属性名を指定し、値にはそのカラムに対してキャストしたいタイプを指定します。サポートしているキャストタイプは`integer`、`real`、`float`、`double`、`string`、`boolean`、`object`、`array`、`collection`、`date`、`datetime`、`timestamp`です。

例としてデータベースには整数の`0`と`1`で保存されている`is_admin`属性を論理値にキャストしてみましょう。

    <?php

    namespace App;

    use Illuminate\Database\Eloquent\Model;

    class User extends Model
    {
        /**
         * ネイティブなタイプへキャストする属性
         *
         * @var array
         */
        protected $casts = [
            'is_admin' => 'boolean',
        ];
    }

これでデータベースには整数で保存されていても`is_admin`属性にアクセスすれば、いつでも論理値にキャストされます。

    $user = App\User::find(1);

    if ($user->is_admin) {
        //
    }

<a name="array-and-json-casting"></a>
### 配列とJSONのキャスト

`array`キャストタイプは、シリアライズされたJSON形式で保存されているカラムを取り扱うときに特に便利です。たとえば、データベースにシリアライズ済みのJSONを持つ`JSON`か`TEXT`フィールドがある場合、その属性に`array`キャストを追加すれば、Eloquentモデルにアクセスされた時点で自動的に非シリアライズ化され、PHPの配列へとキャストされます。

    <?php

    namespace App;

    use Illuminate\Database\Eloquent\Model;

    class User extends Model
    {
        /**
         * ネイティブなタイプへキャストする属性
         *
         * @var array
         */
        protected $casts = [
            'options' => 'array',
        ];
    }

キャストを定義後、`options`属性にアクセスすると自動的に非シリアライズされPHP配列になります。`options`属性へ値をセットすると配列は保存のために自動的にJSONへシリアライズされます。

    $user = App\User::find(1);

    $options = $user->options;

    $options['key'] = 'value';

    $user->options = $options;

    $user->save();
