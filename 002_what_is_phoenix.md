# ゴール

この章で学べること...

- Phoenixフレームワークの基礎知識を身につける
- Phoenixフレームワークをインストールする
- ディレクトリ構成について把握する
- プロジェクトの作り方、サーバの実行方法を実践する

# コンテキスト

この章では、Phoenixフレームワークの概要と構成について学んでいきます。

## Phoenix is 何？

Phoenixフレームワークとは何なのか・・・何なんでしょうね？
それを把握するために、Phoenixフレームワークの概要や特徴、構成要素について見ていきましょう。

### 概要

Phoenixフレームワークについて説明をする前にElixir言語について説明をしなければいけないでしょう。
Elixir言語は、ErlangVM上で動作する関数型言語です。
ErlangVMを利用しているため、Erlang/OTPを利用することができます。
そして、Rubyライクに記述することができる言語です。
つまり、ErlangとRubyの良いとこ取りをした言語です。

#### Note: Erlang/OTPはエリクソンで開発された関数型プログラミング言語です。

PhoenixフレームワークはElixir言語で実装されたWebフレームワークです。
サーバをMVCパターンに沿って構築していくことができます。
その構成要素や概念の多くは、Ruby on RailsやDjangoのような他のWebフレームワークに影響を受けています。
それらのフレームワーク経験があれば、使うのにさほど苦労をすることはないと思います。
高い生産性とアプリケーションパフォーマンスの両方の長所を提供しているお得なフレームワークです。
また、リアルタイム性を提供する機能がデフォルトの構成に入っています。
もちろん、その機能を利用しないで一般的なWebサイトを構築することもできます。
これは個人的な所見になりますが、フロントエンドというよりはバックエンドでの動作を期待します。

### 特徴

- RoRライク
- MVCモデル
- ジェネレータ(Mix Tasks)
- ルーティング
- テンプレート
- Ecto(データベース用ドメイン固有言語)
- WebSocket

### 構成要素
  
- Endpoint
- Router
- Controllers
- Views
- Templates
- Channels(WebSocket)
- PubSub

構成要素の名前だけを提示されてもわかりませんね。
いつかどこかで役に立つかもしれませんので、Phoenixの構造や作りを見てみましょう。
Phoenixのサーバサイドにおけるリクエストのライフサイクルを意識しなくても実装をしていくことができますが、この情報が頭の中にあることは有益です。少なくとも害になることはないでしょう。

- 概略図

<div class="separator" style="clear: both; text-align: center;">
<a href="https://2.bp.blogspot.com/-H9jckX4mwa8/WAs9H0CalAI/AAAAAAAAAbk/-7glg8fCRPMMkaEuRRCCK5HvhoAK1iONQCLcB/s1600/phoenix_architecture_001.jpg" imageanchor="1" style="margin-left: 1em; margin-right: 1em;"><img border="0" height="238" src="https://2.bp.blogspot.com/-H9jckX4mwa8/WAs9H0CalAI/AAAAAAAAAbk/-7glg8fCRPMMkaEuRRCCK5HvhoAK1iONQCLcB/s320/phoenix_architecture_001.jpg" width="320" /></a></div>
<br />

#### Note: CowboyはErlang/OTPで実装されているHTTPサーバです。
#### Note: PlugはElixirで実装されたWebサーバのためのアダプタを提供しています。
#### Note: EctoはElixirで実装されたデータベースラッパでDSLです。

- リクエストのライフサイクル

<div class="separator" style="clear: both; text-align: center;">
<a href="https://2.bp.blogspot.com/-DOuZ7vOYEU4/WAs9J0BJd3I/AAAAAAAAAbo/JICJe17EQQkXqPT6fy1BRb_faFnU7K82ACLcB/s1600/phoenix_architecture_002.jpg" imageanchor="1" style="margin-left: 1em; margin-right: 1em;"><img border="0" height="240" src="https://2.bp.blogspot.com/-DOuZ7vOYEU4/WAs9J0BJd3I/AAAAAAAAAbo/JICJe17EQQkXqPT6fy1BRb_faFnU7K82ACLcB/s320/phoenix_architecture_002.jpg" width="320" /></a></div>
<br />

Phoenixの構造や作りは、なんとなく分かりましたか？
実際に実装をするようになったら意識してみてください。
(なお、Channelに関しては本チュートリアルでは扱っていませんのでご注意を)

