# オブジェクト指向とはなにか？ ~ ガチャの仕組みから考える ~

オブジェクト指向とはなにか？  
これは一言ではすごく難しい上に、メッセージング指向の文脈、抽象データ型の文脈のそれぞれで使われていて難しいです。

ここでは「現実世界の様々なもの」をプログラミングの世界で表現し、何かしらのモデルを作って、そのモデルを操作させながらソフトウェアとして動かす。
みたいな世界観で考えています。

※ネットの記事に左右されず、オブジェクト指向に関しては書籍をしっかりと読み込むことが重要になります。

## オブジェクト指向のプログラミング的な側面

オブジェクト指向と聞くと、大体の書籍やドキュメントに記載してあるのが以下の言葉です。

- クラス
- 継承
- ポリモフィズム

ただ、これらの技術を組み合わせたらオブジェクト指向かといわれるとちょっと違います。というかだいぶ違います。

重要なのは「オブジェクトをどう表現していくか」「オブジェクトとオブジェクトの関係性がどうなっているか」というところです。

### 余談: 手続き型とはなにか？
よくオブジェクト指向と比較されるプログラミングスタイルです。  
手順(タスク)をひとつひとつ実行して処理していく実装方法です。
よくフローチャートで表現して、それを実装していくみたいなスタイルが昔はありましたがそれも手続き型と言えるでしょう。

再利用性の観点から行くと手続き型は非常に難しいです。


# ガチャを例にオブジェクト指向的に考えてみる

そもそもガチャとはなんでしょうか？

![](/images/image6.png)

最初にイメージするのはこういったものを引く仕組みです。

## 抽象的に考えてみる

ガチャが実現していることはなにかを一言で表すと  
**「10種類の商品の中からランダムで1つなにか商品を引くことができ、商品を1つ入手できる」 **
ということになります。

アナログなガチャをソフトウェアとして表現する場合には抽象化して考える必要があります。

まずは、
先程の言葉「10種類の商品の中からランダムで1つなにか商品を引くことができ、商品を1つ入手できる」を分解して考えてみます。

- 10種類
- 商品
- ランダム
- 1つ
- 引く

だいたい上記のように分解できます。

さて、ここで重要なのはソフトウェアとして表現にする上で必要かどうかを考えてみます。  
まずは ** 「引く」「ランダム」「1つ」** という言葉について考えてみましょう。

ランダム、1つというのは、引くという行為の言葉があって初めて成り立つものです。
つまり「ランダム、1つ」というのは引くという行為に対する ** 「条件」 ** ということになります。

更に **「引く」** という行為をもう少し考察してみましょう。
「引く」という行為は単体では存在できません。引くという言葉を表現する場合は大抵が「xxを引く」を引くと表現しますよね？  
つまり何かないと成り立たない行為となります。

**「ガチャを引く」 ** と表現することからしてもガチャというモノ(Object)に付随する機能ということになります。

整理すると以下の用な図になります。

![](/images/image7.jpg)

あとはコレをクラスなどのプログラミング言語の機能を使って実装していけばいいです。


## データモデル(データベース)の文脈でガチャを捉える

先程はオブジェクト指向の文脈からガチャの構造を洗い出してみました。
しかしそれだけではダメです。

webアプリケーションはRDBMSなどのデーターベースなどにデータを保存します。
そこで必要になってくるのがデータモデルです。

ただしデータモデルとオブジェクトモデルは結構似たものになります。

## 先程のオブジェクトモデルから導き出すテーブル構造

まずは素直にオブジェクトモデルからテーブル構造を考えてみましょう。

- ガチャテーブル
- 商品テーブル

の2つになり、構造は以下の通りになります。


## ガチャテーブルに関する考察

オブジェクトモデルから想定されるガチャテーブルは以下のようになる可能性があります。

#### 商品テーブル

|  カラム      |　型           | 説明        |
| ----------- | ------------ | ---------- |
| id          | int          | 一意なID    |
| name        | varchar(255) | 商品名 |


