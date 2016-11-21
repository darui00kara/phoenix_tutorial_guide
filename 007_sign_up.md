# サインアップ

## ゴール

この章で学べること...

- サインアップ機能の作り方
- ユーザの表示機能の作り方
- Gravatar画像を使う方法

## 概要

## 準備

```cmd
$ git checkout -b sign_up
```

## ユーザの表示

#### File: web/router.ex

```elixir
defmodule SampleApp.Router do
  ...

  scope "/", SampleApp do
    pipe_through :browser # Use the default browser stack

    ...
    resources "/user", UserController, except: [:new]
  end

  ...
end
```

#### Example:

```cmd
$ mix phoenix.routes
...
       user_path  GET     /user           SampleApp.UserController :index
       user_path  GET     /user/:id/edit  SampleApp.UserController :edit
       user_path  GET     /user/:id       SampleApp.UserController :show
       user_path  POST    /user           SampleApp.UserController :create
       user_path  PATCH   /user/:id       SampleApp.UserController :update
                  PUT     /user/:id       SampleApp.UserController :update
       user_path  DELETE  /user/:id       SampleApp.UserController :delete
```

#### Example:

```cmd
get "/signup", UserController, :new

$ mix phoenix.routes
      user_path  GET     /signup         SampleApp.UserController :new
```

#### Note:

```txt
Phoenixフレームワークのresourcesで追加されるアクションは、  
new、index、edit、show、crate、update、deleteになります。  

この中のアクションであれば、オプションで指定できます。  
```

#### File: web/controllers/user_controller.ex

```elixir
defmodule SampleApp.UserController do
  use SampleApp.Web, :controller

  alias SampleApp.User

  ...

  def show(conn, %{"id" => id}) do
    user = Repo.get!(User, id)
    render(conn, "show.html", user: user)
  end
end
```

#### File: web/templates/user/show.html.eex

```html
<div class="jumbotron">
  <strong>Name:</strong><%= @user.name %>
  <strong>Email:</strong><%= @user.email %>
</div>
```

#### Example:

```elixir
$ iex -S mix

iex> alias SampleApp.{Repo, User}
iex> params = %{name: "hoge", email: "hoge@test.com", password: "hogehoge"}
iex> changeset = User.changeset(%User{}, params)
iex> changeset.valid?
iex> Repo.insert(changeset)
{:ok,
 %SampleApp.User{__meta__: #Ecto.Schema.Metadata<:loaded, "users">,
  email: "hoge@test.com", id: 1,
  inserted_at: #Ecto.DateTime<2016-11-20 06:57:40>, name: "hoge",
  password: "hogehoge",
  password_digest: "$2b$12$HHH.f.h5QyGl.efIyNMU7.i47oc6gCBOB69PZaec7QMqbFXPCo7dq",
  updated_at: #Ecto.DateTime<2016-11-20 06:57:40>}}
iex> Repo.get(User, 1)
%SampleApp.User{__meta__: #Ecto.Schema.Metadata<:loaded, "users">,
 email: "hoge@test.com", id: 1,
 inserted_at: #Ecto.DateTime<2016-11-20 06:57:40>, name: "hoge", password: nil,
 password_digest: "$2b$12$HHH.f.h5QyGl.efIyNMU7.i47oc6gCBOB69PZaec7QMqbFXPCo7dq",
 updated_at: #Ecto.DateTime<2016-11-20 06:57:40>}
```

インサート時にはパスワードの値があります。
インサート後、DBからデータを取得するとnilになっています。

seeds.exsを利用してもよい。

## Gravatar画像

#### File: lib/helpers/gravatar.ex

```elixir
defmodule SampleApp.Helpers.Gravatar do
  def get_gravatar_url(email, size) do
    gravatar_id = email_to_gravator_id(email)
    "https://secure.gravatar.com/avatar/#{gravatar_id}?s=#{size}"
  end

  defp email_to_gravator_id(email) do
    email |> email_downcase |> email_crypt_md5
  end

  defp email_crypt_md5(email) do
    :erlang.md5(email)
    |> :erlang.bitstring_to_list
    |> Enum.map(&(:io_lib.format("~2.16.0b", [&1])))
    |> List.flatten
    |> :erlang.list_to_bitstring
  end

  defp email_downcase(email) do
    String.downcase(email)
  end
end
```

#### File: web/views/user_view.ex

```elixir
defmodule SampleApp.UserView do
  use SampleApp.Web, :view

  alias SampleApp.User
  alias SampleApp.Helpers.Gravatar

  def gravatar_for(%User{email: email}) do
    Gravatar.get_gravatar_url(email, 50)
  end
end
```

#### File: web/templates/user/show.html.eex

```elixir
<div class="jumbotron">
  <img src="<%= gravatar_for(@user) %>" class="gravatar">
  <strong>Name:</strong><%= @user.name %>
  <strong>Email:</strong><%= @user.email %>
</div>
```

#### File: web/static/css/custom/_gravatar.scss

```css
/* gravatar */

.gravatar {
  float: left;
  margin-right: 10px;
}

.gravatar_edit {
  margin-top: 15px;
}
```

## サイドバー

#### File: web/templates/user/show.html.eex

```elixir

```

#### File: web/static/css/custom/_sidebar.scss

```css

```

## ユーザのサインアップ

#### File: web/templates/user/new.html.eex

```html

```

#### File: web/controllers/user_controller.ex

```elixir

```

#### web/model/user.ex

```elixir

```

#### File: web/controllers/user_controller.ex

```elixir

```

#### File: web/controllers/user_controller.ex

```elixir

```

#### File: web/templates/user/new.html.eex

```html

```

#### File: web/controllers/user_controller.ex

```elixir

```

#### Note:

```txt
phoenix_html, form_for/4
```

## おわりに

#### Example:

```cmd
$ git add -A
$ git commit -m "Finish sign_up."
$ git checkout master
$ git merge sign_up
```

## 参考



