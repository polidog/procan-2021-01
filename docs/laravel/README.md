# Laravelについて

今回はWebフレームワークにLaravelを使います。
採用した理由は、下記のとおりです。

- Laravelは最近人気があるフレームワークでありドキュメントが多い
- 機能が揃ってるので開発がしやすい

## Laravelチュートリアル

Laravelの大枠を理解したい方は、下記のサイトを参照するとよいかと思います。  
[Laravel入門 - 使い方チュートリアル -](https://qiita.com/sano1202/items/6021856b70e4f8d3dc3d)


## MVCフレームワークについて

LaravelはいわゆるWeb MVCフレームワークになります。
(本来のMVCとは若干ことなるのでそこは気をつけてください)

- M -> モデル: Larvelの場合はEloquentをモデルとしているので、どちらかというとデータモデルになります。
- V -> ビュー: 見た目、表示の処理の部分になります。LaravelではbladeというテンプレートエンジンがViewの部分に相当します。
- C -> コントローラ: HTTPリクエストを受け取り、必要なモデルを取得したり、ビューを選択してレスポンスするなどの役割を担います。

本来のMVCがなんである方を知りたい方は下記の記事を読んでください。  
[MVCとはなにか](https://note.com/tenjuu99/n/n0232ccd1089d#q0bYG)


## Eloquentについて

EloquentはORマッパーと呼ばれる、データベースのデータとオブジェクトを紐付けるためのソリューションです。  
ActiveRecordモデルを採用しているORMになります。

今回のハンスオンのなかで触っていくのでそこで使い方等を覚えてもらえればいいのですが、下記のページも目を通してもらえると理解しやすいです。

https://readouble.com/laravel/8.x/ja/eloquent.html


その他参考: https://qiita.com/shosho/items/5ca6bdb880b130260586


## Blade

基本的な使い方は下記のサイトに記載があります。
今回はそこまでメインな部分ではないのですが、一応記載しておきます。

https://readouble.com/laravel/8.x/ja/blade.html


## artisanコマンドについて

Laravelにはartisanコマンドというものがあります。
雛形のプログラムを生成したり、ルーティング情報を確認したり様々な場面で活用できます。

```shell
$ php artisan list
```

上記のコマンドを実行すれば使えるコマンド一覧が表示されます。


### 余談: 人気があるフレームワークが良いフレームワークなのか？

人気があるフレームワークだからという理由だけで仕事(プロジェクト)で採用すると大変なことになる場合が多々見受けられます。  
フレームワークには特性があります。作るものをみて決めるべきです。  
[Symfonyフレームワーク](https://symfony.com/)を使うと堅牢なアプリケーションが作りやすいのでおすすめです。  
興味があったら触ってみてください。  

また国産フレームワーク[BEAR Sunday](https://bearsunday.github.io/index.html)もおすすめなので一度さわってみるといいかもしれません。  

