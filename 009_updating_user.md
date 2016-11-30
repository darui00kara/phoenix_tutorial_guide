# 第?章 xxx

## ゴール

この章で学べること...

- データの更新方法
- 共有テンプレートの使い方
- ページネーションのやり方
- データの削除方法

## 概要

## 準備

#### Example:

```cmd
$ git checkout -b updating_user
```

## プロフィールの編集

### Editアクション

#### File: web/controllers/user_controller.ex

```elixir
defmodule SampleApp.UserController do
  ...

  def edit(conn, %{"id" => id}) do
    user = Repo.get(User, id)
    changeset = User.changeset(user)
    render conn, "edit.html", user: user, changeset: changeset
  end
end
```

### Editテンプレート

#### File: web/templates/user/edit.html.eex

```elixir
<%= form_for @changeset, user_path(@conn, :update, @user), fn f -> %>
  <%= if @changeset.action do %>
    <div class="alert alert-danger">
      <p>Oops, something went wrong! Please check the errors below.</p>
    </div>
  <% end %>

  <div class="form-group">
    <%= label f, :name, class: "control-label" %>
    <%= text_input f, :name, class: "form-control" %>
    <%= error_tag f, :name %>
  </div>

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
    <%= submit "Update", class: "btn btn-primary" %>
  </div>
<% end %>
```

### Editのリンク

#### File: web/templates/user/show.html.eex

```html
<div class="row">
  <aside class="col-md-4">
    <section>
      <h1>
        <img src="<%= gravatar_for(@user) %>" class="gravatar">
        <%= @user.name %>
      </h1>
    </section>
    <section>
      <%= link "Edit", to: user_path(@conn, :edit, @user), class: "btn btn-default btn-xs" %>
    </section>
  </aside>
</div>
```

## プロフィールの更新

### Updateアクション

#### File: web/controllers/user_controller.ex

```elixir
defmodule SampleApp.UserController do
  ...

  def update(conn, %{"id" => id, "user" => user_params}) do
    user = Repo.get(User, id)
    changeset = User.changeset(user, user_params)

    case Repo.update(changeset) do
      {:ok, _} ->
        conn
        |> put_flash(:info, "User updated successfully.")
        |> redirect(to: user_path(conn, :show, user.id))
      {:error, failed_changeset} ->
        render(conn, "edit.html", user: user, changeset: failed_changeset)
    end
  end
end
```

## 共有テンプレート

### 共有テンプレート

#### File: web/templates/user/form.html.eex

```elixir
<%= form_for @changeset, @action, fn f -> %>
  <%= if @changeset.action do %>
    <div class="alert alert-danger">
      <p>Oops, something went wrong! Please check the errors below.</p>
    </div>
  <% end %>

  <div class="form-group">
    <%= label f, :name, class: "control-label" %>
    <%= text_input f, :name, class: "form-control" %>
    <%= error_tag f, :name %>
  </div>

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
    <%= submit @submit, class: "btn btn-primary" %>
  </div>
<% end %>
```

#### File: web/templates/user/new.html.eex

```html
<h1>Sign up</h1>

<%= render SampleApp.UserView, "form.html", changeset: @changeset,
                                            action: user_path(@conn, :create ),
                                            submit: "Signup" %>
```

#### File: web/templates/user/new.html.eex

```html
<h1>Edit Profile</h1>

<%= render SampleApp.UserView, "form.html", changeset: @changeset,
                                            action: user_path(@conn, :update, @user),
                                            submit: "Update" %>
```

## 認証と認可

### 認証と認可の違い

### そのユーザサインインしてますか？

#### File: lib/plugs/signed_in_user.ex

```elixir
defmodule SampleApp.Plugs.SignedInUser do
  import Plug.Conn
  import Phoenix.Controller, only: [put_flash: 3, redirect: 2]
  import SampleApp.Router.Helpers, only: [session_path: 2]

  def init(options) do
    options
  end

  def call(conn, _) do
    if conn.assigns[:current_user] do
      conn
    else
      conn
      |> put_flash(:info, "Please signin.")
      |> redirect(to: session_path(conn, :new))
      |> halt
    end
  end
end
```

#### File: web/controllers/user_controller.ex

```elixir
defmodule SampleApp.UserController do
  use SampleApp.Web, :controller

  alias SampleApp.User

  plug SampleApp.Plugs.SignedInUser

  ...
end
```

#### File: web/controllers/user_controller.ex

```elixir
defmodule SampleApp.UserController do
  use SampleApp.Web, :controller

  alias SampleApp.User

  plug SampleApp.Plugs.SignedInUser when action in [:show, :edit, :update]

  ...
end
```

### あなたは正しい私ですか？

#### File: web/controllers/user_controller.ex

