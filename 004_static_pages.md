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

それでは0から始めるPhoenixアプリケーション開発の始まりの一歩です。
おっと、Phoenixチュートリアルでしたね。失敬失敬、タイトルを間違えてしまうとは！
前章までにやったことをおさらいするつもりで実施していきましょう。

### サンプルアプリケーションの作成

これからチュートリアルを通して作っていくサンプルアプリケーションを作成しましょう。

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
ここまで進めたら、作業用のGitリポジトリとして初期化し、ブランチを切りましょう。
前章でも行ったように作業用のブランチを切ってから作業を行っていきます。
この作業用のブランチを切るのは章の始めに毎回行います。
理由としては、マスターに影響を及ぼさずに作業ができること、何かがあったときに切り捨てやすいからです。
マスターを直接触って、手戻りをしなければいけないときはどうしましょうか？
そんなときにコマンド一発でブランチを捨てれば終わりだと楽ですよね。
皆さんも作業をするときは面倒くさがらずに作業用ブランチを切ることを推奨します。

### 作業用のブランチを切る

```cmd
$ git init
$ git branch
* master
$ git branch static_pages
$ git branch
* master
  static_pages
$ git checkout static_pages
$ git branch
demo_app
  master
* static_pages
```

今回は合間に確認を行っていますが、慣れてくれば上記の作業を簡略化してもよいです。
ブランチまでできたら、早速コミットをしましょう。
こまめにコミットしていれば、最小限の手間で戻すことができることでしょう。

```cmd
$ git add -A
$ git commit -m "Initialize repository"
```

それでは、Phoenixフレームワークで静的ページを追加するための方法について説明していきます。

## 静的なページ

皆さんは静的なページというとどういったものを想像するでしょうか？
JavaScriptがないこと？HTMLのみで構成されていること？あるいはテキストのみでしょうか？(Hなサイト？それは性的です)
ここではHTML(EEx)で構成されてた静的ページを表示できるようにします。
Phoenixにはカスタムタスクが用意されていますので、本来であればコマンドでほぼ生成することができます。
しかし、ここではあえて本来は何をしなければいけないのかを含め学習していきましょう。
そうすれば、何か問題があったときの一助になると思います。

### ルーティングの追加

新しいページを追加するときに最初にやらなければいけないのはルーティングの追加です。
デモアプリでもやりましたが、デモアプリのルーティング記述では一つのルーティングだけを追加するということができません。
ここでは、一つのルーティングだけを追加する方法を見てみましょう。

#### Note: :only、:exceptを使えばできなくもないですが、それなら一つだけのルーティングを用意する方がコードとして建設的です。

router.exに新しくルーティングを追加してみましょう。

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
1行しか書いていないって？大丈夫です！Phoenixフレームワークが頑張ってくれます。
実際、追加したルーティングが反映されていることを確認してみましょう。

#### Example:

```cmd
$ mix phoenix.routes
       page_path  GET  /      SampleApp.PageController :index
static_page_path  GET  /home  SampleApp.StaticPageController :home
```

ちゃんと反映されていました。(よかった、よかった)
このとき、指定しているコントローラは存在していなくても問題ありません。
(実際にWebページを動かそうとするとコンパイルエラーになりますが)
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
上記の1行で下記のように複数のルーティングを生成していました。

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

現状では、resourcesを使えば複数のルーティングを作ってくれるといった認識で大丈夫です。
チュートリアルを進めるうちに自由にルーティングを作成できるようになっています。

#### Note: Phoenixを構成しているのはマクロ？

