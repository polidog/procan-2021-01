# アイテムボックスを実装しよう

まずはアイテムボックスの仕様をもう一度みてみましょう。

{% include "../spec/box_spec.md" %}

ガチャとの関係性を図にして整理してみるとこのような形になります。

![](/images/image13.jpg)

** 10個までアイテムを入れることができる ** という仕様なので、ガチャを引いた後アイテムボックスにアイテムが入るか調べる必要があります。  
あとは当然ですが、ガチャを引いたらアイテムボックスに入れる必要があります。


## アイテムボックスを中心としたモデルを考えてみよう

先程の図は、ガチャを中心としたモデル図になっています。  
今度はアイテムボックスを中心にした図をみてみましょう。

まずは仕様から言葉を分解してみます。

- アイテムボックス
- 10個までアイテムを入れることができる
- アイテム一覧を取得できる
- 破棄することができる

![](/images/image14.jpg)

図で表すと、上記になります。

「ユーザー」という言葉は記述がありませんが、暗黙的に入っています。
これは、アイテムボックスというのは自然に**「誰かが保持している」**ということを想像できるかという理由でユーザーという概念も図にしています。  
ソフトウェア開発の現場では、こういった「暗黙的な概念」が必ず潜んでいます。仕様には書ききれない部分を発見するための努力はどの現場でも必要になります。

## メソッドになりそうな候補を探す
動詞的な要素は大体においてメソッドになります。
今回の場合だと以下のようなものが候補になります。

- アイテムを入れることができる -> add
- アイテム一覧を取得できる -> getItems
- 破棄できる -> remove

これらはメソッドの候補になります。
また、ガチャからみた図(このページ最初の図)には「ガチャからアイテムボックスに問い合わせる」記述があります。
つまりアイテムボックスがいっぱいかどうかを判定するメソッドも必要になります。
なので最終的には以下のようになります。

- アイテムを入れることができる -> add
- アイテム一覧を取得できる -> getItems
- アイテムを破棄できる -> remove
- アイテムボックスがいっぱいか確認する -> isFull

## まずはEloquenに依存せずに書いてみる

Step4で `app/Gacha` の中に実装していましたが、まずはそこでアイテムリストを実装してみましょう。

### テストから記述します

今回はガチャを引く際にアイテムボックスを渡すようにします。
isFullで事前にアイテムボックスがいっぱいなのかを確認し、addメソッドでアイテムボックスに入れます。


```php
<?php declare(strict_types=1);

namespace Tests\Unit\Gacha;

use App\Gacha\Gacha;
use App\Gacha\Item;
use App\Gacha\ItemBox;
use App\Gacha\Prize;
use PHPUnit\Framework\TestCase;
use Prophecy\PhpUnit\ProphecyTrait;

class GachaTest extends TestCase
{
    use ProphecyTrait;

    public function testDraw(): void
    {
        $gacha = new Gacha();
        $item = new Item();

        $itemBox = $this->prophesize(ItemBox::class);
        $itemBox->isFull()->willReturn(false);

        $prize = $this->prophesize(Prize::class);
        $prize->getProbability()->willReturn(1);
        $prize->getItem()->willReturn($item);

        $gacha->addPrize($prize->reveal());
        $item = $gacha->draw($itemBox->reveal());

        $prize->getProbability()->shouldHaveBeenCalledTimes(2);
        $prize->getItem()->shouldHaveBeenCalledTimes(1);

        $itemBox->isFull()->shouldHaveBeenCalledTimes(1);
        $itemBox->add($item)->shouldHaveBeenCalledTimes(1);
    }

    public function testHasPrizes(): void
    {
        $gacha = new Gacha();

        foreach (range(1, 10) as $_) {
            $gacha->addPrize(new Prize(new Item(), 10));
        }

        self::assertTrue($gacha->hasPrizes());
    }
}
```

実装が終わったら、テストが失敗することを確認しましょう。

### 次にアイテムボックスの実装をしましょう。