```elixir
defmodule SampleApp.UserController do
  use SampleApp.Web, :controller

  alias SampleApp.User

  plug SampleApp.Plugs.SignedInUser when action in [:show, :edit, :update]
  plug :correct_user? when action in [:edit, :update]

  ...

  defp correct_user?(conn, _) do
    user = Repo.get(User, String.to_integer(conn.params["id"]))

    if current_user?(conn, user) do
      conn
    else
      conn
      |> put_flash(:info, "Please signin.")
      |> redirect(to: session_path(conn, :new))
      |> halt
    end
  end

  defp current_user?(conn, user) do
    conn.assigns[:current_user] == user
  end
end
```

## 全てのユーザ

### Indexアクション

#### File: web/controllers/user_controller.ex

```elixir
defmodule SampleApp.UserController do
  ...

  plug SampleApp.Plugs.SignedInUser when action in [:show, :edit, :update, :index]

  ...

  def index(conn, _params) do
    users = Repo.all(User)
    render(conn, "index.html", users: users)
  end

  ...
end
```

### Indexテンプレート

#### File: web/templates/user/index.html.eex

```html
<h1>All users</h1>

<%= if !is_empty_list?(@users) do %>
  <ul class="users">
    <%= for user <- @users do %>
      <%= render "user.html", conn: @conn, user: user %>
    <% end %>
  </ul>
<% end %>
```

#### File: web/templates/user/user.html.eex

```html
<li>
  <img src="<%= gravatar_for(@user) %>" class="gravatar">
  <%= link @user.name, to: user_path(@conn, :show, @user) %>
</li>
```

#### File: web/views/user_view.ex

```elixir
defmodule SampleApp.UserView do
  ...

  def is_empty_list?(list) when is_list(list) do
    list == []
  end
end
```

#### File: web/static/css/custom/_user.scss

```css
/* Users index */

.users {
  list-style: none;
  margin: 0;
  li {
    overflow: auto;
    padding: 10px 0;
    border-top: 1px solid #eeeeee;
    &:last-child {
      border-bottom: 1px solid #eeeeee;
    }
  }
}
```

#### File: web/static/css/custom/custom.scss

```css
/* custom main scss */

...
@import "user";
```

### 一覧へのリンク

#### File: web/templates/layout/header.html.eex

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
                <li><%= link "All Users", to: user_path(@conn, :index) %><li>
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

## ページネーション

### ページネーションの準備

#### File: mix.exs

```elixir
defmodule SampleApp.Mixfile do
  ...

  # Configuration for the OTP application.
  #
  # Type `mix help compile.app` for more information.
  def application do
    [mod: {SampleApp, []},
     applications: [... :scrivener, :scrivener_ecto, :scrivener_html]]
  end

  ...

  # Specifies your project dependencies.
  #
  # Type `mix help deps` for examples and options.
  defp deps do
    [...
     {:scrivener, "~> 2.1.1"},
     {:scrivener_ecto, "~> 1.0.3"},
     {:scrivener_html, "~> 1.3.3"}]
  end

  ...
end
```

#### Example:

```cmd
$ mix deps.get
```

#### File: config/config.exs

```elixir
use Mix.Config

...

# Configures Scrivener
config :scrivener_html,
  routes_helper: SampleApp.Router.Helpers

import_config "#{Mix.env}.exs"
```

#### File: lib/sample_app/repo.ex

```elixir
defmodule SampleApp.Repo do
  ...
  use Scrivener, page_size: 30
end
```

### Indexアクションにページネーションを追加

#### File: web/controllers/user_controller.ex

```elixir
defmodule SampleApp.UserController do
  ...

  def index(conn, params) do
    users = from(u in User, order_by: [asc: :name])
            |> Repo.paginate(params)

    if users do
      render(conn, "index.html", users: users)
    else
      conn
      |> put_flash(:error, "Invalid page number!!")
      |> render("index.html", users: [])
    end
  end

  ...
end
```

### ページネーションのビューとテンレプート

#### File: web/views/pagination_view.ex

```elixir
defmodule SampleApp.PaginationView do
  use SampleApp.Web, :view

  import Scrivener.HTML
end
```

#### File: web/templates/pagination/pagination.html.eex

```html
<%= pagination_links @conn, @pages, view_style: :bootstrap %>
```

#### File: web/templates/user/index.html.eex

```html
<h1>All users</h1>

<%= if !is_empty_list?(@users.entries) do %>
  <%= render SampleApp.PaginationView, "pagination.html", conn: @conn, pages: @users %>
  <ul class="users">
    <%= for user <- @users.entries do %>
      <%= render "user.html", conn: @conn, user: user %>
    <% end %>
  </ul>
  <%= render SampleApp.PaginationView, "pagination.html", conn: @conn, pages: @users %>
<% end %>
```

## ユーザの削除

### Deleteアクション

### Deleteのリンク

## おわりに

#### Example:

```cmd
$ git add -A
$ git commit -m "Finish updating_user"
$ git checkout master
$ git merge updating_user
```

## 参考



