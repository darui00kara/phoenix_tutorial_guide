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

#### File: web/models/user.ex

```elixir
defmodule SampleApp.User do
  ...

  def changeset(struct, params \\ %{}) do
    struct
    |> cast(params, [:name, :email])
    |> validate_required([:name, :email])
    |> validate_format(:email, ~r/\A[\w+\-.]+@[a-z\d\-.]+\.[a-z]+\z/i)
    |> unique_constraint(:name)
    |> unique_constraint(:email)
    |> validate_length(:name, min: 1)
    |> validate_length(:email, max: 50)
  end
end
```

## パスワードカラムの追加

- 追加のデータモデル
  * 対象モデル名: User
  * 対象テーブル名: users
  * 追加カラム(カラム名:型(オプション)): password:string(virtual)、password_digest:string

#### Example:

```cmd
$ mix ecto.gen.migration add_password_clumn_to_users
```

#### File: priv/repo/migrations/[timestamp]_add_password_clumn_to_users

```elixir
defmodule SampleApp.Repo.Migrations.AddPasswordClumnToUsers do
  use Ecto.Migration

  def change do
    alter table(:users) do
      add :password, :string
      add :password_digest, :string, null: false
    end
  end
end
```

#### Example:

```cmd
$ mix ecto.migrate
```

#### File: web/models/user.ex

```elixir
defmodule SampleApp.User do
  use SampleApp.Web, :model

  schema "users" do
    ...

    field :password, :string, virtual: true
    field :password_digest, :string

    timestamps()
  end

  @doc """
  Builds a changeset based on the `struct` and `params`.
  """
  def changeset(struct, params \\ %{}) do
    struct
    |> cast(params, [:name, :email, :password, :password_digest])
    |> validate_required([:name, :email, :password])
    |> ...
  end
end
```

## パスワードの暗号化

他のライブラリを導入しパスワードの暗号化を行ってみましょう。
riverrun/comeonin

#### File: mix.exs

```elixir
defmodule SampleApp.Mixfile do
  ...

  def application do
    [mod: {SampleApp, []},
     applications: [:phoenix, :phoenix_pubsub, :phoenix_html, :cowboy, :logger, :gettext,
                    :phoenix_ecto, :postgrex, :comeonin]]
  end

  ...

  defp deps do
    [{:phoenix, "~> 1.2.1"},
     {:phoenix_pubsub, "~> 1.0"},
     {:phoenix_ecto, "~> 3.0"},
     {:postgrex, ">= 0.0.0"},
     {:phoenix_html, "~> 2.6"},
     {:phoenix_live_reload, "~> 1.0", only: :dev},
     {:gettext, "~> 0.11"},
     {:cowboy, "~> 1.0"},
     {:floki, "~> 0.11.0"},
     {:comeonin, "~> 2.5"}]
  end

  ...
end
```

#### Example:

```cmd
$ mix deps.get
$ mix compile
```

#### Note: Comeoninのオプション設定について

テスト時に速度を落とさないためラウンドの数を減らすように設定することができます。
設定をしなくても動くんで大丈夫です。設定したい方だけ設定してください。

```elixir
file: config/test.exs

# Configure comeonin option
config :comeonin, :bcrypt_log_rounds, 4
config :comeonin, :pdkdf2_rounds, 1
```

#### File: lib/helpers/encryption.ex

```elixir
defmodule SampleApp.Helpers.Encryption do
  import Comeonin.Bcrypt

  def encrypt(password) do
    hashpwsalt(password)
  end

  def check_password(password, password_digest) do
    checkpw(password, password_digest)
  end
end
```

#### File: web/models/user.ex

```elixir
defmodule SampleApp.User do
  ...

  @doc """
  Using Ecto.Multi.run/3 before insert function.
  """
  def set_password_digest(changeset) do
    case Ecto.Changeset.get_change changeset, :password do
      nil ->
        {:error, changeset}
      _ ->
        password_digest = get_field(changeset, :password) |> Encryption.encrypt
        change(changeset, %{password_digest: password_digest})
        {:ok, changeset}
    end
  end
end
```

Callbacksは廃止されたためもうないんです。(バイバイCallbacks!!)
Ecto.Multiを使ってDBへの操作前に上記の関数を実行させます。

## パスワードへのバリデーション

#### File: web/models/user.ex

```elixir
defmodule SampleApp.User do
  ...

  def changeset(struct, params \\ %{}) do
    struct
    |> cast(params, [:name, :email, :password, :password_digest])
    |> validate_required([:name, :email, :password])
    ...
    |> validate_length(:password, min: 8)
    |> validate_length(:password, max: 72)
  end

  ...
end
```

## おわりに

#### Example:

```cmd
$ git add -A
$ git commit -m "Finish modeling_users"
$ git checkout master
$ git merge modeling_users
```

## 参考



