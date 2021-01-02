# データベースと連携しよう

webアプリケーションを作りに当たり、情報を保存する仕組みは必須です。  
現代だと大半はリレーショナルデータベースを利用することになるでしょう。

step4の実装では、まだ動くアプリケーションとは言えません。  
それは情報を記録したり読み出したりする部分がないからです。


## Laravelにおけるデータベースとやり取りをする方法

LaravelではEloquentと呼ばれるO/Rマッパをつかってデータベースとやり取りするのが一般的です。
O/Rマッパとはオブジェクトリレーショナルマッピングと呼ばれるもので、データベースの構造からオブジェクトの構造に変化させたり、その逆を担うためのソリューションです。

## オブジェクトのモデルと、データモデルを見比べてみる

オブジェクトモデル
![](/images/image9.jpg)

データモデル
![](/images/image10.jpg)

この図を見ればわかると思いますが、大体一緒なんですよね。  
(もちろんデータモデルとオブジェクトモデルが乖離することも結構あります)

## オブジェクトのモデルと、データモデルをどう連携させるか？

これは結構難しい問題です。作るアプリ、データモデルとオブジェクトモデルの類似性などをみながらどう連携させていくのか決めます。
ただし連携方法はいくつかのパターンに別れます。

1. データモデルに寄せていいく
1. オブジェクトモデルの中にデータモデルを内包させる
1. データモデルからオブジェクトモデルに変換させる


### データモデルに寄せていいく
これはEloquent側に先程のガチャを実装していくという形になります。
データモデルとの類似性が高ければこの方法は有効になります。

### オブジェクトモデルの中にデータモデルを内包させる
オブジェクトモデル側のコンストラクタなどでデータモデルを内包出来るようにします。
オブジェクトモデル側で必要なデータをデータモデルから取り出して利用するような実装です。

### データモデルからオブジェクトモデルに変換させる
データ取得時にはデータモデルからオブジェクトモデルに変換したり、データ保存時にはオブジェクトモデルからデータモデルに変換する手法です。
変換処理部分でのバグが出やすいのと、実装コストが上がりやすい傾向にあります。


どの方法を採用してもオブジェクトモデルでただ実装するよりは辛さが出てきます。  
しかしこれはしかたないので、モデルや様々なソフトウェアを実装するための条件をみて判断していくしかありません。  
永続化(データの保存)は複雑なものなので仕方がありません。

今回は **「データモデルに寄せていいく」** アプローチで実装していきます。

## まずはデータベース周りの準備をする

まずはElouqentのモデルを作成します。

```shell
$ docker-compose exec php php artisan make:model Gacha -m 
$ docker-compose exec php php artisan make:model Item -m 
$ docker-compose exec php php artisan make:model Prize -m 
```

`-m` オプションをつける理由はマイグレーションファイルも同時に生成するために付けています。 

### マイグレーションとは？
データベースの構造を管理するためのツールです。
つまりテーブルの作成や変更、削除、などを管理する機構です。

## マイグレーションファイルに定義を記述する

`database/migrations` にマイグレーションファイルが生成されているかと思います。

- xxxxx_create_gachas_table.php
- xxxxx_create_items_table.php
- xxxxx_create_prizes_table.php

それぞれ編集していきましょう。


```php
<?php
// database/migrations/xxxxx_create_gachas_table.php

use Illuminate\Database\Migrations\Migration;
use Illuminate\Database\Schema\Blueprint;
use Illuminate\Support\Facades\Schema;

class CreateGachasTable extends Migration
{
    /**
     * Run the migrations.
     *
     * @return void
     */
    public function up()
    {
        Schema::create('gachas', function (Blueprint $table) {
            $table->id();
            $table->string('name')->comment('ガチャの名前');
            $table->timestamps();
        });
    }

    /**
     * Reverse the migrations.
     *
     * @return void
     */
    public function down()
    {
        Schema::dropIfExists('gachas');
    }
}
```

```php
<?php
// database/migrations/xxxxx_create_items_table.php

use Illuminate\Database\Migrations\Migration;
use Illuminate\Database\Schema\Blueprint;
use Illuminate\Support\Facades\Schema;

class CreateItemsTable extends Migration
{
    /**
     * Run the migrations.
     *
     * @return void
     */
    public function up()
    {
        Schema::create('items', function (Blueprint $table) {
            $table->id();
            $table->string('name');
            $table->timestamps();
        });
    }

    /**
     * Reverse the migrations.
     *
     * @return void
     */
    public function down()
    {
        Schema::dropIfExists('items');
    }
}
```

