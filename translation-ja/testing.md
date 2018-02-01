# テスト: テストの準備

- [イントロダクション](#introduction)
- [環境](#environment)
- [テストの生成と実行](#creating-and-running-tests)

<a name="introduction"></a>
## イントロダクション

Laravelはユニットテストも考慮して構築されています。実際、PHPUnitをサポートしており、最初から含まれています。アプリケーションのために`phpunit.xml`ファイルも最初から準備されています。さらにフレームワークはアプリケーションを記述的にテストするために便利なヘルパメソッドも持っています。

デフォルトでアプリケーションの`tests`ディレクトリには、２つのディレクトリが存在しています。`Feature` と`Unit`です。ユニットテストは極小さい、コードの独立した一部をテストします。実際、殆どのユニット(Unit)テストは一つのメソッドに焦点をあてます。機能(Feature)テストは、多くのオブジェクトがそれぞれどのように関しているかとか、JSONエンドポイントへ完全なHTTPリクエストを送ることさえ含む、コードの幅広い範囲をテストします。

`Feature`と`Unit`、両テストディレクトリには、`ExampleTest.php`が用意されています。真新しいLaravelアプリケーションをインストールしたらテストを実行するため、コマンドラインから`phpunit`を実行してください。

<a name="environment"></a>
## 環境

`phpunit.xml`ファイル中で環境変数が設定されているため、`phpunit`を実行するとLaravelは自動的に設定環境を`testing`にセットします。Laravelはまた、セッションとキャッシュの設定を`array`ドライバーに設定し、テスト中のセッションやキャッシュデータが残らないようにします。

必要であれば他のテスト設定環境を自由に作成することもできます。`testing`動作環境変数は`phpunit.xml`の中で設定されています。テスト実行前には、`config:clear` Artisanコマンドを実行し、設定キャッシュをクリアするのを忘れないでください。

<a name="creating-and-running-tests"></a>
## テストの生成と実行

新しいテストケースを作成するには、`make:test` Artisanコマンドを使います。

    // Featureディレクトリにテストを生成する
    php artisan make:test UserTest

    // Unitディレクトリにテストを生成する
    php artisan make:test UserTest --unit

テストを生成したら、PHPUnitを使用するときと同じように、テストメソッドを定義してください。テストを実行するには、ターミナルで`phpunit`コマンドを実行します。

    <?php

    namespace Tests\Unit;

    use Tests\TestCase;
    use Illuminate\Foundation\Testing\RefreshDatabase;

    class ExampleTest extends TestCase
    {
        /**
         * 基本的なテスト例
         *
         * @return void
         */
        public function testBasicTest()
        {
            $this->assertTrue(true);
        }
    }

> {note} テストクラスに独自の`setUp`メソッドを定義する場合は、`parent::setUp()`を確実に呼び出してください。