```php
<?php declare(strict_types=1);
// app/Gacha/ItemBox.php

namespace App\Gacha;

class ItemBox
{
    private const MAX_ITEMS = 10;

    /**
     * @var Item[]
     */
    private $items = [];

    public function add(Item $item): void
    {
        $this->items[] = $item;
    }

    public function getItems(): array
    {
        return $this->items;
    }

    public function remove(int $index): void
    {
        unset($this->items[$index]);
    }

    public function isFull(): bool
    {
        return count($this->items) === self::MAX_ITEMS;
    }
}
```

```php
<?php declare(strict_types=1);
// app/Gacha/Gacha.php

namespace App\Gacha;

class Gacha
{
    private const MAX_PRIZE = 10;

    /**
     * @var Prize[]
     */
    private $prizes = [];

    /**
     * @return Item
     * @throws \Exception
     */
    public function draw(ItemBox $itemBox): Item
    {
        if ($itemBox->isFull()) {
            throw new \Exception('アイテムボックスがいっぱいです');
        }

        $totalProbability = array_reduce($this->prizes, static function (int $ac, Prize $prize) {
            return $ac + $prize->getProbability();
        }, 0);
        $boundary = random_int(1, $totalProbability);
        $countPriority = 0;

        foreach ($this->prizes as $prize) {
            $countPriority += $prize->getProbability();

            if ($boundary <= $countPriority) {
                $item = $prize->getItem();
                $itemBox->add($item);
                return $item;
            }
        }

        throw new \RuntimeException('item not found.');
    }

    public function addPrize(Prize $prize): void
    {
        if ($this->hasPrizes()) {
            throw new \Exception('景品の上限を超えています');
        }
        $this->prizes[] = $prize;
    }

    public function hasPrizes(): bool
    {
        return count($this->prizes) === self::MAX_PRIZE;
    }
}
```

## Eloquentを使って実装してみる

次にEloquentを使って実際に実装していきます。

### まずはデータ構造を考える

オブジェクトのモデルとは少々異なります。図にすると以下のようになります。

![](/images/image15.jpg)

テーブル定義は以下のようになります。

#### item_boxesテーブル

|  カラム      |　型           | 説明        |
| ----------- | ------------ | ---------- |
| id          | bigint          | 一意なID    |
| user_id    | bigint          | 所有しているユーザーID    |
| item_id    | bigint          | アイテムID    |

オブジェクトは階層的な性質が強いのに対して、RDBは表としての性質が強いのでどうしてもこのようなインピーダンスミスマッチが起きてしまいます。

`app/Gacha/ItemBox` と表現していたものが単純にEloquentのモデルとしては表現するのが難しくなります。  
便利な反面こういったデータ構造の扱い辛さEloquentの欠点と言えます。

こういう問題をどう扱うかは非常に難しいです。  
特に規模が大きいソフトウェア開発の場合は更に大変です。  
LaravelのようにEloquentのモデルを中心なフレームワークの場合、こういった問題の対処が難しいのです。
ある程度の規模からは、違うフレームワークを選択するほうが良いでしょう。

今回は ** EloquentのModelで表現できないオブジェクトは一旦 `app/Service` ディレクトリに実装を置く ** というルールを設定して実装します。



## まずはモデルクラスとマイグレーションを用意する

artisanコマンドを使って生成しましょう。

```shell
$ docker-compose exec php php artisan make:model ItemBox -m
```


そしてマイグレーションファイルを修正しましょう。

```php
<?php declare(strict_types=1);
// database/migrations/xxxxx_create_item_boxes_table.php

use Illuminate\Database\Migrations\Migration;
use Illuminate\Database\Schema\Blueprint;
use Illuminate\Support\Facades\Schema;

class CreateItemBoxesTable extends Migration
{
    /**
     * Run the migrations.
     */
    public function up(): void
    {
        Schema::create('item_boxes', function (Blueprint $table): void {
            $table->id();
            $table->unsignedBigInteger('user_id')->unique();
            $table->unsignedBigInteger('item_id');
            $table->foreign('user_id')->references('id')->on('users')->onDelete('cascade');
            $table->foreign('item_id')->references('id')->on('items')->onDelete('cascade');
            $table->smallInteger('max_items');
            $table->timestamps();
        });
    }

    /**
     * Reverse the migrations.
     */
    public function down(): void
    {
        Schema::dropIfExists('item_boxes');
    }
}
```

