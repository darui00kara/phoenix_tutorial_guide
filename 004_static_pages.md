# ほぼ静的なページ

本章から本格的なPhoenixアプリケーションの開発を始めます。
残りのチュートリアルでは、ここで作成したアプリケーションを元に進めていきます。
最終的には、ユーザやマイクロポスト、ログイン/ログアウトなどの機能を持ちますが、
ここではPhoenixフレームワークにおける、一番簡単な静的なページの追加から始めていきたいと思います。
非常に単純なことをやりますが、静的なページの追加から始めることは良いスタート地点です。

## ゴール

この章で学べること...

- Phoenixアプリケーションに静的なページを追加する方法
- ユニットテストの使い方

## 概要

Railsでもそうですが、Phoenixでもデータベースと連携して動的なWebページを開発するように設計されています。
だからと言って、HTMLやテキストだけの静的なページを作ることができないわけではありません。
実際、あえて静的なページを使用しておき、後から動的なコンテンツを追加することもできます。
本章では、このような静的ページの作成について学んでいきます。
また、ExUnit(ユニットテスト)を利用したテストについても少しだけ触りたいと思います。
(テストの雰囲気をつかむことが目的ですので、深くはやりませんが...)

## 準備

それでは、Phoenixアプリケーションの作成から開始します。
前章でも行ったように、作業用のブランチを切ってから作業を行っていきます。
この作業用のブランチを切るのは章の始めに毎回行います。
理由としては、マスターに影響を及ぼさずに作業ができること、何かがあったときに切り捨てやすいからです。
マスターを直接触って、手戻りをしなければいけないときはどうしましょうか？
そんなときにコマンド一発でブランチを捨てれば終わりだと楽ですよね。
皆さんも作業をするときは面倒くさがらずに作業用ブランチを切ることを推奨します。
それでは、前章まででやっていたことをおさらいするつもりで実施していきましょう。

### 作業用のブランチを切る

```cmd
$ cd path/to/application/directory
$ git branch
  demo_app
* master
$ git branch static_pages
$ git branch
  demo_app
* master
  static_pages
$ git checkout static_pages
$ git branch
demo_app
  master
* static_pages
```

今回は合間に確認を行っていますが、慣れてくれば上記の作業を簡略化してもよいと思います。
次は、これから作成していくサンプルアプリケーションの作成を行います。

### サンプルアプリケーションを作成する

```cmd
$ mix phoenix.new sample_app
...
Fetch and install dependencies? [Yn] y
* running mix deps.get
* running npm install && node node_modules/brunch/bin/brunch build

We are all set! Run your Phoenix application:

    $ cd sample_app
    $ mix phoenix.server

You can also run your app inside IEx (Interactive Elixir) as:

    $ iex -S mix phoenix.server

Before moving on, configure your database in config/dev.exs and run:

    $ mix ecto.create

$ cd sample_app
$ mix ecto.create
$ mix phoenix.server
<C-c>
```

起動まで確認できれば問題ありません。
それでは、静的ページを追加するための方法について説明していきます。

## 静的なページ

皆さんは静的なページというとどういったものを想像するでしょうか？
JavaScriptがないこと？HTMLのみで構成されていること？あるいはテキストのみでしょうか？
ここではHTML(EEx)で構成されてた静的ページを表示できるようにしていきます。
Phoenixにはカスタムタスクが用意されていますので、本来であればコマンドでほぼ生成することができます。
しかし、ここではあえて本来は何をしなければいけないのかを含め学習していきましょう。
そうすれば、何か問題があったときの一助になると思います。

### ルーティングの追加

新しいページを追加するときに最初にやらなければいけないのはルーティングの追加です。
デモアプリでもやりましたが、デモアプリのルーティング記述では一つのルーティングだけを追加するということができません。
ここでは、一つのルーティングだけを追加する方法を見てみましょう。

#### Note: :only、:exceptを使えばできなくもないですが、それなら一つだけのルーティングを用意する方がコードとして建設的です。

router.exに新しくルーティングを追加してみます。

#### File: web/router.ex

```elixir
defmodule SampleApp.Router do
  ...

  scope "/", SampleApp do
    pipe_through :browser # Use the default browser stack

    get "/", PageController, :index
    get "/home", StaticPageController, :home
  end

  ...
end
```

これでhomeページのルーティングが追加できました。
追加したルーティング反映されているか確認してみましょう。

#### Example:

```cmd
$ mix phoenix.routes
       page_path  GET  /      SampleApp.PageController :index
static_page_path  GET  /home  SampleApp.StaticPageController :home
```

ちゃんと反映されています。
このとき、指定しているコントローラは存在していなくても問題ありません。
(実際にWebサイトとして動かすのであれば、コンパイルエラーもしくは実行時エラーになりますが...)
ルーティングの記述についてもう少し詳しく見てみましょう。
Phoenixにおけるルーティングの記述は次のような構成になっています。

```elixir
get "/home", StaticPageController, :home
```

上記を分解すると、次のようになります。

- get: HTTPメソッド名(マクロ)
- "/home": パス
- StaticPageController: コントローラ名
- :home: アクション名

