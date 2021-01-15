# 認証機能を実装しよう

webアプリケーション開発では絶対的に必要になる認証機能。
認証にはいくつかの種類が存在しますが、今回はパスワード使ったログイン方法で実装してきます。

ほかにもOAuth認証や、JWT認証などもありますが、今回は一般的なセッションを使った認証を作りましょう。



## そもそもパスワードを使った認証の仕組みとはどのようなものか？
大まかに説明すると、パスワードをDBに格納しておき、ユーザーが入力したパスワードとDBに格納されているパスワードがあっているかをチェックする。
チェックして大丈夫だったら、セッションにログインしたことを記録する。

俗にセッション認証と呼ばれる実装方法ですね。


## セッション認証とはなにか

一言で表すと  
** CookieにセッションIDを保持して、セッションIDを元にユーザーを特定させる認証方法 ** になります。

### Cookieとはなにか

そもそもCookieとは何でしょうか？
基本的にHTTPの技術はステートレス、つまり状態を持たないようになっています。  
ただ、状態をもたないとアクセスしてきたユーザーがだれなのか特定ができません。

そこで利用するのがCookieです。  
Cookieは様々な情報を格納できます。

便利ですが、欠点があります。それはユーザーが勝手に書き換えることができてしまうことです。

### セッションとはなにか？

セッションとは簡単に言うと「サーバ側に情報を保持する仕組み」になります。  
Cookieとは違って、セッションはサーバ側で情報を保持するので、ユーザーが勝手に書き換える事はできません。


### セッションIDとはなにか？
セッションが格納されてる情報にアクセスするためのIDになります。
このIDがなければセッション情報を取得できません。

### セッションIDはCookieで保持される
セッションIDはCookieにより管理されます。
HTTPリクエスト時にブラウザはCookieを送信します。そのCookieのなかに入っているセッションIDを使ってログインしているユーザーかを認証します。

ここで疑問なのは「Cookieに適当なセッションIDをセットすれば他人になりすませるのでは？」という事です。
あとは他人のセッションIDをどこかで入手すれば、なりすませるのでは？
たしかにその通りなんですが、セッションは基本的には通信するときに毎回新しいセッションIDを発行し盗まれても問題ありません。
(いや問題はあるので、盗まれないようにHTTPSを使うことは今の時代必須かと。またいろんな攻撃によりセッションIDが漏洩することもあるのでセキュリティ気をつけて実装しないとだめです)


## 実際にLaravelの認証機能をつかってセッション認証を作ってみよう

まずはデータベースのテーブルを作成します。
Larvelには認証に必要なテーブルのマイグレーションが用意されているのでそれを今回は使います。

```shell
$ docker-compose exec php php artisan migrate    

Migration table created successfully.
Migrating: 2014_10_12_000000_create_users_table
Migrated:  2014_10_12_000000_create_users_table (64.16ms)
Migrating: 2014_10_12_100000_create_password_resets_table
Migrated:  2014_10_12_100000_create_password_resets_table (41.82ms)
Migrating: 2019_08_19_000000_create_failed_jobs_table
Migrated:  2019_08_19_000000_create_failed_jobs_table (48.92ms)
```

次に認証をつくるためには `laravel/ui` というパッケージが必要になります。  
composerを使ってインストールしましょう。

```shell
$ docker-compose exec php composer req laravel/ui 
```

次に `artisan ui:auth`コマンドを使って必要なファイルを生成します。

```shell
$  docker-compose exec php php artisan ui:auth
```

ここで http://localhost/home にアクセスしてみましょう。
すると以下のような画面ができます。

<img src="/images/image2.png" style="border: 1px solid #222; width: 100%">


これだと見た目がおかしいので修正します。  
以下のコマンドを入力することでbootstrapな見た目のUIになります。

```shell
$ docker-compose exec php php artisan ui bootstrap  
$ docker-compose exec php npm install  
$ docker-compose exec php npm run dev
```

※ npm関連はdocker経由だと遅いので、必要ならローカル環境のものを使ってください。


これで見た目が綺麗になりました。

<img src="/images/image3.png" style="border: 1px solid #222; width: 100%">