```php
<?php
// database/migrations/xxxxx_create_prizes_table.php

use Illuminate\Database\Migrations\Migration;
use Illuminate\Database\Schema\Blueprint;
use Illuminate\Support\Facades\Schema;

class CreatePrizesTable extends Migration
{
    /**
     * Run the migrations.
     *
     * @return void
     */
    public function up()
    {
        Schema::create('prizes', function (Blueprint $table) {
            $table->id();
            $table->unsignedBigInteger('gacha_id');
            $table->unsignedBigInteger('item_id');

            $table->foreign('gacha_id')->references('id')->on('gachas')->onDelete('cascade');
            $table->foreign('item_id')->references('id')->on('items')->onDelete('cascade');
            $table->smallInteger('probability');
            $table->timestamps();
        });
    }

    /**
     * Reverse the migrations.
     *
     * @return void
     */
    public function down()
    {
        Schema::dropIfExists('prizes');
    }
}

```

## マイグレーションを実行する

```shell
$ docker-compose exec php php artisan migrate
```

## データを用意する

Seederを使って初期値(データ)を用意します。

```php
<?php declare(strict_types=1);
//  database/seeders/DatbaseSeeder.php

namespace Database\Seeders;

use App\Models\Gacha;
use App\Models\Item;
use App\Models\Prize;
use Illuminate\Database\Seeder;

class DatabaseSeeder extends Seeder
{
    /**
     * Seed the application's database.
     */
    public function run(): void
    {
        $gachas = Gacha::factory(1)->create();
        $gacha = $gachas->get(0);
        Item::factory(10)->create()->each(function ($item) use ($gacha): void {
            Prize::factory()->for($item)->for($gacha)->create();
        });
    }
}
```

Seederを実行してテーブルにデータを反映します。

```shell
$ docker-compose exec php php artisan db:seed 
```

## Gacha::draw()メソッドの実装をする

まずはテストを用意しましょう。

```shell
$ docker-compose exec php php artisan make:test -u Models/GachaTest   
```

ひな形を生成したら`use PHPUnit\Framework\TestCase;` を削除し、代わりに`use Tests\TestCase;` を記述しましょう。

```php
<?php declare(strict_types=1);
// tests/Unit/Modles/GachaTest.php

namespace Tests\Unit\Models;

use App\Models\Gacha;
use App\Models\Item;
use Tests\TestCase;

class GachaTest extends TestCase
{
    /**
     * A basic unit test example.
     */
    public function testDraw(): void
    {
        $gacha = Gacha::first();
        $item = $gacha->draw();
        $this->assertInstanceOf(Item::class, $item);
    }
}
```

テストを用意したら次は実装します。

```php
<?php declare(strict_types=1);

namespace App\Models;

use Illuminate\Database\Eloquent\Factories\HasFactory;
use Illuminate\Database\Eloquent\Model;

class Gacha extends Model
{
    use HasFactory;

    public function draw(): Item
    {
        return new Item();
    }
}
```

これでテストが通ります。

```shell
$ docker-compose exec php php artisan test tests/Unit/Models/GachaTest.php
```

## 実際にロジックを実装する

step4で実装した `App\Gacha\Gacha` クラスの実装を移していきます。  
Eloquentにあわせて少し実装を変更していますが、ほぼ一緒になります。


```php
<?php declare(strict_types=1);

namespace App\Models;

use Illuminate\Database\Eloquent\Collection;
use Illuminate\Database\Eloquent\Factories\HasFactory;
use Illuminate\Database\Eloquent\Model;

/**
 * Class Gacha
 *
 * @property Collection $prizes
 */
class Gacha extends Model
{
    use HasFactory;

    /**
     * @return Item
     * @throws \Exception
     */
    public function draw(): Item
    {
        $totalProbability = $this->prizes->reduce(static function (int $ac, Prize $prize) {
            return $ac + $prize->probability;
        }, 0);

        $boundary = random_int(1, $totalProbability);
        $countPriority = 0;

        foreach ($this->prizes as $prize) {
            $countPriority += $prize->probability;

            if ($boundary <= $countPriority) {
                return $prize->item;
            }
        }

        throw new \RuntimeException('item not found.');
    }

    public function prizes()
    {
        return $this->hasMany(Prize::class);
    }
}
```

実装が完了したら、テストを回して正しく動作しているか確認しましょう。

```shell
$ docker-compose exec php php artisan test tests/Unit/Models/GachaTest.php
```

型情報しかテストをしてないのでテストを修正します。


```php
<?php declare(strict_types=1);

namespace Tests\Unit\Models;

use App\Models\Gacha;
use Tests\TestCase;

class GachaTest extends TestCase
{
    /**
     * A basic unit test example.
     */
    public function testDraw(): void
    {
        /** @var Gacha $gacha */
        $gacha = Gacha::first();
        $item = $gacha->draw();

        self::assertTrue($item->id > 0);
        self::assertTrue($item->id < 11);
    }
}
```

## addPrizeを実装する




これでEloquentのモデルの中にロジックを実装することができ、データベースとの連携ができました。