そしてマイグレーションを実行します。

```shell
$ docker-compose exec php php artisan migrate
```

## ItemBoxモデルを修正します。

`app/Models/ItemBox` が自動生成されてるので、修正していきます。
リレーションの設定を追記します。


```php
<?php declare(strict_types=1);

namespace App\Models;

use Illuminate\Database\Eloquent\Factories\HasFactory;
use Illuminate\Database\Eloquent\Model;

class ItemBox extends Model
{
    use HasFactory

    public function user()
    {
        return $this->belongsTo(User::class, 'user_id');
    }

    public function item()
    {
        return $this->belongsTo(Item::class, 'item_id');
    }
}

```


## テストの修正を行います。

実装する前に`GachaTest::testDraw()`テストを修正しましょう。

```php
<?php declare(strict_types=1);
// tests/Unit/Modles/GachaTest.php

namespace Tests\Unit\Models;

use App\Gacha\ItemBoxService;
use App\Models\Gacha;
use App\Models\Item;
use App\Models\Prize;
use Tests\TestCase;

class GachaTest extends TestCase
{
    /**
     * A basic unit test example.
     */
    public function testDraw(): void
    {
        $itemBox = $this->prophesize(ItemBoxService::class);
        $itemBox->isFull()->willReturn(false);


        /** @var Gacha $gacha */
        $gacha = Gacha::first();
        $item = $gacha->draw($itemBox->reveal());

        self::assertTrue($item->id > 0);
        self::assertTrue($item->id < 11);

        $itemBox->isFull()->shouldHaveBeenCalledTimes(1);
        $itemBox->add($item)->shouldHaveBeenCalledTimes(1);
    }

    public function testAddPrize(): void
    {
        $gacha = new Gacha();
        $gacha->name = 'test';
        $gacha->save();

        $item = Item::first();

        $prize = new Prize();
        $prize->probability = 1;
        $prize->item()->associate($item);

        $gacha->addPrize($prize);
        self::assertSame(1, $gacha->prizes()->count('id'));
    }
}
```

テストを実行して、テストが失敗することを確認します。

```shell
$ docker-compose exec php php artisan test tests/Unit/Models/GachaTest.php
```

## 次にItemBoxServiceを実装します。

```php
<?php declare(strict_types=1);

namespace App\Service;

use App\Models\Item;
use App\Models\ItemBox;
use App\Models\User;
use Illuminate\Database\Eloquent\Collection;

class ItemBoxService
{
    private const MAX_ITEMS = 10;

    /**
     * @var User
     */
    private $user;

    /**
     * ItemBoxRepository constructor.
     * @param User $user
     */
    public function __construct(User $user)
    {
        $this->user = $user;
    }

    public function add(Item $item): void
    {
        $itemBox = new ItemBox();
        $itemBox->user()->associate($this->user);
        $itemBox->item()->associate($item);
        $itemBox->save();
    }

    /**
     * @return Collection
     */
    public function getItems(): Collection
    {
        return ItemBox::where('user_id', $this->user->id)->get();
    }

    /**
     * @param int $itemBoxId
     * @throws \Exception
     */
    public function remove(int $itemBoxId): void
    {
        $itemBox = ItemBox::find($itemBoxId);

        if (!$itemBox instanceof ItemBox) {
            throw new \Exception('アイテムが取得できません');
        }
        $itemBox->delete();
    }

    public function isFull(): bool
    {
        return ItemBox::where('user_id', $this->user->id)->count() >= self::MAX_ITEMS;
    }
}
```

もう一度テストを実行すると、成功します。

```shell
$ docker-compose exec php php artisan test tests/Unit/Models/GachaTest.php
```

## ItemBoxServiceのテストを書こう

DBのテストなのでちょっとめんどくさいんですが、こんな感じに実装しましょう。

