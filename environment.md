---
sort: 2
---
# データベース環境構築

今回は、`items`テーブルに加え、カート内の商品を管理するための`cart`テーブルを作成します。
なお、`.env`ファイルは既に編集済みなので、再度編集する必要はありません。

## マイグレーション

今回は、`cart`テーブルを作成するためのマイグレーションファイルを追加し、コマンドを実行してテーブルを作成します。
なお、`cart`テーブルの構造は前期同様以下の通りです。

| カラム名 | データ型 | 制約 | 備考 |
| - | - | - | - |
|ident|int型|主キー|商品番号|
|quantity|int型||注文数|

1. VSCode上で、`Ctrl+Shift+P`(Macの場合は`Cmd+Shift+P`)を押し、コンテナを起動する(既に起動しているなら不要)
2. VSCode上で、`Ctrl+J`(Macの場合は`Cmd+J`)を押し、ターミナルを表示する
3. 以下のコマンドを実行して、`cart`テーブル用のマイグレーションファイルを作成する

    ```bash
    php artisan make:migration create_cart_table
    ```

4. `database/migrations/20xx_xx_xx_xxxxxx_create_cart_table.php` が作成されていることを確認する
5. `up`メソッドを以下のように修正する

    ```php
    public function up(): void
    {
        Schema::create('cart', function (Blueprint $table) {
            // デフォルトの記述はコメントアウト
            // $table->id();
            // $table->timestamps();

            // --- 以下を追加 ---
            $table->integer('ident')->primary();
            $table->integer('quantity');
            // 外部キー制約を追加
            $table->foreign('ident')->references('ident')->on('items')->onDelete('cascade');
            // --- ここまで ---
        });
    }
    ```

    **【解説】**

    `$table->foreign('ident')->references('ident')->on('items')->onDelete('cascade');`: <br>
    上記はメソッドチェーンと呼ばれる記述方法です。
    メソッドチェーンは、メソッドを連続して呼び出す記述方法で、コードを簡潔に書くことができます。

    `foreign`メソッドは、外部キー制約を追加するメソッドです。
    ここでは、`cart`テーブルの`ident`カラムに外部キー制約を追加しています。
    `references`メソッドで、外部キー制約の参照先を指定しています。
    ここでは、`items`テーブルの`ident`カラムを参照しています。
    `onDelete('cascade')`は、参照先のレコードが削除された際に、`cart`テーブルのレコードも削除されるように設定しています。

    **【補足(外部キー制約について)】**
    そもそも外部キー制約とはなんでしょうか？
    外部キー制約とは、テーブル間の関連性を強制する制約のことです。
    例えば、`cart`テーブルの`ident`カラムに外部キー制約を設定することで、`cart`テーブルの`ident`カラムには、`items`テーブルの`ident`カラムに存在する値のみが入るように制約を設けることができます。

6. 以下のコマンドを実行して、マイグレーションを実行する

    ```bash
    php artisan migrate
    ```

これで、`cart`テーブルが作成されました。

## モデルの作成

以下のコマンドで作成した`Cart`モデルを使って、`cart`テーブルとのやり取りを行います。

```bash
php artisan make:model Cart
```

**app/Models/Cart.php**

```php
<?php

namespace App\Models;

use Illuminate\Database\Eloquent\Factories\HasFactory;
use Illuminate\Database\Eloquent\Model;

class Cart extends Model
{
    use HasFactory;
    // --- 以下を追加 ---
    protected $table = 'cart';
    protected $primaryKey = 'ident';
    protected $fillable = ['ident','quantity'];
    public $timestamps = false;

    public function item()
    {
        return $this->belongsTo(Item::class, 'ident', 'ident');
    }
    // --- ここまで ---
}
```

**【解説】**

`protected $table = 'cart';`: <br>
`protected $table`プロパティは、モデルが対応するテーブル名を指定するプロパティです。

Laravelでは、基本的には、**モデル名は単数形**、そのモデルに対応する**テーブル名は複数形**でなければ追加の設定を記述しなければエラーとなります。
以前作成した、`Item`モデルに対応するテーブルが`items`テーブルで、上記ルールに則っていたため、エラーなくデータベース操作が可能でした。

しかし今回の場合は、`Cart`モデルに対応するテーブルは`cart`テーブルであり、テーブル名が**単数系**です。
この場合、明示的に`$table`プロパティに`cart`を指定しています。
これにより、今までどおりコントローラで`Cart`モデルを使ってデータベースとのやり取りを行うことができます。

`protected $primaryKey = 'ident';`: <br>
`protected $primaryKey`プロパティは、モデルの主キーを指定するプロパティです。
ここでは、`cart`テーブルの主キーが`ident`カラムであるため、`$primaryKey`プロパティに`ident`を指定しています。

`protected $fillable = ['ident','quantity'];`: <br>
`protected $fillable`プロパティは、モデルのプロパティに値を代入する際に、代入可能なカラムを指定するプロパティです。
ここでは、`cart`テーブルの`ident`カラムと`quantity`カラムに値を代入することを許可しています。

`public $timestamps = false;`: <br>
`public $timestamps`プロパティは、モデルの作成日時と更新日時を自動で更新するかどうかを指定するプロパティです。
ここでは、`cart`テーブルには作成日時と更新日時を持たせないため、`false`を指定しています。

`public function item()`: <br>
`item`メソッドは、`Cart`モデルと`Item`モデルのリレーションを設定するメソッドです。
※これを設定する理由は、前記のように`cart`テーブルに`items`テーブルの商品情報を結合するためです。
今回は`cart`テーブルへのCreateのみですので、`item`メソッドが無くても動きますが、**課題にて記述しますので**覚えておきましょう。

`return $this->belongsTo(Item::class, 'ident', 'ident');`: <br>
`belongsTo`メソッドは、リレーション先のモデルを取得するメソッドです。
第1引数には、リレーション先のモデルを指定します。
第2引数には、リレーション先のモデルの外部キー(`items`テーブルの`ident`カラム)を指定します。
第3引数には、リレーション先のモデルの主キー(`cart`テーブルの`ident`カラム)を指定します。

**【補足(リレーションについて)】**

`Cart`モデルに`item`メソッドを追加する理由は、`Cart`モデルと`Item`モデルのリレーションを設定するためです。
リレーションとは、データベースのテーブル間における関連性をモデル間にも反映させるための機能です。
リレーションを設定することで、モデル間のデータ取得が容易になり、コードの記述量が減ります。

では、今回の場合は`Cart`モデルと`Item`モデルにどのような関連性があるのでしょうか。
それには、`cart`テーブルと`items`テーブルの`ident(商品番号)`カラムが関連しているということが挙げられます。
ここでいう関連しているとは、`cart`テーブルの`ident`カラムの値が`items`テーブルの`ident`カラムの値と一致するということです。

また、1つのカートに対して複数の商品が存在します。
このことをデータベースの用語で言うと、`cart`テーブルと`items`テーブルは**1対多の関係にある**と言えます。
これらを踏まえて、`Cart`モデルに`item`メソッドを追加します。