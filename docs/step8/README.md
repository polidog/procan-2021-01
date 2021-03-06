# 新たな機能を付け足してみよう

自分自身で、付け加えたい機能を考えて実装してみましょう。
例えば「アイテムボックスを拡張するアイテム」「時間帯によってアイテムの確率を変更する」なんてものいいでしょう。
好きに改良してみてください。

その際に守って欲しいプロセスがあります。

- 作りたいものを文章で書いてみる
- どのようなクラスが必要なのか、図で書いてみる
- 図で表現したクラスを実装してみる
- データベースと連携するためにデータモデルを考えてみる
- 実際にデータベースと連携した実装をする

## 作りたいものを文章で書いてみる

ガチャの仕様を箇条書きで記載したように書いてみましょう。
実装するまえに、作りたいものをより明確化させるためにも、まずは文章を書いて整理しましょう。

### どのようなクラスが必要なのか、図で書いてみる

どのようなクラスが必要になるのか図で書いてみましょう。
紙に書くのでもいいです。UMLとかそういったものは気にせず書きましょう。

### 図で表現したクラスを実装してみる

図で書いたもを、まずはプロトタイプとしての実装をしてみましょう。  
このときはDBと連携とか意識しなくても大丈夫です。  
モデルとして正しいか、という視点でまずは実装してみましょう。

### データベースと連携するためにデータモデルを考えてみる

プロトタイプの実装ができたら、次にデータベースと連携させるためにどのようなデータモデルが必要かを考えてみましょう。  
必要に応じてER図を書いてもいいですし、なにかしら整理するために図や表を書いてみるのもいいでしょう。

### 実際にデータベースと連携した実装をする

実際にEloquentをうまく活用してDBを利用した実装をしてみましょう。

# なぜこのようなめんどくさいプロセスなのか？

例えば家を建てる時、建築家は小さな模型で家をつくりますよね？  
設計図を書きますよね？

それと一緒です。

仕事でプログラミングをするということは、とても複雑で難しいです。
さらに予算(費用)というのが決まっています。
だからこそ、作るものを明確にすることによって、無駄のないプログラミングをすることが求められます。  
プログラミングは「解法」です。文章や図などで作るものを考えるという行為は「問題」を発見する行為です。

この「問題」と「解法」という視点で考えることは非常に重要です。  
解法ばかり求めても、よいコードは書けませんしプロダクトも作れません。  
逆に問題ばかり発見できても解決できなければ価値がでません。

その２つを補うためにもこのプロセスを守ってもらいたいです。

私の好きな技術書「マルチパラダイムデザイン」にこのような記載があります。

> 設計とは、ある問題に対して解決策となるような構造を与えるアクティビティのことである(p.9)

https://www.seshop.com/product/detail/2568

コードが書けるということは、日本語が書けるということと一緒です。
日本語が書けたら、素晴らしい小説が誰でも書けるのでしょうか？
それは違いますよね。

よいプログラムを書く、よいサービスを作るためにも「問題」を発見する、「解法」を正しく書けるという２つの要素が重要だと私は考えます。

ぜひ今回のプロセスを守って良いプログラマへの一歩を踏み出してもらえれば嬉しいです。