#### Info:  導入実績

外部リンク: [voluntas/japanese-erlang-elixir-companies](https://github.com/voluntas/japanese-erlang-elixir-companies/blob/master/README.md)

## インストール

Phoenixフレームワークをインストールし、プロジェクトの生成まで行っていきます。

### アーカイブ・インストール

Phoenixのインストールはmixコマンドから行うことができます。
下記を実行してください。

#### Example:

```elixir
$ mix archive.install https://github.com/phoenixframework/archives/raw/master/phoenix_new.ez
```

簡単ですね。こんな簡単にインストールできてしまいます。

### プロジェクトの生成

オプションの指定はなしで、一番シンプルなプロジェクトを作ってみましょう。
さきほどのインストールでmixコマンドへPhoenixのプロジェクトを生成するコマンドが追加されています。
まずはそのコマンドを確認します。

#### Example:

```cmd
$ mix help | grep phoenix
mix local.phoenix     # Updates Phoenix locally
mix phoenix.new       # Creates a new Phoenix v1.2.1 application
```

二つコマンドが表示されています。
ひとつめは、Phoenixフレームワークをアップデートするときに使います。
新しいバージョンが出たら、このコマンド使ってアップデートしましょう。
ふたつめが目的のものです。こちらのコマンドを使ってプロジェクトの生成ができます。
さっそく、プロジェクトを作ってみましょう。

#### Example:

```cmd
$ mix phoenix.new demo_app
...

Fetch and install dependencies? [Yn] y
* running mix deps.get
* running npm install && node node_modules/brunch/bin/brunch build

We are all set! Run your Phoenix application:

    $ cd demo_app
    $ mix phoenix.server

You can also run your app inside IEx (Interactive Elixir) as:

    $ iex -S mix phoenix.server

Before moving on, configure your database in config/dev.exs and run:

    $ mix ecto.create

```

プロジェクトを生成すると、軽く怯むくらいディレクトリやファイルが作られています。
全てを把握するのは大変ですが、次の項目で詳しく見ていきましょう。

## ディレクトリ構成

プロジェクトを生成したときに作られている、各階層のディレクトリやファイルについて詳しく見ていきましょう。

### 各階層のディクレトリやファイル

Phoenixでプロジェクトを生成したときのディレクトリ構成は以下になります。
使っているうちに頭に入っているでしょうから、覚える必要はありません。
ここでは、こんな構成で作られているのかというのが把握できれば大丈夫です。

#### Example:

| 説明                                                      |                                  |                                                                   |
|-----------------------------------------------------------|----------------------------------|-------------------------------------------------------------------|
| .gitignore                                                |                                  | GitへCommitしないファイル&ディレクトリを記述するファイル          |
| README.md                                                 |                                  | 説明書                                                            |
| _build/                                                   |                                  | ビルドされたファイルが格納されるディレクトリ                      |
| brunch-config.js                                          |                                  | ブランチの構成、静的資産を管理するための設定用JavaScript          |
| config/                                                   |                                  | アプリケーションの設定ファイルが格納されているディレクトリ        |
|                                                           | config.exs                       | "共通の設定を記述するファイル                                     |
| 下記の設定ファイルを環境変数により                        |                                  |                                                                   |
| 動的に呼び出し各環境に合わせた設定を持たせることができる" |                                  |                                                                   |
|                                                           | dev.exs                          | 開発版の設定を記述するファイル                                    |
|                                                           | prod.exs                         | 製品版の設定を記述するファイル                                    |
|                                                           | prod.secret.exs                  | 製品版で公開しない設定を記述するファイル                          |
|                                                           | test.exs                         | テストで使う設定を記述するファイル                                |
| deps/                                                     |                                  | mixの依存関係(ライブラリ)が格納されるディレクトリ                 |
| lib/                                                      |                                  | アプリケーション内で共有する機能を格納するディレクトリ            |
|                                                           | [Application Name]/endpoint.ex   | "PhoenixにおけるWebアプリケーションへの                           |
| すべての要求を開始する境界となるファイル"                 |                                  |                                                                   |
|                                                           | [Application Name]/repo.ex       | データベース操作用のモジュール(実体はEcto)                        |
|                                                           | [Application Name].ex            | アプリケーションのスーパーバイザ                                  |
| mix.exs                                                   |                                  | アプリケーションで利用するmixの設定を記述するファイル             |
| mix.lock                                                  |                                  | 取得した依存関係のバージョンをロックするためのファイル            |
| node_modules/                                             |                                  | npmで取得したパッケージを格納しているディレクトリ                 |
| package.json                                              |                                  | npmの依存関係を記述するファイル                                   |
| priv/                                                     |                                  | "静的資産を格納するディレクトリ                                   |
| web/static配下からコピーされる                            |                                  |                                                                   |
| 公開したくない静的資産を格納する場合、                    |                                  |                                                                   |
| web/staticではなくこちらを利用するとよい"                 |                                  |                                                                   |
|                                                           | gettext/en/LC_MESSAGES/errors.po | 英語用のpoファイル                                                |
|                                                           | gettext/errors.pot               | Gettextの元となるファイル                                         |
|                                                           | repo/                            | データベース用のディレクトリ                                      |
|                                                           | repo/migrations                  | マイグレーションファイル格納ディレクトリ                          |
|                                                           | repo/seeds.exs                   | DBへテストデータを作成するためのファイル                          |
|                                                           | static/                          | "静的資産を格納するディレクトリ                                   |
| priv/static配下へコピーされる"                            |                                  |                                                                   |
|                                                           | static/css                       | cssファイルを格納するディレクトリ                                 |
|                                                           | static/css/app.css               | アプリケーションのスタイルシート                                  |
|                                                           | static/css/app.css.map           | app.cssのソースマップ                                             |
|                                                           | static/favicon.ico               | Phoenixのシンボルマーク・イメージ                                 |
|                                                           | static/images                    | 画像を格納するディレクトリ                                        |
|                                                           | static/images/phoenix.png        | Phoenixのロゴ                                                     |
|                                                           | static/js                        | JavaScriptファイルを格納するディレクトリ                          |
|                                                           | static/js/app.js                 | アプリケーションのJavaScript                                      |
|                                                           | static/js/app.js.map             | app.jsのソースマップ                                              |
|                                                           | static/robots.txt                | ロボットファイル                                                  |
| test/                                                     |                                  | アプリケーションのテストに使うファイルを格納するディレクトリ      |
|                                                           | channels/                        | チャネルのテストファイルを格納するディレクトリ                    |
|                                                           | controllers/                     | コントローラのテストファイルを格納するディレクトリ                |
|                                                           | madels/                          | モデルのテストファイルを格納するディレクトリ                      |
|                                                           | support/                         | テストのサポートファイルを格納するディレクトリ                    |
|                                                           | support/channel_case.ex          | チャネルを使ったテストをするときのサポートファイル                |
|                                                           | support/conn_case.ex             | Plug.Connを利用するときのサポートファイル                         |
|                                                           | support/model_case.ex            | モデルを使ったテストをするときのサポートファイル                  |
|                                                           | views/                           | ビューのテストファイルを格納するディレクトリ                      |
|                                                           | test_helper.exs                  | テストのヘルパー                                                  |
| web/                                                      |                                  | "アプリケーションを格納するディレクトリ                           |
| 主要なプログラムはこの配下に格納"                         |                                  |                                                                   |
|                                                           | channels/                        | チャネルのファイルを格納するディレクトリ                          |
|                                                           | channels/user_socket.ex          | デフォルトで生成されているソケットハンドラ                        |
|                                                           | controllers/                     | コントローラのファイルを格納するディレクトリ                      |
|                                                           | controllers/page_controller.ex   | デフォルトで生成されているWelcomeページ用のコントローラモジュール |
|                                                           | models/                          | モデルのファイルを格納するディレクトリ                            |
|                                                           | static/                          | 静的ファイルを格納するディレクトリ                                |
|                                                           | static/assets                    | 静的資産を格納するディレクトリ                                    |
|                                                           | static/assets/favicon.ico        | Phoenixのシンボルマーク・イメージ                                 |
|                                                           | static/assets/images             | 画像を格納するディレクトリ                                        |
|                                                           | static/assets/images/phoenix.png | Phoenixのロゴ                                                     |
|                                                           | static/assets/robots.txt         | ロボットファイル                                                  |
|                                                           | static/css                       | CSSを格納するディレクトリ                                         |
|                                                           | static/css/app.css               | アプリケーションのスタイルシートを記述するファイル                |
|                                                           | static/css/phoenix.css           | Phoenoxがデフォルトで用意しているスタイルシート(Bootstrap v3.3.5) |
|                                                           | static/js                        | JavaScriptを格納するディレクトリ                                  |
|                                                           | static/js/app.js                 | アプリケーションのJavaScriptを記述するファイル                    |
|                                                           | static/js/socket.js              | WebSocket用の内容が記述されたJavaScript                           |
|                                                           | static/vendor                    | ベンダーのライブラリなどを格納するディレクトリ                    |
|                                                           | templates/                       | テンプレートファイルを格納するディレクトリ                        |
|                                                           | templates/layout                 | アプリケーションのレイアウトを格納するディレクトリ                |
|                                                           | templates/layout/app.html.eex    | アプリケーションのレイアウトを記述するファイル                    |
|                                                           | templates/page                   | Welcomeページ用のテンプレートが格納されているディレクトリ         |
|                                                           | templates/page/index.html.eex    | Welcomeページ用のテンプレート                                     |
|                                                           | views/                           | ビューのファイルを格納するディレクトリ                            |
|                                                           | views/error_helpers.ex           | Web画面表示の際、エラーが発生したときのヘルパー                   |
|                                                           | views/error_view.ex              | Web画面表示の際、エラーが発生したときに使われるエラービュー       |
|                                                           | views/layout_view.ex             | アプリケーションレイアウトのビュー                                |
|                                                           | views/page_view.ex               | Welcomeページ用のビュー                                           |
|                                                           | gettext.ex                       | 他言語対応用のGettextファイル                                     |
|                                                           | router.ex                        | ルーティングを記述するファイル                                    |
|                                                           | web.ex                           | 各機能に展開する定義が記述されたファイル                          |

#### Note: 静的資産についてはbrunch-config.jsの設定が必要になることが多いので注意が必要。

### ファイル名の規約

Phoenixのコマンド(Mix Tasks)を使ってい生成されているファイル名について確認してみましょう。
大体の場合は、以下の規約に沿ってファイルやモジュールを作れば問題ないと思います。
何のファイルなのか分かりやすい状態にしておくと、後々幸せになれるかもしれません。
少なくとも不幸になることはないでしょう。

| ディレクトリ          | ファイル                   | モジュール        | useモジュール                           |
|-----------------------|----------------------------|-------------------|-----------------------------------------|
| web/contorollers/     | xxx_contoroller.ex         | XxxController     | use [Application Name].Web, :controller |
| web/models/           | xxx.ex                     | Xxx               | use [Application Name].Web, :model      |
| web/views/            | xxx_view.ex                | XxxView           | use [Application Name].Web, :view       |
| web/templates/xxx/    | yyy.html.eex               | なし              | なし                                    |
| web/channels/         | zzz_channel.ex             | ZzzChannel        | use [Application Name].Web, :channel    |
| priv/repo/migrations/ | [timestamp]_create_mmm.exs | CreateXxx         | use Ecto.Migration                      |
| test/controllers/     | xxx_controller_test.ex     | XxxControllerTest | use [Application Name].ConnCase         |
| test/models/          | xxx_test.ex                | XxxTest           | use [Application Name].ModelCase        |
| test/views/           | xxx_view_test.ex           | XxxViewTest       | use Practice.ConnCase, async: true      |
| test/channels/        | zzz_channel_test.ex        | ZzzChannelTest    | use [Application Name].ChannelCase      |
| test/support/         | なし                       | なし              | なし                                    |

xxx = コントローラ名、モデル名
yyy = アクション名
zzz = チャネル名
mmm = マイグレーション名

DBテーブル名 = xxxs (モデル名の複数系)

#### Note: マイグレーションファイルは操作名＋テーブル名などにすると分かりやすい。

## ようこそPhoenixへ！

最後にPhoenixのWelcomeページを確認して、この章は終わりにしたいと思います。

#### Example:

```cmd
>mix phoenix.server

# "Ctrl+C"で終了できます。
```

以下のアドレスへアクセスしてみましょう。

#### URL: http://localhost:4000/

ようこそPhoenixへ！！

#### Note: Phoenixフレームワークのデフォルトポート番号は4000です。

## おわりに

次の項目では実際にPhoenixフレームワークでデモ用のアプリケーションを作成していきます。



