---
sort: 3
---
# コントローラ、ビューの作成

## コントローラの作成

コマンドで作成した`CartController`を使って、コントローラを作成します。

```bash
php artisan make:controller CartController
```

まずは、カート情報を追加するためのフォームを表示する`create`メソッドを作成します。

**app/Http/Controllers/CartController.php**

```php
<?php
namespace App\Http\Controllers;

use Illuminate\Http\Request;
use App\Models\Cart; // 追加

class CartController extends Controller
{
    // --- 以下を追加 ---
    public function create()
    {
        return view('cart.create');
    }
    // --- ここまで ---
}
```

**【解説】**

`use App\Models\Cart`: `Cart`モデルを使用する宣言をします。

`public function create`: <br>
Laravelでは、コントローラに記述する`create` メソッドは、「新規登録画面を表示するためのメソッド」として一般的に使われます。

`return view('cart.create');`: <br>
`view`関数は、ビューを返す関数です。
ここでは、`cart.create`ビューファイルを表示するように設定しています。

## ビューの作成(カート追加画面)

次に、カートに商品を追加するためのフォームを作成します。

1. `resources/views`ディレクトリに`cart`ディレクトリを作成する
2. `cart`ディレクトリに`create.blade.php`を作成し、以下のように記述する

    ```php
    <!DOCTYPE html>
    <html lang="ja">
    <head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>サンプル</title>
    </head>
    <body>
        <h3>カートに追加</h3>
        <form action="{{ route('cart.store') }}" method="POST">
        @csrf
        番号:<input type="number" name="ident" min="1" max="15"><br>
        数量:<input type="number" name="quantity" min="1" max="10"><br><br>
        <input type="submit" value="カートに追加">
        </form>
    </body>
    </html>
    ```
