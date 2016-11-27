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

#### File: lib/helpers/authentication.ex

```elixir
defmodule SampleApp.Authentication do
  alias SampleApp.Helpers.Encryption

  def authentication(user, password) do
    case user do
      nil -> false
        _ ->
          Encryption.check_password(password, user.password_digest)
    end
  end
end
```

## サインイン

#### File: lib/helpers/signin.ex

```elixir
defmodule SampleApp.Signin do
  import SampleApp.Authentication

  alias SampleApp.User

  def signin(user, password) do
    case authentication(user, password) do
      true -> {:ok, user}
         _ -> :error
    end
  end
end
```

#### File: web/controllers/session_controller.ex

```elixir
defmodule SampleApp.SessionController do
  use SampleApp.Web, :controller

  alias SampleApp.User

  ...

  def create(conn, %{"signin_params" => %{"email" => email, "password" => password}}) do
    case Repo.get_by(User, email: email) |> signin(password) do
      {:ok, user} ->
        conn
        |> put_flash(:info, "User signin is success!")
        |> redirect(to: static_page_path(conn, :home))
      :error ->
        conn
        |> put_flash(:error, "User signin is failed! email or password is incorrect.")
        |> redirect(to: session_path(conn, :new))
    end
  end

  ...
end
```

## サインインのフォーム

#### File: web/templates/session/signin.html.eex

```html
<%= form_for @conn, session_path(@conn, :create), [as: :signin_params], fn f -> %>
  <%= if f.errors != [] do %>
    <div class="alert alert-danger">
      <p>Oops, something went wrong! Please check the errors below.</p>
    </div>
  <% end %>

  <div class="form-group">
    <%= label f, :email, class: "control-label" %>
    <%= text_input f, :email, class: "form-control" %>
    <%= error_tag f, :email %>
  </div>

  <div class="form-group">
    <%= label f, :password, class: "control-label" %>
    <%= text_input f, :password, class: "form-control" %>
    <%= error_tag f, :password %>
  </div>

  <div class="form-group">
    <%= submit "Signin", class: "btn btn-primary" %>
  </div>
<% end %>
```

## サインインのリンク

#### File: web/templates/layout/app.html.eex

```html
<header class="header navbar navbar-inverse">
  <div class="navbar-inner">
    <div class="container">
      <a class="logo" href="<%= page_path(@conn, :index) %>"></a>
      <nav role="navigation">
        <ul class="nav nav-pills pull-right">
          <li><%= link "Home", to: static_page_path(@conn, :home) %></li>
          <li><%= link "Help", to: static_page_path(@conn, :help) %></li>
          <li><%= link "Signin", to: session_path(@conn, :new) %></li>
        </ul>
      </nav>
    </div> <!-- container -->
  </div> <!-- navbar-inner -->
</header>
```

## セッションはどのように？

#### File: config/config.exs

```elixir
...

# Configures the endpoint
config :sample_app, SampleApp.Endpoint,
  url: [host: "localhost"],
  secret_key_base: "****",
  render_errors: [view: SampleApp.ErrorView, accepts: ~w(html json)],
  pubsub: [name: SampleApp.PubSub,
           adapter: Phoenix.PubSub.PG2]

...
```

#### File: lib/[app_name]/endpoint.ex

```elixir
defmodule SampleApp.Endpoint do
  ...

  # The session will be stored in the cookie and signed,
  # this means its contents can be read but not tampered with.
  # Set :encryption_salt if you would also like to encrypt it.
  plug Plug.Session,
    store: :cookie,
    key: "_sample_app_key",
    signing_salt: "nMpjG932"

  ...
end
```

#### Example:

```elixir
defmodule HelloPhoenix.SessionExampleController do
  use HelloPhoenix.Web, :controller

  def session_example(conn, _params) do
    conn = put_session(conn, :message, "Hello world!")
    message = get_session(conn, :message)

    text conn, message
  end
end
```

## セッションを使う

#### File: web/controllers/session_controller.ex

```elixir
defmodule SampleApp.SessionController do
  ...

  def create(conn, %{"signin_params" => %{"email" => email, "password" => password}}) do
    case Repo.get_by(User, email: email) |> signin(password) do
      {:ok, user} ->
        conn
        |> put_flash(:info, "User signin is success!")
        |> put_session(:user_id, user.id)
        |> redirect(to: static_page_path(conn, :home))
      :error ->
        ...
    end
  end

  ...
end
```

