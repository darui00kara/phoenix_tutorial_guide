# フォロー機能

## ゴール

この章で学べること...

- 多対多関係を構築する方法
- フォロー/アンフォロー機能の作り方
-

## 概要

## 準備

#### Example:

```cmd
$ git checkout -b following_users
```

## リレーションシップのデータモデル

### データモデル

#### Example:

```txt

```

### リレーションシップテーブルの作成

#### Example:

```cmd
$ mix phoenix.gen.model Relationship relationships follower_id:references:users followed_id:references:users
```

#### File: priv/repo/migrations/[timestamp]_create_relationship.exs

```elixir
defmodule SampleApp.Repo.Migrations.CreateRelationship do
  use Ecto.Migration
  @disable_ddl_transaction true

  def change do
    create table(:relationships) do
      add :follower_id, references(:users, on_delete: :nothing), null: false
      add :followed_id, references(:users, on_delete: :nothing), null: false

      timestamps()
    end

    create index(:relationships, [:follower_id], concurrently: true)
    create index(:relationships, [:followed_id], concurrently: true)
    create index(:relationships, [:follower_id, :followed_id], unique: true, concurrently: true)
  end
end
```

#### Example:

```cmd
$ mix ecto.migrate
```

## ユーザとリレーションシップ

### 構築する関連の説明

多対多のつながりを設定する場合は:throughオプションを利用する。
2つのモデルの間に第3のモデルが介在するのが特徴です。
本章で言えば第3のモデルとなるのはリレーションシップになります。
しかし、対象としているモデルがUserしかありません。
これは一体どうゆうことでしょうか？私が作り方を間違えてしまったのでしょうか。

大丈夫！安心してください。この形で間違えてはいません。
それがどういうことなのか、これから説明していきます。

#### Example:

```txt
+----+       +------------+       +----+
|User| 1---n |Relationship| n---1 |User|
+----+       +------------+       +----+
```

```txt
users table
+----+-------+
| id | name  |
+----+-------+
| 1  | user1 |
+----+-------+
| 2  | user2 |
+----+-------+

relationships table
+-------------+-------------+
| followed_id | follower_id |
+-------------+-------------+
| 1           | 2           |
+-------------+-------------+
| 2           | 1           |
+-------------+-------------+
```

### 多対多の関連付け

#### File: web/models/user.ex

```elixir
defmodule SampleApp.User do
  ...

  schema "users" do
    ...

    has_many :followed_users, SampleApp.Relationship, foreign_key: :follower_id
    has_many :relationships, through: [:followed_users, :followed_user]

    timestamps()
  end

  ...
end
```

#### File: web/models/relationship.ex

```elixir
defmodule SampleApp.Relationship do
  use SampleApp.Web, :model

  schema "relationships" do
    belongs_to :followed_user, SampleApp.User, foreign_key: :follower_id
    ...

    timestamps()
  end

  ...
end
```

#### File: web/models/user.ex

```elixir
defmodule SampleApp.User do
  ...

  schema "users" do
    ...

    has_many :followers, SampleApp.Relationship, foreign_key: :followed_id
    has_many :reverse_relationships, through: [:followers, :follower]

    timestamps()
  end

  ...
end
```

#### File: web/models/relationship.ex

```elixir
defmodule SampleApp.Relationship do
  use SampleApp.Web, :model

  schema "relationships" do
    ...
    belongs_to :follower, SampleApp.User, foreign_key: :followed_id

    timestamps()
  end

  ...
end
```

#### File: web/models/relationship.ex

```elixir
defmodule SampleApp.Relationship do
  ...

  def changeset(struct, params \\ %{}) do
    struct
    |> cast(params, [:follower_id, :followed_id])
    |> validate_required([:follower_id, :followed_id])
  end
end
```

### 補助関数

#### File: lib/Helpers/following.ex

```elixir
defmodule SampleApp.Helpers.Following do
  import Ecto.Query, only: [from: 2]

  alias SampleApp.{Repo, Relationship}

  def follow!(signin_id, followed_id) do
    changeset = Relationship.changeset(
                  %Relationship{},
                  %{follower_id: signin_id, followed_id: followed_id})

    Repo.insert!(changeset)
  end

  def following?(signin_id, followed_id) do
    relationship = from(r in Relationship,
                     where: r.follower_id == ^signin_id and r.followed_id == ^followed_id,
                     limit: 1) |> Repo.all

    !Enum.empty?(relationship)
  end

  def unfollow!(signin_id, followed_id) do
    [relationship] = from(r in Relationship,
              where: r.follower_id == ^signin_id and r.followed_id == ^followed_id,
              limit: 1) |> Repo.all

    Repo.delete!(relationship)
  end
end
```

## フォローとフォロワー

### フォローとフォロワーは何人ですか？

### フォローとフォロワーの一覧

### ページネーション

## フォローボタン

## おわりに

#### Example:

```cmd
$ git add -A
$ git commit -m "Finish following_users"
$ git checkout master
$ git merge following_users
```

## 参考