```txt
Phoenixフレームワークの大部分は、Elixirの機能であるマクロで作られています。
実際に、ルーティング機能の大部分がマクロで実装されています。
チュートリアルでは触れることはありませんが、興味があればメタプログラミングに触れてみると面白いと思います。
注意としては、マクロは最後の手段として考えた方が良いことです。
マクロ以外の方法でスマートに実装できるなら、そちらを優先した方が良いです。
展開後のソースコードを追うのはなかなか大変なんです。(実体験)
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

続いて、コントローラの作成に入ります。
コントローラでは主に、ルーティングで定義したアクション名と同名のアクション用関数を実装します。
Phoenixフレームワークがないければ、このルーティングとコントローラの結びつきもなくなります。
自分で実装するのは大変ですね・・・Phoenixフレームワークに感謝を捧げましょう！(大げさ)
さて、ルーティングへ定義したのはhomeアクションとhelpアクションでしたね。
それと同じ名前の関数を用意します。

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

少ししかコードを書いていませんから本当に動くのか心配でしょうか？
問題ありません！難しいことのほとんどはPhoenixフレームワークがどうにかしてくれます。

### ビューとテンプレートの作成

Webページにレンダリングされるビューとテンプレートの作成を行っていきましょう。
実際に人が見る画面の部分になります。まずは、ビューから作成していきましょう。
ビューで実装した関数は、テンプレートでも利用することができます。
今回は、特に何もすることがないのでビューを作成するだけになります。

#### File: web/views/static_page_view.ex

```elixir
defmodule SampleApp.StaticPageView do
  use SampleApp.Web, :view
end
```

#### Note: 

```txt
ビューはレンダリングのときに必ず必要になります。
テンプレートに対応したビューがないとレンダリングできませんので注意して下さい。
```

続けてテンプレートの作成をしましょう。
まずはテンプレートを格納するディレクトリの作成からです。
"static_page"と言う名称でディレクトリを作成して下さい。

#### Directory: web/templates/static_page

ディレクトリ名を間違えないように注意して下さい。
テンプレートを格納するディレクトリ名はコントローラの先頭名と合わせる必要があります。

テンプレートを作成する前にPhoenixフレームワークにおけるテンプレートいうものが何なのか簡単に予習をしましょう。
PhoenixフレームワークではデフォルトでEExというテンプレートが使えます。
Ruby on Railsで使えるERBのようなものと思っておけば大丈夫です。(実際に非常によく似ています)
今回作るのも、そのEExのテンプレートになります。
色々と書いているけど、じゃあ一体どういうものなの？百聞は一見にしかずですね。実際に見てみましょう。

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

拡張子は.eexですが、見た目はただのHTMLでは？
そうですね。今の段階で作成するものはEExの要素はありません。
あれだけ書いておいて申し訳ないです。
しかし安心してください！EExの要素については、この章の最後の方で使います。
何事も順序が大事です。(言い訳)

### 実行

サーバを起動し、作ったページを確認してみましょう。

#### Example:

```cmd
$ mix phoenix.server
<C-c>
```

#### URL: http://localhost:4000/home

#### URL: http://localhost:4000/help

ページの確認ができました。まだ何も動的なWebページらしい要素がない静的ページです。
手動でファイルを作成していくのはなかなか大変でしたか？
大変だと感じたならそれは良いことです。手間を省きたいと考えたならなおのこと良いです！
プログラマにとって怠惰は美徳です。
それはさておき、ここまでやったことがPhoenixフレームワークにおけるページ追加の基本手順になります。
是非、覚えておいてください。きっと役に立つと思います。

## テストを交える

作ったページのテストを作成してみましょう。

### まずはテスト書いてみる

何はともあれテストを書いてみるとしましょう。
本来であれば機能の実装前にテストを作った方が良いのですが、
ちゃんとしたテストサイクルを回すのは新しくページを追加するときから始めます。
まずはExUnitをお試しで使ってみたいと思います。

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

テストが書けたら実行して確認しましょう。

#### Example:

```cmd
$ mix test
......

Finished in 0.1 seconds
6 tests, 0 failures

