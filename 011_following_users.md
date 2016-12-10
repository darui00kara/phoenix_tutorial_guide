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

#### File: web/router.ex

```elixir
defmodule SampleApp.Router do
  ...

  scope "/", SampleApp do
    pipe_through :browser # Use the default browser stack

    ...
    get "/user/:id/following", UserController, :following
    get "/user/:id/followers", UserController, :followers
  end

  ...
end
```

```elixir
user_path  GET     /user/:id/following  SampleApp.UserController :following
user_path  GET     /user/:id/followers  SampleApp.UserController :followers
```

#### File: web/controllers/user_controller.ex

```elixir
defmodule SampleApp.UserController do
  ...

  def show(conn, %{"id" => id} = params) do
    user  = Repo.get(User, id)
            |> Repo.preload(:relationships)
            |> Repo.preload(:reverse_relationships)
    ...
  end

  ...
end
```

#### File: web/templates/user/show.html.eex

```elixir
<div class="row">
  <aside class="col-md-4">
    <section>
      ...
    </section>
    <section>
      <%= render SampleApp.SharedView, "stats.html",
                                       conn: @conn,
                                       user: @user %>
    </section>
    <%= if current_user?(@conn, @user) do %>
      <section>
        ...
      </section>
    <% end %>
    ...
  </aside>

  <div class="col-md-8">
    ...
  </div>
</div>
```

#### File: web/templates/shared/stats.html.eex

```html
<div class="stats">
  <a href="<%= user_path(@conn, :following, @user) %>">
    <strong id="following" class="stat">
      (<%= Enum.count(@user.followed_users) %>)
    </strong>
    following
  </a>
  <a href="<%= user_path(@conn, :followers, @user) %>">
    <strong id="followers" class="stat">
      (<%= Enum.count(@user.followers) %>)
    </strong>
    followers
  </a>
</div>
```

#### File: web/static/css/custom/_user.scss

```css
/* Users index */

...

/* following and followers */

.stats {
  overflow: auto;
  a {
    float: left;
    padding: 0 10px;
    border-left: 1px solid #eeeeee;
    color: gray;
    &:first-child {
      padding-left: 0;
      border: 0;
    }
    &:hover {
      text-decoration: none;
      color: #3677a3;
    }
  }
  strong {
    display: block;
  }
}
```

#### File: web/controllers/user_controller.ex

```elixir
defmodule SampleApp.UserController do
  ...

  def following(conn, %{"id" => id} = params) do
    user    = Repo.get(User, id)
              |> Repo.preload(:relationships)
              |> Repo.preload(:reverse_relationships)
    id_list = extract_follow_ids(user.followed_users, :followed_id)
    users   = from(u in SampleApp.User,
                where: u.id in ^id_list,
                  order_by: [asc: :name])
              |> Repo.paginate(params)

    if users do
      render(conn, "following.html", user: user, users: users)
    else
      conn
      |> put_flash(:error, "Invalid page number!!")
      |> render("following.html", user: user, users: [])
    end
  end

  def followers(conn, %{"id" => id} = params) do
    user    = Repo.get(User, id)
              |> Repo.preload(:relationships)
              |> Repo.preload(:reverse_relationships)
    id_list = extract_follow_ids(user.followers, :follower_id)
    users   = from(u in SampleApp.User,
                where: u.id in ^id_list,
                  order_by: [asc: :name])
              |> Repo.paginate(params)

    if users do
      render(conn, "followers.html", user: user, users: users)
    else
      conn
      |> put_flash(:error, "Invalid page number!!")
      |> render("followers.html", user: user, users: [])
    end
  end

  ...

  defp extract_follow_ids(follow_users, key) do
    Enum.reduce(follow_users, [], fn(follow_user, acc) ->
      case Map.get(follow_user, key) do
        nil -> acc
         id -> [id | acc]
      end
    end) |> Enum.reverse
  end
end
```

#### File: web/templates/user/show_follow.html.eex

