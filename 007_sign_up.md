# サインアップ

## ゴール

この章で学べること...

- 一番簡単のユーザ認証のやり方
- Ectoを使ったデータのインサートと取得の方法
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
[SampleApp.Repo, SampleApp.User]
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

## サイドバー

## ユーザのサインアップ

## おわりに

#### Example:

```cmd
$ git add -A
$ git commit -m "Finish sign_up."
$ git checkout master
$ git merge sign_up
```

## 参考



