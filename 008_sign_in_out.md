# サインインとサインアウト

## ゴール

この章で学べること...

- サインイン/サインアウトの方法
- Phoenixフレームワークでのセッションの取り扱い方法
- Function Plug/Module Plugの作り方、使い方

## 概要

## 準備

#### Example:

```cmd
$ git checkout -b sign_in_out
```

## セッションコントローラの作成

#### File: web/router.ex

```elixir
defmodule SampleApp.Router do
  ...

  scope "/", SampleApp do
    pipe_through :browser # Use the default browser stack

    ...
    get "/signin", SessionController, :new
    post "/session", SessionController, :create
    delete "/signout", SessionController, :delete
  end

  ...
end
```

#### Example:

```cmd
$ mix phoenix.routes
...
    session_path  GET     /signin         SampleApp.SessionController :new
    session_path  POST    /session        SampleApp.SessionController :create
    session_path  DELETE  /signout        SampleApp.SessionController :delete
```

#### File: web/controllers/session_controller.ex

```elixir
defmodule SampleApp.SessionController do
  use SampleApp.Web, :controller

  def new(conn, _params) do
    render conn, "signin_form.html"
  end

  def create(conn, _params) do
    redirect(conn, to: static_page_path(conn, :home))
  end

  def delete(conn, _params) do
    redirect(conn, to: static_page_path(conn, :home))
  end
end
```

#### File: web/views/session_view.ex

```elixir
defmodule SampleApp.SessionView do
  use SampleApp.Web, :view
end
```

#### Directory: web/templates/session

#### File: web/templates/session/signin_form.html.eex

```html
<div class="jumbotron">
  <h2>Sign in!!</h2>
</div>
```

## 認証

#### File: web/models/user.ex

```elixir

```

#### File: lib/helpers/authentication.ex

```elixir

```

## サインイン

#### File: lib/helpers/signin.ex

```elixir

```

#### File: web/controllers/session_controller.ex

```elixir

```

## サインインのフォーム

#### File: web/templates/session/signin.html.eex

```html

```

## サインインのリンク

#### File: web/templates/layout/app.html.eex

```html

```

## セッションはどのように？

#### File: config/config.ex

```elixir

```

#### File: lib/[app_name]/endpoint.ex

```elixir

```

#### Example:

```elixir

```

## セッションを使う

#### File: web/controllers/session_controller.ex

```elixir

```

## サインイン状態の継続

#### Directory: lib/plugs

#### File: lib/plugs/check_authentication.ex

```elixir

```

#### File: web/web.ex

```elixir

```

#### Note:

```txt
Phoenixフレームワークの方で"secret_key_base"を設定しているためCookieは暗号化も署名もされています。
そのため、鍵が漏れない限りは改ざんすることは難しいです。(機能的にはPlugに分類される)

しかし、それでも心配だと考える方はいらっしゃると思います。
その場合は、Cookieではない別のストア(redis、memcachedなど)の利用を考えた方がよいと思います。
```

## 現在のユーザは？

#### File: lib/helpers/view_helper.ex

```elixir

```

#### File: web/web.ex

```elixir

```

#### File: web/temmplates/layout/debug.html.eex

```html

```

## リンクとレイアウトを動的に変更する

#### Example:

```elixir

```

#### File: web/temmplates/layout/header.html.eex

```html

```

#### File: web/static/css/custom/_header.scss

```css

```

## サインアウト

#### File: web/controllers/session_controller.ex

```elixir

```

## サインアップ後のサインイン

#### File: web/controllers/user_controller.ex

```elixir

```

## おわりに

#### Example:

```cmd
$ git add -A
$ git commit -m "Finish sign_in_out"
$ git checkout master
$ git merge sign_in_out
```

## 参考