## 実際にうごかしてみる
[ユーザー登録](http://localhost/register) して [ログイン](http://localhost/login) してみましょう。

## 実際の認証はどこで行われているのか？

簡単に機能を用意できましたが、実際はどのようにログイン認証をしているのでしょうか？
少しコードをみてみましょう。

### 1. まずはルーティングでコントローラと認証をの関係性を見つめてみる

`artisan route:list` コマンドで確認してみましょう。

```shell
$ docker-compose exec php php artisan route:list

+--------+----------+------------------------+------------------+------------------------------------------------------------------------+------------+
| Domain | Method   | URI                    | Name             | Action                                                                 | Middleware |
+--------+----------+------------------------+------------------+------------------------------------------------------------------------+------------+
|        | GET|HEAD | /                      |                  | Closure                                                                | web        |
|        | GET|HEAD | api/user               |                  | Closure                                                                | api        |
|        |          |                        |                  |                                                                        | auth:api   |
|        | GET|HEAD | home                   | home             | App\Http\Controllers\HomeController@index                              | web        |
|        |          |                        |                  |                                                                        | auth       |
|        | GET|HEAD | login                  | login            | App\Http\Controllers\Auth\LoginController@showLoginForm                | web        |
|        |          |                        |                  |                                                                        | guest      |
|        | POST     | login                  |                  | App\Http\Controllers\Auth\LoginController@login                        | web        |
|        |          |                        |                  |                                                                        | guest      |
|        | POST     | logout                 | logout           | App\Http\Controllers\Auth\LoginController@logout                       | web        |
|        | GET|HEAD | password/confirm       | password.confirm | App\Http\Controllers\Auth\ConfirmPasswordController@showConfirmForm    | web        |
|        |          |                        |                  |                                                                        | auth       |
|        | POST     | password/confirm       |                  | App\Http\Controllers\Auth\ConfirmPasswordController@confirm            | web        |
|        |          |                        |                  |                                                                        | auth       |
|        | POST     | password/email         | password.email   | App\Http\Controllers\Auth\ForgotPasswordController@sendResetLinkEmail  | web        |
|        | GET|HEAD | password/reset         | password.request | App\Http\Controllers\Auth\ForgotPasswordController@showLinkRequestForm | web        |
|        | POST     | password/reset         | password.update  | App\Http\Controllers\Auth\ResetPasswordController@reset                | web        |
|        | GET|HEAD | password/reset/{token} | password.reset   | App\Http\Controllers\Auth\ResetPasswordController@showResetForm        | web        |
|        | GET|HEAD | register               | register         | App\Http\Controllers\Auth\RegisterController@showRegistrationForm      | web        |
|        |          |                        |                  |                                                                        | guest      |
|        | POST     | register               |                  | App\Http\Controllers\Auth\RegisterController@register                  | web        |
|        |          |                        |                  |                                                                        | guest      |
+--------+----------+------------------------+------------------+------------------------------------------------------------------------+------------+
```

注目すべきはここのルーティングの設定になります。

```
 | POST     | login                  |                  | App\Http\Controllers\Auth\LoginController@login                        | web        |
```

ここで`App\Http\Controllers\Auth\LoginController` をみてると実際の記述はないのですが、よくみると `use AuthenticatesUsers` しています。  
`Illuminate\Foundation\Auth\AuthenticatesUsers`をみてみると、`login`メソッドが記述されています。

![](/images/image4.png)

更にコードを読み進めていくと、`Illuminate\Auth::attempt()`で、認証処理をしていることがわかります。

![](/images/image5.png)


retrieveByCredentialsでメールアドレスを元にDBからユーザー情報を取得しています。

```php
$this->provider->retrieveByCredentials($credentials)
```

hasValidCredentials入力されたパスワードと、ハッシュ値があっているかを確認しています。

```php
 $this->hasValidCredentials($user, $credentials)
```

そして`login()` メソッドでセッションにデータを格納しています。


コマンドで簡単に認証の機能を用意できますが、ちゃんと理解するためにも一度はコードリーディングを行うことが重要です。

## TOPページにアクセスした場合にログインしていない場合はログインページにリダレイクトさせる

ログインしているかどうかを判定する方法はいくつかあります。  
しかし基本的には`auth` middlewareでチェックしています。

いくつか設定する方法ありますが、今回はルーティングファイルに記載してきましょう。

`routes/web.php`を編集します。

```php
// routes/web.php

Route::middleware('auth')->group(function (): void {
    Route::get('/', function () {
        return view('welcome');
    });
});
```

このように設定し、保存したらTOPページにアクセスしてみましょう。
ログインしていない場合はログインページにリダイレクトされます。

## 余談: JWT認証について
余談ですが、最近だとJWTトークンを使った認証を使うことも増えてきました。
余裕がある方は、認証を後からJWTにしてもいいかもですね。

JWT認証の場合、大半は認証サービス基盤を使うことが多いです。
代表的なところで行くと下記のサービスになります。

- Firebase Authentication
- Auth0
- Amazon Cognito
