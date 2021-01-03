# ガチャを実装しよう

まずは今回の仕様を再確認しましょう。

{% include "../spec/spec.md" %}

このうちstep4では以下の部分を実装します。

{% include "../spec/base_spec.md" %}

## まずはモデルをコードに落とし込もう

![](/images/image9.jpg)

まずはこのモデルをコードとして表現してみましょう。  
`app` ディレクトリに `Gacha`というディレクトリを用意します。

```shell
$ mkdir app/Gacha
```

次に `app/Gacha` ディレクトリに `Gacha.php`, `Item.php`, `Prize.php` の3つのファイルを用意します。

```php
<?php declare(strict_types=1);
// app/Gacha/Gacha.php

namespace App\Gacha;

class Gacha
{
}
```

```php

<?php declare(strict_types=1);
// app/Gacha/Item.php

namespace App\Gacha;

class Item
{
}

```

```php
<?php declare(strict_types=1);
// app/Gacha/Prize.php

namespace App\Gacha;

class Prize
{
}

```

## 引くというメソッドを用意する


まずは引くというメソッドを用意します。

```php
// app/Gacha/Gacha.php

<?php declare(strict_types=1);

namespace App\Gacha;

class Gacha
{
    public function draw(): Item
    {
    }
}
```

次にテストを生成します。

```shell
$ docker-compose exec php php artisan make:test Gacha/GachaTest --unit
```

`tests/Unit/Gacha/GachaTest.php`が生成されるのでテストを書きましょう。

```php
// tests/Unit/Gacha/GachaTest.php

<?php

namespace Tests\Unit\Gacha;

use App\Gacha\Gacha;
use App\Gacha\Item;
use PHPUnit\Framework\TestCase;

class GachaTest extends TestCase
{
    /**
     * A basic unit test example.
     *
     * @return void
     */
    public function testDraw(): void
    {
        $gacha = new Gacha();
        $actual = $gacha->draw();
        $this->assertInstanceOf(Item::class, $actual);
    }
}
```

テストを実行しましょう。

```shell
$ docker-compose exec php php artisan test tests/Unit/Gacha/GachaTest.php  

   FAIL  Tests\Unit\Gacha\GachaTest
  ⨯ example

  ---

  • Tests\Unit\Gacha\GachaTest > example
   TypeError 

  Return value of App\Gacha\Gacha::draw() must be an instance of App\Gacha\Item, none returned

  at app/Gacha/Gacha.php:9
      5▕ class Gacha
      6▕ {
      7▕     public function draw(): Item
      8▕     {
  ➜   9▕     }
     10▕ }
     11▕ 

  1   tests/Unit/Gacha/GachaTest.php:19
      App\Gacha\Gacha::draw()


  Tests:  1 failed
  Time:   0.08s

```

当たり前ですが`draw` メソッドの中身を実装していなのでエラーが発生します。  
まずはエラーが出ないように修正しましょう。

```php
<?php declare(strict_types=1);
// app/Gacha/Gacha.php


namespace App\Gacha;

class Gacha
{
    public function draw(): Item
    {
        return new Item();
    }
}
```

もう一度テストを実行すると以下のようにテストが通ります。

```shell
$ docker-compose exec php php artisan test tests/Unit/Gacha/GachaTest.php  

   PASS  Tests\Unit\Gacha\GachaTest
  ✓ example

  Tests:  1 passed
  Time:   0.07s

```

## 仕様を整理してTODOリストを作る

{% include "../spec/base_spec.md" %}

この言葉からTODOリストを考えてみましょう。

- ガチャを引く
- 10個のアイテムからアイテムを1つ取得
- 確率は0 ~ 10
- 商品一覧を取得できる

ガチャを引くという実装は一旦後回しにして「10個のアイテムからアイテムを1つ取得」という言葉に注目してみましょう。  
これだとちょっと解像度が低いですね。誰がアイテムを10個もつのでしょうか？ ここの裏にはガチャという言葉が隠れています。
さらにstep2で見つけた「景品」という概念とも照らし合わせて考えてみます。


つまり「10個のアイテムからアイテムを1つ取得」は「ガチャは10個の景品を持つことができる」と言い換えられます。

- ガチャを引く
- ガチャは10個の景品を持つことができる
- 確率は0 ~ 10

「確率は0 ~ 10」に関しても「景品に設定されている」という言葉抜けています。
ここは「景品には0 ~ 10の確率の値が設定される」という定義のほうがしっくりきますね。

- ガチャを引く
- ガチャは10個の景品を持つことができる
- 景品には0 ~ 10の確率の値が設定される


## 「ガチャは10個の景品を持つことができる」を実装する

- 景品を追加できるaddメソッドを用意する
- アイテムが10個あるかを確認するメソッド 

上記のメソッドを用意します。