```elixir
<div class="row">
  <aside class="col-md-4">
    <section>
      <%= render SampleApp.SharedView, "user_info.html",
                                       conn: @conn,
                                       user: @user %>
    </section>
    <section>
      <%= render SampleApp.SharedView, "stats.html",
                                       conn: @conn,
                                       user: @user %>
      <%= unless is_empty_list?(@users.entries) do %>
        <div class="user_avatars">
          <%= for follow_user <- @users.entries do %>
            <a href="<%= user_path(@conn, :show, follow_user) %>">
              <img src="<%= gravatar_for(follow_user) %>" class="gravatar">
            </a>
          <% end %>
        </div>
      <% end %>
    </section>
  </aside>

  <div class="col-md-8">
    <h3>Users</h3>
    <%= unless is_empty_list?(@users.entries) do %>
      <ul class="users">
        <%= for follow_user <- @users.entries do %>
          <%= render "user.html", conn: @conn, user: follow_user %>
        <% end %>
      </ul>

      
      <%= Scrivener.HTML.pagination_links @conn, @users, [@user],
                                          view_style: :bootstrap,
                                          path: @path,
                                          action: @action %>     
    <% end %>
  </div>
</div>
```

#### File: web/templates/user/following.html.eex

```html
<h1>Followed users</h1>

<%= render  "show_follow.html",
      action: :following, path: &user_path/4,
      conn: @conn, user: @user, users: @users %>
```

#### File: web/templates/user/followers.html.eex

```html
<h1>Follower users</h1>

<%= render  "show_follow.html",
      action: :followers, path: &user_path/4,
      conn: @conn, user: @user, users: @users %>
```

#### File: web/router.ex

```elixir
defmodule SampleApp.Router do
  ...

  scope "/", SampleApp do
    pipe_through :browser # Use the default browser stack

    ...
    resources "/relationship", RelationshipController, only: [:create, :delete]
  end

  ...
end
```

#### File: web/templates/user/show.html.eex

```elixir
<div class="row">
  <aside class="col-md-4">
    ...
  </aside>

  <div class="col-md-8">
    <%= render SampleApp.RelationshipView, "form.html", conn: @conn, user: @user %>
    ...
  </div>
</div>
```

#### Directory: web/templates/relationship

#### File: web/templates/relationship/form.html.eex

```elixir
<%= unless current_user?(@conn, @user) do %>
  <div id="follow_form">
  <%= if following?(@conn, @user) do %>
    <%= form_tag(relationship_path(@conn, :delete, current_user(@conn)), method: :delete) %>
      <input type="hidden" name="unfollow_id" value="<%= @user.id %>">
      <%= submit "Unfollow", class: "btn btn-default" %>
    </form>
  <% else %>
    <%= form_tag(relationship_path(@conn, :create), method: :post) %>
      <input type="hidden" name="follow_id" value="<%= @user.id %>">
      <%= submit "Follow", class: "btn btn-primary" %>
    </form>
  <% end %>
  </div>
<% end %>
```

#### File: web/views/relationship_view.ex

```elixir
defmodule SampleApp.RelationshipView do
  use SampleApp.Web, :view

  alias SampleApp.User
  alias SampleApp.Helpers.{Following, ViewHelper}

  def following?(conn, %User{id: showing_user_id}) do
    current_user = ViewHelper.current_user(conn)
    Following.following?(current_user.id, showing_user_id)
  end
end
```

#### File: web/controllers/relationship_controller.ex

```elixir
defmodule SampleApp.RelationshipController do
  use SampleApp.Web, :controller

  alias SampleApp.Helpers.Following

  plug SampleApp.Plugs.SignedInUser

  def create(conn, %{"follow_id" => follow_id}) do
    Following.follow!(conn.assigns[:current_user].id, follow_id)

    conn
    |> put_flash(:info, "Follow successfully!")
    |> redirect(to: user_path(conn, :show, follow_id))
  end

  def delete(conn, %{"unfollow_id" => unfollow_id}) do
    Following.unfollow!(conn.assigns[:current_user].id, unfollow_id)

    conn
    |> put_flash(:info, "Unfollow successfully!")
    |> redirect(to: user_path(conn, :show, unfollow_id))
  end
end
```

#### File: web/controllers/user_controller.ex

```elixir
defmodule SampleApp.UserController do
  ...

  def show(conn, %{"id" => id} = params) do
    ...
    id_list = extract_follow_ids(user.followed_users, :followed_id)
    posts = from(m in Micropost,
                   where: m.user_id == ^user.id or m.user_id in ^id_list,
                     order_by: [desc: :inserted_at])
            |> Repo.paginate(params)

    if posts do
      ...
    else
      ...
    end
  end

  ...
end
```

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



