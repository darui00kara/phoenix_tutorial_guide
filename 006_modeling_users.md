# ユーザモデルの実装

ユーザモデルの実装を行っていきましょう。
少しWebプログラミングっぽくなってきました。

## ゴール

この章で学べること...

- Phoenixフレームワークでモデルを作成する方法
- Ectoを使ったマイグレーションの方法
- モデルのデータに対するバリデーションの方法
- Ectoの基本的な使い方

## 概要

## 準備

#### Example:

```cmd
$ git checkout -b modeling_users
```

## ユーザのデータモデル

- ユーザのデータモデル
  * モデル名: User
  * テーブル名: users
  * 生成カラム(カラム名:型): name:string, email:string
  * 自動生成カラム(カラム名:型): id:integer, inserted_at:timestamp, updated_at:timestamp
  * インデックス(対象カラム名): name, email

```txt
アスキーでの図
```

#### Example:

```cmd
$ mix phoenix.gen.model User users name:string email:string
* creating web/models/user.ex
* creating test/models/user_test.exs
* creating priv/repo/migrations/[timestamp]_create_user.exs

Remember to update your repository by running migrations:

    $ mix ecto.migrate

```

まだマイグレーションしちゃめ！

#### File: priv/repo/migrations/[timestamp]_create_user.exs 

```elixir
defmodule SampleApp.Repo.Migrations.CreateUser do
  use Ecto.Migration
  @disable_ddl_transaction true
 
  def change do
    create table(:users) do
      add :name, :string
      add :email, :string

      timestamps()
    end
    
    create unique_index :users, [:name], concurrently: true
    create unique_index :users, [:email], concurrently: true
  end
end
```

#### Example:

```cmd
$ mix ecto.migrate

00:47:27.897 [info]  == Running SampleApp.Repo.Migrations.CreateUser.change/0 forward

00:47:27.897 [info]  create table users

00:47:27.901 [info]  create index users_name_index

00:47:27.903 [info]  create index users_email_index

00:47:27.908 [info]  == Migrated in 0.0s
```

作られたテーブルを確認してみる。

#### Example:

```txt
                                      Table "public.users"
   Column    |            Type             |                     Modifiers                      
-------------+-----------------------------+----------------------------------------------------
 id          | integer                     | not null default nextval('users_id_seq'::regclass)
 name        | character varying(255)      | 
 email       | character varying(255)      | 
 inserted_at | timestamp without time zone | not null
 updated_at  | timestamp without time zone | not null
Indexes:
    "users_pkey" PRIMARY KEY, btree (id)
    "users_email_index" UNIQUE, btree (email)
    "users_name_index" UNIQUE, btree (name)
```

#### Example:

```elixir
create table(テーブル名 do
  add カラム名, 型
  add カラム名, 型
  ...

  timestamps
end
```

#### Example:

```elixir
@disable_ddl_transaction true
```

#### Example:

```elixir
create unique_index :users, [:name], concurrently: true
create unique_index :users, [:email], concurrently: true
```

マイグレーションについて、もっと詳しく知りたい方はエーフィーのアトリエ2を買いましょう。(堂々宣伝)

## ユーザ

#### File: web/router.ex

```elixir
defmodule SampleApp.Router do
  ...

  scope "/", SampleApp do
    ...

    get "/signup", UserController, :new
  end

  ...
end
```

#### File: web/controllers/user_controller.ex

```elixir
defmodule SampleApp.UserController do
  use SampleApp.Web, :controller

  def new(conn, _params) do
    render conn, "new.html"
  end
end
```

#### File: web/views/user_view.ex

```elixir
defmodule SampleApp.UserView do
  use SampleApp.Web, :view
end
```

#### Directory: web/templates/user

#### File: web/templates/user/new.html.eex

```html
<div class="jumbotron">
  <h1>Sign up</h1>
  <p>Find me in web/templates/user/new.html.eex</p>
</div>
```

#### File: web/templates/static_page/home.html.eex

```html
<div class="jumbotron">
  ...

  <%= link "Sign up now!", to: user_path(@conn, :new), class: "btn btn-large btn-primary" %>
</div>
```

## バリデーション

## パスワードカラムの追加

## パスワードの暗号化

他のライブラリを導入しパスワードの暗号化を行ってみましょう。

## バリデーションの追加

## おわりに

#### Example:

```cmd
$ git add -A
$ git commit -m "Finish modeling_users"
$ git checkout master
$ git merge modeling_users
```

## 参考