```php
<?php declare(strict_types=1);

// testse/Unit/Gacha/GachaTest.php

namespace Tests\Unit\Gacha;

use App\Gacha\Gacha;
use App\Gacha\Item;
use App\Gacha\Prize;
use PHPUnit\Framework\TestCase;

class GachaTest extends TestCase
{
    /**
     * A basic unit test example.
     */
    public function testDraw(): void
    {
        $gacha = new Gacha();
        $actual = $gacha->draw();
        self::assertInstanceOf(Item::class, $actual);
    }

    public function testAddPrize(): void
    {
        $gacha = new Gacha();
        $gacha->addPrize(new Prize());
    }

    public function testHasPrizes(): void
    {
        $gacha = new Gacha();
        foreach (range(1, 10) as $_) {
            $gacha->addPrize(new Prize());
        }
        self::assertTrue($gacha->hasPrizes());
    }
}
```

これを実行するとメソッド自体実装されていないのでエラーになります。

```shell
$ docker-compose exec php php artisan test tests/Unit/Gacha/GachaTest.php

...
  Tests:  2 failed, 1 passed
  Time:   0.16s
```

次にテストが通るようにメソッドを実行しましょう。


```php
<?php declare(strict_types=1);
// app/Gacha/Gacha.php

namespace App\Gacha;

class Gacha
{
    /**
     * @var Prize[]
     */
    private $prizes = [];

    public function draw(): Item
    {
        return new Item();
    }

    public function addPrize(Prize $prize): void
    {
    }

    public function hasPrizes(): bool
    {
        return true;
    }
}
```

これでテストが通るようになりました。
次にちゃんと実装をしていきましょう。

まずは `addPrize` を修正していきます。

```php
<?php declare(strict_types=1);
// app/Gacha/Gacha.php

namespace App\Gacha;

class Gacha
{
    /**
     * @var Prize[]
     */
    private $prizes = [];

    public function draw(): Item
    {
        return new Item();
    }

    public function addPrize(Prize $prize): void
    {
        $this->prizes[] = $prize;
    }

    public function hasPrizes(): bool
    {
        return true;
    }
}

```

テストを実行して問題がなければ、次に `hasPrizes` を実装しましょう。
上限を超えていた場合は `addPrize` 実行時に、エラーにしたいので`addPrize`も同時に修正します。

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

    public function draw(): Item
    {
        return new Item();
    }

    public function addPrize(Prize $prize): void
    {
        if ($this->hasPrizes()) {
            throw new \Exception('景品の上限を超えています')
        }
        $this->prizes[] = $prize;
    }

    public function hasPrizes(): bool
    {
        return count($this->prizes) === self::MAX_PRIZE;
    }
}
```

これでテストを実行すると、成功します。

### 10という数字を定数にする理由について
景品が10個入っているかどうかの判定を `self::MAX_PRIZE` という形で表しているのはなぜでしょうか？  
それが10という数字が用意に変わる可能性があるためです。  
一旦定数として定義するようにしておいて、あとでなにかあっても対応が出来るようにします。
全てではないんですが具体的な数値が出てきたときは気をつけたほうがいいです。変化しやすいので。


## 景品には0 ~ 10の確率の値が設定されるを実装しよう

「ガチャは10個の景品を持つことができる」というタスクはとりあえず消化できたので一旦完了したことにしましょう。

- ガチャを引く
- ~~ガチャは10個の景品を持つことができる~~
- 景品には0 ~ 10の確率の値が設定される

つぎに、景品に確率を設定する実装をします。

```php
<?php declare(strict_types=1);
// app/Gacha/Prize.php

namespace App\Gacha;

class Prize
{
    /**
     * @var Item
     */
    private $item;

    /**
     * @var int
     */
    private $probability;

    /**
     * Prize constructor.
     * @param Item $item
     * @param int $probability
     */
    public function __construct(Item $item, int $probability)
    {
        $this->item = $item;
        $this->probability = $probability;
    }

    /**
     * @return Item
     */
    public function getItem(): Item
    {
        return $this->item;
    }

    /**
     * @return int
     */
    public function getProbability(): int
    {
        return $this->probability;
    }
}

```

### Prizeクラスのテストを書くべきか？

これは結構難しい問題ですが、Prizeのテストを書いたところで今のところは費用対効果が悪いので今回は書きません。
※出来る限りテストは書いたほうがいいのですが、今回は重要でない箇所のテストを省略します。

## GachaTestの修正

今の状態でGachaのテストを実行するとエラーになります。

```shell
$ docker-compose exec php php artisan test tests/Unit/Gacha/GachaTest.php

...

  Tests:  2 failed, 1 passed
  Time:   0.17s
```

Prizeクラスにコンストラクタが必要になったので、修正しましょう。
値は一旦型さえあっていればなんでもいいです。

```php
<?php declare(strict_types=1);
// app/Gacha/Gacha.php

namespace Tests\Unit\Gacha;

use App\Gacha\Gacha;
use App\Gacha\Item;
use App\Gacha\Prize;
use PHPUnit\Framework\TestCase;

