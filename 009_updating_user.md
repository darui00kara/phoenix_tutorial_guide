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

#### File: lib/plugs/signed_in_user.ex

```elixir

```

#### File: web/controllers/user_controller.ex

```elixir

```

#### File: web/controllers/user_controller.ex

```elixir

```

### そのユーザサインインしてますか？

### あなたは私ですか？

## 全てのユーザ

### 一覧へのリンク

## ページネーション

### ページネーション

### ページネーションのビューとテンレプート

### ページネートできますか？

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