Randomized with seed 560404
```

問題なさそうですね。失敗がなしで無事成功しています。テストをするときはいつもどきどきします。
(6個中4個のテストはPhoenixフレームワークがデフォルトで生成しているテストになります)

### テスト駆動開発(TDD)について

さて実際にテストから書き始める前に簡単にテストサイクルと原則について説明しておきましょう。
本内容はテストについて学ぶためのものではありませんので最小限の内容になりますが。

テスト駆動開発(TDD)とはプログラム開発手法の一種です。
もう少し詳しく書くと、プログラムの実装に際して最初に失敗するテストを書き、
次にテストが成功する必要最低限の実装を行います。
実装後、成功を維持しながらリファクタリングを行いコードを洗練させる短い工程を繰り返し行っていくものになります。
最初にテストを書くからテストファーストと言うわけですね。
(他にもテスト駆動開発から派生したビヘイビア駆動開発(BDD)というものもあります)

上記のテストサイクルをまとめたものは下記のとおりです。

- (1) RED: 失敗するテストを書く
- (2) GREEN: テストを成功させる
- (3) REFACTOR: コードをリファクタリングして綺麗に保つ

テストサイクルを素早くテンポよく繰り返していくことが望ましいです。
また、テスト駆動開発を行うときの原則として、我らがUncle Bobより3つことが言われています。

- 失敗するユニットテストを成功させるためにしかプロダクトコードを書いてはならない。
- 失敗させるためにしかユニットテストを書いてはならない。コンパイルエラーは失敗に数える。
- ユニットテストを1つだけ成功させる以上にプロダクトコードを書いてはならない。

なぜ失敗するのをわかっているコードを書くの？最初から成功させれば良いのでは？など色々と思うことがあるかもしれません。
それらにも理由があります。しかし、上記すべてのことに理由を書いていては、
その内容だけで本チュートリアルが人を打倒できる厚さになってしまいますので大変遺憾ですが、割愛をさせていただきます。
さらにテストについて知りたいという方がいればテストに関しての専門的な本で学ぶことをおすすめします。
最低限と言いながら長くなっていまいましたが、テストサイクルと原則についての説明はこれで終わりにしたいと思います。

### レッド

それでは新しいページを追加しながらテストを行っていきましょう。
テストサイクルに従い、まずは失敗するテストを書きます。

#### File: test/controllers/static_page_controller_test.exs

```elixir
defmodule SampleApp.StaticPageControllerTest do
  ...

  test "GET /about", %{conn: conn} do
    conn = get conn, "/about"
    assert html_response(conn, 200) =~ "Welcome to Static Pages About!"
  end
end
```

テストが書けたらテストを実行して失敗することを確認します。

```cmd
$ mix test
....

  1) test GET /about (SampleApp.StaticPageControllerTest)
     test/controllers/static_page_controller_test.exs:14
     ** (RuntimeError) expected response with status 200, got: 404, with body:
     Page not found
     stacktrace:
       ...   

..

Finished in 0.2 seconds
7 tests, 1 failure

Randomized with seed 745020
```

ちゃんと(？)失敗しましたね。
どうやらルーティング先がないため失敗になっているようですね。

### グリーン

テストが失敗から何の要因によって止まっているのか明白です。
ただの色としてのレッドは嫌いではありませんが、テストしてはやはりグリーンがみたいものです。
テストを成功させるための最小限のコード実装しましょう。
ルーティングに/aboutのルーティングを追加します。

#### File: web/router.ex

```elixir
defmodule SampleApp.Router do
  ...

  scope "/", SampleApp do
    pipe_through :browser # Use the default browser stack

    get "/", PageController, :index
    get "/home", StaticPageController, :home
    get "/help", StaticPageController, :help
    get "/about", StaticPageController, :about
  end

  ...
end
```

```cmd
$ mix test
.....

  1) test GET /about (SampleApp.StaticPageControllerTest)
     test/controllers/static_page_controller_test.exs:14
     ** (UndefinedFunctionError) function SampleApp.StaticPageController.about/2 is undefined or private
     stacktrace:
       ...

.

Finished in 0.1 seconds
7 tests, 1 failure

Randomized with seed 565233
```

次はアクションの定義がある(はずの)コントローラでabout/2が定義されていないかプライベートになっているとあります。
アクションの定義は実装していませんのでさっそく追加しましょう。
きっと次は大丈夫ですよ！(フラグ)

#### File: web/controllers/static_page_controller.ex

```elixir
defmodule SampleApp.StaticPageController do
  ...

  def about(conn, _params) do
    render conn, "about.html"
  end
