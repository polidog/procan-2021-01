# ブラウザで動くようにしよう

テストを通じてガチャが動作することはわかりました。  
次はwebブラウザとして動く用に実装していきましょう。

## まずはコントローラを作成する

```shell
 $ docker-compose exec php php artisan make:controller GachaController    
```

生成された `GachaController` に手を入れます。

```php
<?php declare(strict_types=1);

namespace App\Http\Controllers;

use App\Models\Gacha;
use Symfony\Component\HttpKernel\Exception\NotFoundHttpException;

class GachaController extends Controller
{
    public function index()
    {
        $gacha = Gacha::first();
        return view('gacha/index', ['gacha' => $gacha]);
    }

    public function exec(int $id)
    {
        $gacha = Gacha::find($id);
        if (!$gacha instanceof Gacha) {
            throw new NotFoundHttpException('gacha not found.');
        }
        $item = $gacha->draw();
        return view('gacha/exec', ['item' => $item]);
    }
}

```

## viewを作成する

次にそれぞれのコントローラのメソッドに対応するviewを作成します。

まずは `resources/views/gacha/index.blade.php` から作ります。
```blade
@extends('layouts.app')

@section('content')
    <div class="container">
        <div class="row justify-content-center">
            <div class="col-md-8">
                <div class="card">
                    <div class="card-header">{{ $gacha->name }}</div>

                    <div class="card-body">
                        <div style="width: 100%">
                            <img src="https://1.bp.blogspot.com/-sZbaFXJ4y0A/UnyGKAJjwbI/AAAAAAAAacE/RYDWRq73Hsc/s800/gachagacha.png" style="width: 100%; text-align: center"/>
                        </div>
                        <a href="{{ url('/gacha', $gacha->id) }}" class="btn btn-primary btn-block">ガチャを引く</a>
                    </div>
                </div>
            </div>
        </div>
    </div>
@endsection
```

次に `resources/views/gacha/exec.blade.php` から作ります。

```blade
@extends('layouts.app')

@section('content')
    <div class="container">
        <div class="row justify-content-center">
            <div class="col-md-8">
                <div class="card">
                    <div class="card-header">ガチャ結果</div>

                    <div class="card-body">
                        「{{ $item->name }}」を取得しました!!!!
                        <a href="{{ url('/gacha') }}" class="btn btn-primary btn-block">戻る</a>
                    </div>
                </div>
            </div>
        </div>
    </div>
@endsection
```

## ルーティングを設定する

```php
<?php declare(strict_types=1);

use Illuminate\Support\Facades\Route;

/*
|--------------------------------------------------------------------------
| Web Routes
|--------------------------------------------------------------------------
|
| Here is where you can register web routes for your application. These
| routes are loaded by the RouteServiceProvider within a group which
| contains the "web" middleware group. Now create something great!
|
*/

Route::middleware('auth')->group(function (): void {
    Route::get('/gacha', [\App\Http\Controllers\GachaController::class, 'index']);
    Route::get('/gacha/{id}', [\App\Http\Controllers\GachaController::class, 'exec']);
});

Auth::routes();

Route::get('/home', [App\Http\Controllers\HomeController::class, 'index'])->name('home');
```

後はブラウザで [http://localhost/gacha](http://localhost/gacha) にアクセスすればガチャがひけるようになります。