```php
<?php declare(strict_types=1);
// tests/Unit/Service/ItemBoxServiceTest.php

namespace Tests\Unit\Service;

use App\Models\Item;
use App\Models\ItemBox;
use App\Models\User;
use App\Service\ItemBoxService;
use Illuminate\Foundation\Testing\RefreshDatabase;
use Tests\TestCase;

class ItemBoxServiceTest extends TestCase
{
    use RefreshDatabase;

    /**
     * A basic unit test example.
     */
    public function testAdd(): void
    {
        $this->seed();
        $user = User::first();
        $this->saveItemBox($user);

        self::assertTrue(ItemBox::where('user_id', $user->id)->count() > 0);
    }

    public function testRemove(): void
    {
        $this->seed();

        $user = User::first();
        $this->saveItemBox($user);

        $boxCount = ItemBox::where('user_id', $user->id)->count();
        $itemBox = ItemBox::where('user_id', $user->id)->first();

        $service = new ItemBoxService($user);
        $service->remove($itemBox->id);
        self::assertSame(ItemBox::where('user_id', $user->id)->count(), $boxCount - 1);
    }

    public function testIsFull(): void
    {
        $this->seed();

        $user = User::first();
        $service = new ItemBoxService($user);

        foreach (Item::get() as $item) {
            $service->add($item);
        }

        self::assertTrue($service->isFull());
    }

    private function saveItemBox(User $user): void
    {
        $item = Item::first();
        $service = new ItemBoxService($user);
        $service->add($item);
    }
}
```

## ブラウザでの動作も変更しよう

最後にこのままではブラウザで動かないので動くようにしましょう。
まずはコントローラの修正からします。

`createItemBox` のメソッドを用意して各アクションから呼び出します。

```php
<?php declare(strict_types=1);

namespace App\Http\Controllers;

use App\Models\Gacha;
use App\Models\User;
use App\Service\ItemBoxService;
use Illuminate\Support\Facades\Auth;
use Symfony\Component\HttpKernel\Exception\NotFoundHttpException;

class GachaController extends Controller
{
    public function index()
    {
        $itemBoxService = $this->createItemBox();
        $gacha = Gacha::first();
        return view('gacha/index', [
            'gacha' => $gacha,
            'isFull' => $itemBoxService->isFull(),
        ]);
    }

    public function exec(int $id)
    {
        $gacha = Gacha::find($id);

        if (!$gacha instanceof Gacha) {
            throw new NotFoundHttpException('gacha not found.');
        }
        $itemBoxService = $this->createItemBox();
        $item = $gacha->draw($itemBoxService);
        return view('gacha/exec', ['item' => $item]);
    }

    private function createItemBox(): ItemBoxService
    {
        $user = Auth::user();

        if (!$user instanceof User) {
            throw new \Exception('user not found');
        }
        return new ItemBoxService($user);
    }
}
```

次にテンプレートを修正します。

`resources/views/gacha/index.blade.php`

```
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
                        @if ($isFull)
                            <div><a class="btn btn-primary btn-block disabled">ガチャを引く</a></div>
                            <div>
                                <a href="#">アイテムボックスへ</a>
                            </div>
                        @else
                            <a href="{{ url('/gacha', $gacha->id) }}" class="btn btn-primary btn-block">ガチャを引く</a>
                        @endif
                    </div>
                </div>
            </div>
        </div>
    </div>
@endsection
```

### ItemBoxのページも実装しよう

まずは`make`コマンドでコントローラを生成します。

```shell
$ docker-compose exec php php artisan make:Controller ItemBoxController
```

生成されたコードを修正してきます。