デモアプリで追加したルーティングはどうなっていたでしょうか？

```elixir
resources "/users", UserController
```

HTTPメソッド名を指定する部分がresourcesになっていますね。
またアクション名の指定をしていません。

上記の記述を使うとRESTfulなルーティングを生成してくれるようになります。
デモアプリで見たルーティングの出力を覚えていますか？
上記の1行で下記のように複数のルーティングを生成しています。

```cmd
$ mix phoenix.routes
...
     user_path  GET     /users                DemoApp.UserController :index
     user_path  GET     /users/:id/edit       DemoApp.UserController :edit
     user_path  GET     /users/new            DemoApp.UserController :new
     user_path  GET     /users/:id            DemoApp.UserController :show
     user_path  POST    /users                DemoApp.UserController :create
     user_path  PATCH   /users/:id            DemoApp.UserController :update
                PUT     /users/:id            DemoApp.UserController :update
     user_path  DELETE  /users/:id            DemoApp.UserController :delete
...
```

少し強引ですが、さきほどの構成に従って表現すると下記のようになります。

- resources: HTTPメソッド名(マクロ)
- "users": パス
- UserController: コントローラ名
- アクション部分: RESTfulに対応するアクション全部

このようになります。
現状では、resourcesを使えば複数のルーティングを作ってくれるといった認識で大丈夫です。
チュートリアルを進めるうちに自由にルーティングを作成できるようになっています。

#### Note: Phoenixを構成しているのはマクロ？

```txt
Phoenixフレームワークの大部分は、Elixirの機能であるマクロで作られています。
実際に、ルーティング機能の大部分がマクロで実装されています。
チュートリアルでは触れることはありませんが、興味があればメタプログラミングに触れてみると面白いと思います。
注意としては、マクロは最後の手段として考えた方が良いです。
マクロ以外の方法でスマートに実装できるなら、そちらを優先した方が良いです。
展開後のソースコードを追うのはなかなか大変です。(実体験)
```

ルーティングの記述について、ある程度つかめたと思います。
続けて、もう一つルーティングを追加してみましょう。

#### File: web/router.ex

```elixir
defmodule SampleApp.Router do
  ...

  scope "/", SampleApp do
    pipe_through :browser # Use the default browser stack

    get "/", PageController, :index
    get "/home", StaticPageController, :home
    get "/help", StaticPageController, :help
  end

  ...
end
```

ルーティングを追加できたら確認をしましょう。

#### Example:

```cmd
$ mix phoenix.routes
       page_path  GET  /      SampleApp.PageController :index
static_page_path  GET  /home  SampleApp.StaticPageController :home
static_page_path  GET  /help  SampleApp.StaticPageController :help
```

### コントローラの作成

続いて、コントローラの作成に入っていきます。
コントローラでは主に、ルーティングで定義したアクション部分の実装を定義します。
ルーティングへ追加したのはhomeアクションとhelpアクションでしたね。
それと同じ名前の関数を定義します。

#### File: web/controllers/static_page_controller.ex

```elixir
defmodule SampleApp.StaticPageController do
  use SampleApp.Web, :controller

  def home(conn, _params) do
    render conn, "home.html"
  end

  def help(conn, _params) do
    render conn, "help.html"
  end
end
```

### ビューとテンプレートの作成

#### File: web/views/static_page_view.ex

```elixir
defmodule SampleApp.StaticPageView do
  use SampleApp.Web, :view
end
```

#### File: web/templates/static_page/home.html.eex

```html
<div class="jumbotron">
  <h2>Welcome to Static Pages Home!</h2>
</div>
```

#### File: web/templates/static_page/help.html.eex

```html
<div class="jumbotron">
  <h2>Welcome to Static Pages Help!</h2>
</div>
```

### 実行

#### Example:

```cmd
$ mix phoenix.server
<C-c>
```

## テストしてみる

作ったページのテストを作成してみましょう。
本来であれば、機能の実装前にテストを作った方が良いですが、
ちゃんとしたテストサイクルを回すのは次の項目で行います。
ここではExUnitをお試しで使ってみたいと思います。

#### File: test/controllers/static_page_controller_test.exs

```elixir
defmodule SampleApp.StaticPageControllerTest do
  use SampleApp.ConnCase

  test "GET /home", %{conn: conn} do
    conn = get conn, "/home"
    assert html_response(conn, 200) =~ "Welcome to Static Pages Home!"
  end

  test "GET /help", %{conn: conn} do
    conn = get conn, "/help"
    assert html_response(conn, 200) =~ "Welcome to Static Pages Help!"
  end
end
```

ConnCaseが入ってからPlugのテストが簡単になりました。
セッションが入ってくるとまた少し大変なのですが、それでもかなり簡単になりました。

実行確認

#### Example:

```cmd
$ mix test
......

Finished in 0.1 seconds
6 tests, 0 failures

Randomized with seed 560404
```

## 少しだけ動的なページ

新しくページを追加しながらテストサイクルを回していきましょう。

### レッド

### グリーン

### リファクタリング

## おわりに

## 参考