#### ガチャテーブル

|  カラム      |　型           | 説明        |
| ----------- | ------------ | ---------- |
| id          | int          | 一意なID    |
| name        | varchar(255) | ガチャの名前 |
| item_1_id   | int       | 商品1 ID     |
| item_2_id   | int       | 商品2 ID     |
| item_3_id   | int       | 商品3 ID     |
| item_4_id   | int       | 商品4 ID     |
| item_5_id   | int       | 商品5 ID     |
| item_6_id   | int       | 商品6 ID     |
| item_7_id   | int       | 商品7 ID     |
| item_8_id   | int       | 商品8 ID     |
| item_9_id   | int       | 商品9 ID     |
| item_10_id  | int       | 商品10 ID     |

これでも ** 10個の商品を持つ ** ガチャとしては成り立ちます。  
しかし本当にこれでいいのでしょうか？  
例えば「あるイベントだけガチャの商品の種類を20個にして欲しい」みたいな仕様変更があった場合に対応できません。

## ガチャテーブルをどのように分解するのか？

データベースの正規化という考え方に則ってテーブルを整理します。
基本的には重複を排除していくという考え方です。

つまり商品とガチャが紐づくテーブルを作るのがよいです。

### ガチャテーブル

|  カラム      |　型           | 説明        |
| ----------- | ------------ | ---------- |
| id          | int          | 一意なID    |
| name        | varchar(255) | ガチャの名前 |

### ガチャ商品テーブル

|  カラム      |　型           | 説明        |
| ----------- | ------------ | ---------- |
| id          | int          | 一意なID    |
| gacha_id    | int          | ガチャID    |
| item_id    | int          | アイテムID    |

![](/images/image8.jpg)


## 再びオブジェクトを再考する

データベースの視点で考えたときに ** 「商品」「ガチャ商品」「ガチャ」 ** と3つのテーブルを発見しました。  
さてここでオブジェクトのモデルを見てみましょう。

![](/images/image7.jpg)

ガチャで引くと商品が手にはいるが、商品オブジェクトでいいのだろうか？  
ガチャの中に入っているものはなにか？

ここでもう一度仕様を思い出してみよう。

> 商品ごとに取得できる確率が設定でき、確率は整数で重み付けすることができる

最初ガチャを想像したときに「ランダム」という言葉を使っていましたが、ランダムにも仕様があります。
この仕様から考えると、「ガチャの中に入っている商品」というものに必要なパラメータは「商品を指すID」と「確率」がセットなったものと考えられます。
そして、このセットに名前をつけるとしたら「景品(prize)」という言葉がよさそうです。

図にするとこのようになります。

![](/images/image9.jpg)

## ガチャ商品テーブル

さらに確率に関しては、テーブルも修正する必要があります。


### 景品テーブル

|  カラム      |　型           | 説明        |
| ----------- | ------------ | ---------- |
| id          | int          | 一意なID    |
| gacha_id    | int          | ガチャID    |
| item_id    | int          | アイテムID    |
| probability | smallint         | 確率        |


![](/images/image10.jpg)

## まとめ

- オブジェクト指向と手続き型の実装方法がある
- オブジェクト指向は実装のテクニックだけではない
- 言葉を大切にして、抽象化していくことが大切
- オブジェクトモデルだけではなく、データモデルという観点からも考えることが重要 


### 余談: 言葉の大切さ
オブジェクト指向では概念をコードで表現しながら実装していきます。  
概念を知るためにも言葉が重要になってきます。実際のプロダクト開発では言葉の定義には時間をかけています。
最近流行っているDDDなんかでも「ユビキタス言語」というものがありますが、つまりそれは言葉の定義を定める、概念を統一するという事なのです。
言葉、プロジェクトに関わる人同士の言葉の認識が違うとバクや求めてないシステムを作ってしまったり様々な問題が発生します。

ある意味ではプログラミングの実装テクニックなんかよりもぜんぜん重要になります。
言葉、言葉の定義を大事にしてください。