class GachaTest extends TestCase
{

    public function testDraw(): void
    {
        $gacha = new Gacha();
        $actual = $gacha->draw();
        self::assertInstanceOf(Item::class, $actual);
    }

    public function testAddPrize(): void
    {
        $gacha = new Gacha();
        $gacha->addPrize(new Prize(new Item(), 10));
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

これでテストが通ります。

## 「ガチャを引く」を実装しよう

- ガチャを引く
- ~~ガチャは10個の景品を持つことができる~~
- ~~景品には0 ~ 10の確率の値が設定される~~

次はガチャのコアの部分「ガチャを引く」を実装しましょう。
ロジックとしては一般的な抽選のロジックで実装します。

1. 1 ~ 確率のトータルの中でランダムに1つ値を生成する
1. 確率を順に足していき、境界を超えたところの景品を取得する

みたいな感じで`draw()`メソッドの中に実装します。


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
    private $prizes;

    /**
     * @return Item
     * @throws \Exception
     */
    public function draw(): Item
    {
        $totalProbability = array_reduce($this->prizes, static function (int $ac, Prize $prize) {
            return $ac + $prize->getProbability();
        }, 0);
        $boundary = random_int(1, $totalProbability);
        $countPriority = 0;

        foreach ($this->prizes as $prize) {
            $countPriority += $prize->getProbability();

            if ($boundary <= $countPriority) {
                return $prize->getItem();
            }
        }

        throw new \RuntimeException('item not found.');
    }

    public function addPrize(Prize $prize): void
    {
        if ($this->hasPrizes()) {
            throw new \Exception('景品の上限を超えています')
        }
        $this->prizes[] = $prize;
    }

    public function hasPrizes(): bool
    {
        return count($this->prizes) === self::MAX_PRIZE;
    }
}
```

テストも修正していきましょう。

```php
<?php declare(strict_types=1);
// tests/Unit/Gacha/Gacha.php

namespace Tests\Unit\Gacha;

use App\Gacha\Gacha;
use App\Gacha\Item;
use App\Gacha\Prize;
use PHPUnit\Framework\TestCase;

class GachaTest extends TestCase
{

    public function testDraw(): void
    {
        $gacha = new Gacha();
        foreach (range(1, 10) as $_) {
            $gacha->addPrize(new Prize(new Item(), 10));
        }
        $actual = $gacha->draw();
        self::assertInstanceOf(Item::class, $actual);
    }

    public function testAddPrize(): void
    {
        $gacha = new Gacha();
        $gacha->addPrize(new Prize(new Item(), 10));
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

## テストについて深く考えてみる

ここで先程書いたテストについて更に考察して見ましょう。


## testAddPrizeについて考える
このテストは必要なのか？

```php
    public function testAddPrize(): void
    {
        $gacha = new Gacha();
        $gacha->addPrize(new Prize(new Item(), 10));
    }
```

assertionつまり検査をしていないのでほぼ意味がないですね。  
addPrizeの引数も型で保証されているため特にテストする必要を感じません。
削除してしまいましょう。

## testDrawについて考える

testDrawにはテストとしての欠陥があります。

```php
    public function testDraw(): void
    {
        $gacha = new Gacha();
        foreach (range(1, 10) as $_) {
            $gacha->addPrize(new Prize(new Item(), 10));
        }
        $actual = $gacha->draw();
        self::assertInstanceOf(Item::class, $actual);
    }
```

何が問題なのか？  
それはこのテストで検査している項目が型のチェックだけだということです。

ではどのようなテストを書けばいいのでしょうか？

まず着目するのが `darw` メソッドの中で `$prize->getProbability()` がコールされていること。
確率が正しく出るかどうかをユニットテストで検査するのはすごく難しいです。  
なんで、どのようなメソッドがコールされているかというところに着目したほうがこの段階ではいいです。

**オブジェクトをMockしてテストを振る舞いを検査する。** 

という解決策でテストを書いてみましょう。

```php
    public function testDraw(): void
    {
        $gacha = new Gacha();
        $item = new Item();

        $prize = $this->prophesize(Prize::class);
        $prize->getProbability()->willReturn(1);
        $prize->getItem()->willReturn($item);

        $gacha->addPrize($prize->reveal());
        $gacha->draw();

        $prize->getProbability()->shouldHaveBeenCalledTimes(2);
        $prize->getItem()->shouldHaveBeenCalledTimes(1);
    }
```

これがいわゆるSpy方式のテストと呼ばれるものです。  
メソッドから返される値をチェックするのではなく、引数で渡したオブジェクトがどう振る舞うかをテストしています。

 `$this->prophesize(Prize::class)`  -> モックオブジェクトを作成します。  
 `$prize->getProbability()->shouldHaveBeenCalledTimes(2)` -> getProbabilityメソッドが2回呼び出されたかを確認します。