## サインイン状態の継続

#### Directory: lib/plugs

#### File: lib/plugs/check_authentication.ex

```elixir
defmodule SampleApp.Plugs.CheckAuthentication do
  import Plug.Conn

  alias SampleApp.{Repo, User}

  def init(options) do
    options
  end

  def call(conn, _) do
    case user_id = get_session(conn, :user_id) do
      nil ->
        conn
      _ ->
        conn
        |> assign(:current_user, Repo.get(User, user_id))
    end
  end
end
```

#### File: web/web.ex

```elixir
defmodule SampleApp.Web do
  ...

  def controller do
    quote do
      ...

      plug SampleApp.Plugs.CheckAuthentication
    end
  end

  ...
end
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
defmodule SampleApp.Helpers.ViewHelper do
  def current_user(conn) do
    conn.assigns[:current_user]
  end
end
```

#### File: web/web.ex

```elixir
defmodule SampleApp.Web do
  ...

  def view do
    quote do
      ...
      import SampleApp.Helpers.ViewHelper
    end
  end

  ...
end
```

#### File: web/temmplates/layout/debug.html.eex

```html
<div class="debug_dump">
  <p>Controller: <%= get_controller_name @conn %></p>
  <p>Action: <%= get_action_name @conn %></p>
  <%= if current_user(@conn) do %>
    <p>User (ID): <%= current_user(@conn).name %> (<%= current_user(@conn).id %>)</p>
  <% end %>
</div>
```

## リンクとレイアウトを動的に変更する

#### Example:

```elixir
<%= if current_user(@conn) do %>
  ログインしている時の処理...
<% else %>
  ログインしていない時の処理...
<% end %>
```

#### File: web/temmplates/layout/header.html.eex

```html
<header class="header navbar navbar-inverse">
  <div class="navbar-inner">
    <div class="container">
      <a class="logo" href="<%= page_path(@conn, :index) %>"></a>
      <nav role="navigation">
        <ul class="nav nav-pills pull-right">
          <li><%= link "Home", to: static_page_path(@conn, :home) %></li>
          <%= if current_user(@conn) do %>
            <li class="dropdown">
              <!-- Dropdown Menu -->
              <a href="#" class="dropdown-toggle" id="account" data-toggle="dropdown">
                User Menu
                <span class="caret"></span>
              </a>
              <!-- Dropdown List -->
              <ul class="dropdown-menu" aria-labelledby="account">
                <li><%= link "Profile", to: user_path(@conn, :show, current_user(@conn)) %><li>
                <li><%= link "Help", to: static_page_path(@conn, :help) %></li>
                <li class="divider"></li>
              </ul>
              <li><%= link "Signout", to: session_path(@conn, :delete), method: :delete %></li>
            </li>
          <% else %>
            <li><%= link "Signin", to: session_path(@conn, :new) %></li>
          <% end %>
        </ul>
      </nav>
    </div> <!-- container -->
  </div> <!-- navbar-inner -->
</header>
```

#### ~~File: web/static/css/custom/_header.scss~~

```css
いらなくなるかも・・・

/* header */

...

.dropdown-delete-link {
  color: #000000;
  margin-left: 20px;
}
.dropdown-delete-li {
  color: #000000;
  &:hover {
    background-color: #f5f5f5;
  }
}
```

## サインアウト

#### File: web/controllers/session_controller.ex

```elixir
defmodule SampleApp.SessionController do
  ...

  alias SampleApp.User

  ...

  def delete(conn, _params) do
    conn
    |> put_flash(:info, "Signout now! See you again!!")
    |> delete_session(:user_id)
    |> redirect(to: static_page_path(conn, :home))
  end
end
```

## サインアップ後のサインイン

#### File: web/controllers/user_controller.ex

```elixir
defmodule SampleApp.UserController do
  ...

  def create(conn, %{"user" => user_params}) do
    changeset = User.changeset(%User{}, user_params)

    case Repo.insert(changeset) do
      {:ok, user} ->
        conn
        |> put_flash(:info, "User created successfully.")
        |> put_session(:user_id, user.id)
        |> redirect(to: static_page_path(conn, :home))
      {:error, changeset} ->
        render(conn, "new.html", changeset: changeset)
    end
  end
end
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