```php
<?php declare(strict_types=1);
// app/Http/Controllers/ItemBoxController.php

namespace App\Http\Controllers;

use App\Models\User;
use App\Service\ItemBoxService;
use Illuminate\Support\Facades\Auth;

class ItemBoxController extends Controller
{
    /**
     * @return \Illuminate\Contracts\View\View
     * @throws \Exception
     */
    public function index()
    {
        $itemBoxService = $this->createItemBox();

        return view('itemBox.index', [
            'items' => $itemBoxService->getItems(),
        ]);
    }

    /**
     * @param int $id
     * @return \Illuminate\Http\RedirectResponse
     * @throws \Exception
     */
    public function remove(int $id): \Illuminate\Http\RedirectResponse
    {
        $itemBoxService = $this->createItemBox();
        $itemBoxService->remove($id);
        return redirect()->route('itemBox');
    }

    private function createItemBox(): ItemBoxService
    {
        $user = Auth::user();

        if (!$user instanceof User) {
            throw new \Exception('user not found');
        }
        return new ItemBoxService($user);
    }
}
```

次にview用のファイルを用意します。
`reousrces/views/itemBox/index.blade.php`

```php
@extends('layouts.app')

@section('content')
    <div class="container">
        <div class="row justify-content-center">
            <div class="col-md-8">
                <div class="card">
                    <div class="card-header">アイテムボックス</div>
                    <div class="card-body">
                        <div class="row">
                            @foreach ($items as $item)
                                <div class="col-sm-6" style="margin-top: 14px;">
                                    <div class="card">
                                        <div class="card-body">
                                            <h5 class="card-title">{{ $item->item->name }}</h5>
                                            <a href="{{ route('itemBoxRemove', $item->id) }}" class="btn btn-danger">削除する</a>
                                        </div>
                                    </div>
                                </div>
                            @endforeach
                        </div>
                    </div>
                    <div class="card-body">
                        <a href="{{ url('/gacha') }}" class="btn btn-primary btn-block">ガチャへ</a>
                    </div>
                </div>
            </div>
        </div>
    </div>
@endsection
```

ついでにgachaのviewファイルにもアイテムボックスへのリンクを貼ります。

`resources/views/gacha/index.blade.php`

```php
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
                        @if ($isFull)
                            <div><a class="btn btn-primary btn-block disabled">ガチャを引く</a></div>
                        @else
                            <div><a href="{{ url('/gacha', $gacha->id) }}" class="btn btn-primary btn-block">ガチャを引く</a></div>
                        @endif
                        <div style="margin-top: 15px;"><a href="{{ url('/itemBox') }}" class="btn btn-primary btn-block">アイテムボックスへ</a></div>
                    </div>
                </div>
            </div>
        </div>
    </div>
@endsection
```

`resources/views/gacha/exec.blade.php`

```php
@extends('layouts.app')

@section('content')
    <div class="container">
        <div class="row justify-content-center">
            <div class="col-md-8">
                <div class="card">
                    <div class="card-header">ガチャ結果</div>

                    <div class="card-body">
                        <div style="text-align: center">
                            <img src="https://4.bp.blogspot.com/-F5qE4XwBojQ/V5NDtjF5RzI/AAAAAAAA8d0/2zxTHdEKxlQufC6UkcDc_-cdi7DUfBdwgCLcB/s250/capsule_close1_red.png" />
                        </div>
                        <div style="text-align: center">
                            「{{ $item->name }}」を取得しました!!!!
                        </div>
                        <a href="{{ url('/gacha') }}" class="btn btn-primary btn-block">戻る</a>
                        <a href="{{ url('/itemBox') }}" class="btn btn-primary btn-block">アイテムボックスへ</a>
                    </div>
                </div>
            </div>
        </div>
    </div>
@endsection
```

そしてルーティングを設定します。


```php
<?php declare(strict_types=1);
// routes/web.php

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
    Route::get('/itemBox', [\App\Http\Controllers\ItemBoxController::class, 'index'])->name('itemBox');
    Route::get('/itemBox/{id}/remove', [\App\Http\Controllers\ItemBoxController::class, 'remove'])->name('itemBoxRemove');
});

Auth::routes();

Route::get('/home', [App\Http\Controllers\HomeController::class, 'index'])->name('home');
```

これで[http://localhost/itemBox](http://localhost/itemBox)にアクセスすればアイテムボックスが表示されます。

![](/images/image16.png)
![](/images/image17.png)