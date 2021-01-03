## 開発環境について


github上にひな形となるコードを用意してあります。  
[procan-2021-01-code](https://github.com/polidog/procan-2021-01-code)

基本的には `laravel new` で生成した雛形のコード + docker-composeの設定を用意しただけのものになります。


## ソースコードのチェックアウト

今回は `~/procan-2020-01-code` のディレクトリで作業をしていくという想定で進めていきます。  
必要に応じて読み替えてください。


```bash
$ mkdir -p ~/procan-2020-01-code
$ cd ~/procan-2020-01-code
$ git clone https://github.com/polidog/procan-2020-01-code.git
```

## まずは動かしてみる

ソースコードを入手したら次はdockerを使って実際にプログラムを動かしてみましょう。

```
$ cd ~/src/github.com/polidog/procan-2020-01-code
$ docker-compose up -d
```

Dockerが無事に起動したことを確認したら、[http://localhost:8000](http://localhost:8000) にアクセスして動いていることを確認してください。  
アクセス成功すると以下のページが表示されます。

![](/images/image1.png)