end
```

```cmd
$ mix test
....

  1) test GET /about (SampleApp.StaticPageControllerTest)
     test/controllers/static_page_controller_test.exs:14
     ** (Phoenix.Template.UndefinedError) Could not render "about.html" for SampleApp.StaticPageView,
     please define a matching clause for render/2 or define a template at "web/templates/static_page".
     The following templates were compiled:
     
     * help.html
     * home.html
     
     Assigns:
     
     %{conn: %Plug.Conn{...}
     
     stacktrace:
       ...

..

Finished in 0.1 seconds
7 tests, 1 failure

Randomized with seed 1816
```

Oops!テストが失敗してしまいました。
しかし、焦る必要はありません。我らがUncle Bobもいっていました。
追い詰められているときでも規律を厳守し自分のペースを保つのだと！
淡々と素早くテンポよくこなしていくとしましょう。

失敗の内容が書かれています。ちゃんと読みましょう。
テンプレートがないと書かれていますね。

#### web/templates/static_page/about.html.eex

```html
<div class="jumbotron">
  <h2>Welcome to Static Pages About!</h2>
</div>
```

```cmd
$ mix test
.......

Finished in 0.1 seconds
7 tests, 0 failures

Randomized with seed 34167
```

テスト成功！マーベラス！！

### リファクタリング

残念なことに今回実装した内容は、闇のコードでも強大な力を持った機能でもありません。
(Phoenixフレームワークには非常に強力な力がありますが)
そのため、ここでは特にリファクタリングをすることがありません。
しかし本来であればコードから漂う悪臭を嗅ぎ取り、その悪臭を消さなければいけません。
生ゴミより綺麗な食事のほうが誰だって嬉しいものです。

何にせよ、リファクタリングを実施しコードをクリーンに保つということを忘れないでください。
これは独立しているものではなくセットで存在しているものです。

## 少しだけ動的なページ

少しだけ動的に動くページに改造してみましょう。
ここまでに作成した3つのテンプレートでは、あるワードを除き同じですね。

```html
<h2>Welcome to Static Pages Home!</h2>

<h2>Welcome to Static Pages Help!</h2>

<h2>Welcome to Static Pages About!</h2>
```

この異なる部分を動的に指定できるようにしてみましょう。
StaticPagesコントローラの各アクション関数を下記のように変更して下さい。

#### File: web/controllers/static_page_controller.ex

```elixir
defmodule SampleApp.StaticPageController do
  use SampleApp.Web, :controller

  def home(conn, _params) do
    render conn, "home.html", message: "Home"
  end

  def help(conn, _params) do
    render conn, "help.html", message: "Help"
  end

  def about(conn, _params) do
    render conn, "about.html", message: "About"
  end
end
```

上記ではレンダリングするテンプレートへ変数を送っています。
例えば、home/2を例に取るとhome.html(EEx)へレンダリングするときに、
:messageというパラメータ名で"Home"という値を設定しています。
まとめると下記になります。

- :message ・・・ テンプレート内でのパラメータ名
- "Home" ・・・ 値

実際に各テンプレートから上記のパラメータ名を参照し使ってみましょう。

#### File: web/templates/static_page/home.html.eex

```html
<div class="jumbotron">
  <h2>Welcome to Static Pages <%= @message %>!</h2>
</div>
```

#### File: web/templates/static_page/help.html.eex

```html
<div class="jumbotron">
  <h2>Welcome to Static Pages <%= @message %>!</h2>
</div>
```

#### File: web/templates/static_page/about.html.eex

```html
<div class="jumbotron">
  <h2>Welcome to Static Pages <%= @message %>!</h2>
</div>
```

各ページの表示が変わらないことを確認します。
せっかく作ったので、まずは同じ動作をしているかテストから確認してみましょう。

#### Example:

```cmd
$ mix test
.......

Finished in 0.1 seconds
7 tests, 0 failures

Randomized with seed 258003
```

テスト成功です。無事同じ動作をしていることが確認できました。
テストの恩恵としては微々たるものですが、これがテストをする恩恵の一つですね。

テストの結果が同一ということは同じ結果が戻ってきているということです。
ならばWebページ上でも同じ結果になっているはずですね。Webページの方も確認します。

#### Example:

```cmd
$ mix phoenix.server
```

少し遅くなりましたが、さきほど何をやっていたのか説明していきます。
テンプレート内では以下のように記述すると、コントローラやビュー側から送った値を参照することができます。

```html
<%= @param_name %>
```

もう少し踏み込んでみます。上記の記述はテンプレートに対してElixirコードの埋め込みを行っています。
変数以外にもif記述、for記述や関数の実行などを行うことができます。

## おわりに

今回はここまでとなります。
これでページを追加していく方法はばっちりですね！
最後にここまでのソースコードをマージしておわりとしましょう。

```cmd
$ git add -A
$ git commit -m "Finish static_pages."
$ git checkout master
$ git merge static_pages
```
