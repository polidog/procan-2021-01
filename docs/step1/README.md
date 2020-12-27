# 既存の環境を動かしてみよう

まずはGithubにあるコードをcloneしてきて、プログラムを実際に動かしてみましょう。

```sh
$ mkdir ~/procon-shizuoka
$ git clone https://github.com/ptyhard/procon-shizuoka-gacha.git
```
※ディレクトリは好きな構成で設定して構いません。

コードを入手したらDocker Composeで実行環境の仮想マシンを立ち上げます。

```
$ docker-compose up -d
```
docker-composeの使い方に関してはこのドキュメントで説明しないので、以下のドキュメントを確認してください。  
[docker と docker-compose の初歩](https://qiita.com/hiyuzawa/items/81490020568417d85e86)


仮想マシンを立ち上げたら `http://localhost:8000` にブラウザでアクセスして動いていることを確認してください。

![](/images/image1.png)
