# マイクロポスト

## ゴール

この章で学べること...

- 1対多関係を構築する方法
- Ectoの応用的な使い方
- 投稿機能の作り方

## 概要

## 準備

#### Example:

```cmd
$ git checkout -b user_microposts
```

## マイクロポストのデータモデル

#### Example:

```cmd
$ mix phoenix.gen.model Micropost microposts content:string user_id:references:users
* creating web/models/micropost.ex
* creating test/models/micropost_test.exs
* creating priv/repo/migrations/20161201145453_create_micropost.exs

Remember to update your repository by running migrations:

    $ mix ecto.migrate
```

ユーザと連動した削除の設定を行う。
Not Null制約をつける。

#### File: priv/repo/migrations/[timestamp]_create_micropost.exs

```elixir
defmodule SampleApp.Repo.Migrations.CreateMicropost do
  use Ecto.Migration

  def change do
    create table(:microposts) do
      add :content, :string, null: false
      add :user_id, references(:users, on_delete: :delete_all), null: false

      timestamps()
    end
    create index(:microposts, [:user_id])

  end
end
```

#### Example:

```cmd
$ mix ecto.migrate
```

#### Example:

```cmd
                      Table "public.users"
     Column      |            Type             |   Modifiers
-----------------+-----------------------------+--------------------
 id              | integer                     | not null default... 
 name            | character varying(255)      | 
 email           | character varying(255)      | 
 inserted_at     | timestamp without time zone | not null
 updated_at      | timestamp without time zone | not null
 password        | character varying(255)      | 
 password_digest | character varying(255)      | not null
Indexes:
    "users_pkey" PRIMARY KEY, btree (id)
    "users_email_index" UNIQUE, btree (email)
    "users_name_index" UNIQUE, btree (name)
Referenced by:
    TABLE "microposts" CONSTRAINT "microposts_user_id_fkey"
      FOREIGN KEY (user_id) REFERENCES users(id) ON DELETE CASCADE
```

```cmd
                     Table "public.microposts"
   Column    |            Type             |   Modifiers
-------------+-----------------------------+--------------------
 id          | integer                     | not null default...
 content     | character varying(255)      | not null
 user_id     | integer                     | not null
 inserted_at | timestamp without time zone | not null
 updated_at  | timestamp without time zone | not null
Indexes:
    "microposts_pkey" PRIMARY KEY, btree (id)
    "microposts_user_id_index" btree (user_id)
Foreign-key constraints:
    "microposts_user_id_fkey" FOREIGN KEY (user_id) REFERENCES users(id) ON DELETE CASCADE
```

外部キー(foreign key)の名前はデフォルトでは、belongs_toで指定している名前+_idで生成される。
:foreign_keyオプションを使えば自分で名前を指定することもできる。

## ユーザとマイクロポストの関連付け

#### File: web/models/user.ex

```elixir
defmodule SampleApp.User do
  ...

  schema "users" do
    ...

    has_many :microposts, SampleApp.Micropost

    timestamps()
  end

  ...
end
```

外部キーに指定しているカラムがcast/3とvalidate_required/3に入ってないため追加

#### File: web/models/micropost.ex

```elixir
defmodule SampleApp.Micropost do
  ...

  def changeset(struct, params \\ %{}) do
    struct
    |> cast(params, [:content, :user_id])
    |> validate_required([:content, :user_id])
  end
end
```

動作確認

#### Example:

```elixir
iex> alias SampleApp.{Repo, User, Micropost}
[SampleApp.Repo, SampleApp.User, SampleApp.Micropost]

## ユーザの作成
iex> user_param = %{name: "hoge", email: "hoge@test.com", password: "hogehoge"}
%{email: "hoge@test.com", name: "hoge", password: "hogehoge"}
iex> User.changeset(%User{}, user_param) |> Repo.insert!
...
%SampleApp.User{__meta__: #Ecto.Schema.Metadata<:loaded, "users">,
 email: "hoge@test.com", id: 1,
 inserted_at: #Ecto.DateTime<2016-12-04 05:44:28>,
 microposts: #Ecto.Association.NotLoaded<association :microposts is not loaded>,
 name: "hoge", password: "hogehoge",
 password_digest: "$2b$12$/7Ga5hmbO54dyQwFiYuzN.766IM5yqxF2Sc4Kvq35UbQyPUwNmJ9i",
 updated_at: #Ecto.DateTime<2016-12-04 05:44:28>}

## ユーザの取得
iex> user = Repo.get(User, 1)
...
%SampleApp.User{__meta__: #Ecto.Schema.Metadata<:loaded, "users">,
 email: "hoge@test.com", id: 1,
 inserted_at: #Ecto.DateTime<2016-12-04 05:44:28>,
 microposts: #Ecto.Association.NotLoaded<association :microposts is not loaded>,
 name: "hoge", password: nil,
 password_digest: "$2b$12$/7Ga5hmbO54dyQwFiYuzN.766IM5yqxF2Sc4Kvq35UbQyPUwNmJ9i",
 updated_at: #Ecto.DateTime<2016-12-04 05:44:28>}

## 取得したユーザと組み合わせてマイクロポストのChangesetを生成
iex> micropost_changeset = Ecto.build_assoc(user, :microposts, content: "Hello hoge!!")
%SampleApp.Micropost{__meta__: #Ecto.Schema.Metadata<:built, "microposts">,
 content: "Hello hoge!!", id: nil, inserted_at: nil, updated_at: nil,
 user: #Ecto.Association.NotLoaded<association :user is not loaded>, user_id: 1}
iex> Repo.insert!(micropost_changeset)
...
%SampleApp.Micropost{__meta__: #Ecto.Schema.Metadata<:loaded, "microposts">,
 content: "Hello hoge!!", id: 1,
 inserted_at: #Ecto.DateTime<2016-12-04 05:50:38>,
 updated_at: #Ecto.DateTime<2016-12-04 05:50:38>,
 user: #Ecto.Association.NotLoaded<association :user is not loaded>, user_id: 1}

## ユーザとマイクロポストの取得
iex> Repo.get(User, 1) |> Repo.preload(:microposts)
...
SampleApp.User{__meta__: #Ecto.Schema.Metadata<:loaded, "users">,
 email: "hoge@test.com", id: 1,
 inserted_at: #Ecto.DateTime<2016-12-04 05:44:28>,
 microposts: [%SampleApp.Micropost{__meta__: #Ecto.Schema.Metadata<:loaded, "microposts">,
   content: "Hello hoge!!", id: 1,
   inserted_at: #Ecto.DateTime<2016-12-04 05:50:38>,
   updated_at: #Ecto.DateTime<2016-12-04 05:50:38>,
   user: #Ecto.Association.NotLoaded<association :user is not loaded>,
   user_id: 1}], name: "hoge", password: nil,
 password_digest: "$2b$12$/7Ga5hmbO54dyQwFiYuzN.766IM5yqxF2Sc4Kvq35UbQyPUwNmJ9i",
 updated_at: #Ecto.DateTime<2016-12-04 05:44:28>}

## ユーザに関連したマイクロポストを全て取得
iex> Ecto.assoc(user, :microposts)
#Ecto.Query<from m in SampleApp.Micropost, where: m.user_id == ^1>
iex> Ecto.assoc(user, :microposts) |> Repo.all  
...
[%SampleApp.Micropost{__meta__: #Ecto.Schema.Metadata<:loaded, "microposts">,
  content: "Hello hoge!!", id: 1,
  inserted_at: #Ecto.DateTime<2016-12-04 05:50:38>,
  updated_at: #Ecto.DateTime<2016-12-04 05:50:38>,
  user: #Ecto.Association.NotLoaded<association :user is not loaded>,
  user_id: 1}]
```

### 連動した削除

連動した削除を確認

#### Example:

```elixir
iex> Repo.all(User)
iex> Repo.all(Micropost)
iex> Repo.get(User, 1) |> Repo.delete
...
{:ok,
 %SampleApp.User{__meta__: #Ecto.Schema.Metadata<:deleted, "users">,
  email: "hoge@test.com", id: 1,
  inserted_at: #Ecto.DateTime<2016-12-04 05:44:28>,
  microposts: #Ecto.Association.NotLoaded<association :microposts is not loaded>,
  name: "hoge", password: nil,
  password_digest: "$2b$12$/7Ga5hmbO54dyQwFiYuzN.766IM5yqxF2Sc4Kvq35UbQyPUwNmJ9i",
  updated_at: #Ecto.DateTime<2016-12-04 05:44:28>}}
iex> Repo.all(User)
iex> Repo.all(Micropost)
```

## マイクロポストへのバリデーション

#### File: web/models/micropost.ex

```elixir
defmodule SampleApp.Micropost do
  ...

  def changeset(struct, params \\ %{}) do
    struct
    |> cast(params, [:content, :user_id])
    |> validate_required([:content, :user_id])
    |> validate_length(:content, min: 1)
    |> validate_length(:content, max: 140)
  end
end
```

## マイクロポストの一覧

#### File: web/controllers/user_controller.ex

```elixir
defmodule SampleApp.UserController do
  ...

  def show(conn, %{"id" => id} = params) do
    user = Repo.get!(User, id)
    posts = Ecto.assoc(user, :microposts) |> Repo.all

    render(conn, "show.html", user: user, posts: posts)
  end

  ...
end
```

#### File: web/templates/user/show.html.eex

```html
<div class="row">
  <aside class="col-md-4">
    ...
  </aside>

  <div class="col-md-8">
    <%= unless is_empty_list?(@posts.entries) do %>
      <h3>Microposts</h3>
      <ol class="microposts">
        <li>
          <%= for post <- @posts.entries do %>
            <span class="content"><%= post.content %></span>
            <span class="timestamp">
              Posted <%= post.inserted_at %> ago.
            </span>
          <% end %>
        </li>
      </ol>
    <% end %>
  </div>
</div>
```

#### File: web/static/css/custom/_micropost.scss

```css
/* microposts */

.microposts {
  list-style: none;
  margin: 10px 0 0 0;
  li {
    padding: 10px 0;
    border-top: 1px solid #e8e8e8;
  }
  .content {
    display: block;
  }
  .timestamp {
    color: #777777;
  }
}
```

#### File: web/static/css/custom/_sidebar.scss

```css
/* sidebar */

aside {
  section {
    ... 
  }
  textarea {
    height: 100px;
    margin-bottom: 5px;
  }
}
```

#### File: web/static/css/custom/custom.scss

```css
/* custom main scss */

...
@import "micropost";
```

## マイクロポストのページネーション

#### File: web/controllers/user_controller.ex

```elixir
defmodule SampleApp.UserController do
  ...

  alias SampleApp.User
  alias SampleApp.Micropost

  ...

  def show(conn, %{"id" => id} = params) do
    user  = Repo.get!(User, id)
    posts = from(m in Micropost,
                   where: m.user_id == ^user.id,
                     order_by: [desc: :inserted_at])
            |> Repo.paginate(params)

    if posts do
      render(conn, "show.html", user: user, posts: posts)
    else
      conn
      |> put_flash(:error, "Invalid page number!!")
      |> render("show.html", user: user, posts: [])
    end
  end

  ...
end
```

#### File: web/templates/user/show.html.eex

```html
<div class="row">
  <aside class="col-md-4">
    ...
  </aside>

  <div class="col-md-8">
    <%= unless is_empty_list?(@posts.entries) do %>
      <h3>Microposts</h3>
      <ol class="microposts">
        ...
      </ol>
      <%= Scrivener.HTML.pagination_links @conn, @posts, [@user],
                                          view_style: :bootstrap,
                                          path: &user_path/4,
                                          action: :show %>
    <% end %>
  </div>
</div>
```

## 投稿と削除

### マイクロポストのコントローラ

#### File: web/router.ex

```elixir

defmodule SampleApp.Router do
  ...

  scope "/", SampleApp do
    pipe_through :browser # Use the default browser stack

    ...
    resources "/post", MicropostController, only: [:create, :delete]
  end

  ...
end
```

#### Example:

```elixir
micropost_path  POST    /post           SampleApp.MicropostController :create
micropost_path  DELETE  /post/:id       SampleApp.MicropostController :delete
```

#### File: web/contrllers/micropost_controller.ex

```elixir
defmodule SampleApp.MicropostController do
  use SampleApp.Web, :controller

  plug SampleApp.Plugs.SignedInUser

  def create(conn, _params) do
    redirect conn, to: user_path(conn, :show, conn.assigns[:current_user])
  end

  def delete(conn, _params) do
    redirect conn, to: user_path(conn, :show, conn.assigns[:current_user])
  end
end
```

### 投稿機能

#### Directory: web/templates/micropost

#### File: web/templates/micropost/form.html.eex

```elixir
<%= if current_user?(@conn, @user) do %>
  <%= form_for @conn, micropost_path(@conn, :create),
               [as: :micropost_param], fn f -> %>
    <%= if f.errors != [] do %>
      <div class="alert alert-danger">
        <p>Oops, something went wrong! Please check the errors below.</p>
      </div>
    <% end %>

    <div class="form-group">
      <%= label f, :content, class: "control-label" %>
      <%= textarea f, :content, class: "form-control" %>
      <%= error_tag f, :content %>
    </div>

    <div class="form-group">
      <%= submit "Post!", class: "btn btn-primary" %>
    </div>
  <% end %>
<% end %>
```

#### File: lib/helpers/view_helper.ex

```elixir
defmodule SampleApp.Helpers.ViewHelper do
  alias SampleApp.{Repo, User}

  ...

  def current_user?(conn, %User{id: id}) do
    current_user(conn) == Repo.get(User, id)
  end
end
```

#### File: web/templates/user/show.html.eex

```html
<div class="row">
  <aside class="col-md-4">
    ...
    <%= if current_user?(@conn, @user) do %>
      <section>
        <%= link "Edit", to: user_path(@conn, :edit, @user),
                         class: "btn btn-default btn-xs" %>
        <%= button "Delete", to: user_path(@conn, :delete, @user),
                             method: :delete,
                             onclick: "return confirm(\"Are you sure?\");",
                             class: "btn btn-danger btn-xs" %>
      </section>
    <% end %>
    <section>
      <%= render SampleApp.MicropostView, "form.html", conn: @conn, user: @user %>
    </section>
  </aside>

  ...
</div>
```

#### File: web/contrllers/micropost_controller.ex

```elixir
defmodule SampleApp.MicropostController do
  ...

  def create(conn, %{"micropost_param" => %{"content" => content}}) do
    changeset =  Ecto.build_assoc(conn.assigns[:current_user],
                                  :microposts, content: content)

    conn = case Repo.insert(changeset) do
      {:ok, _} ->
        put_flash(conn, :info, "Post successfully.")
      {:error, _} ->
        put_flash(conn, :error, "Post Failed.")
    end

    redirect conn, to: user_path(conn, :show, conn.assigns[:current_user])
  end

  ...
end
```

### 削除機能

#### File: web/contrllers/micropost_controller.ex

```elixir
defmodule SampleApp.MicropostController do
  ...

  alias SampleApp.Micropost

  ...

  def delete(conn, _%{"id" => id}) do
    Repo.get(Micropost, id) |> Repo.delete

    conn
    |> put_flash(:info, "Micropost deleted successfully.")
    |> redirect(to: user_path(conn, :show, conn.assigns[:current_user]))
  end
end
```

#### File: web/templates/user/show.html.eex

```html
<div class="row">
  <aside class="col-md-4">
    ...
  </aside>

  <div class="col-md-8">
    <%= unless is_empty_list?(@posts.entries) do %>
      <h3>Microposts</h3>
      <ol class="microposts">
        <li>
          <%= for post <- @posts.entries do %>
            <span class="content"><%= post.content %></span>
            <span class="timestamp">
              Posted <%= post.inserted_at %> ago.
            </span>
            <%= if @user.id == post.user_id do %>
              <%= link "Delete", to: micropost_path(@conn, :delete, post),
                                 method: :delete,
                                 class: "btn btn-danger btn-xs" %>
            <% end %>
          <% end %>
        </li>
      </ol>
      <%= Scrivener.HTML.pagination_links @conn, @posts, [@user],
                                          view_style: :bootstrap,
                                          path: &user_path/4,
                                          action: :show %>
    <% end %>
  </div>
</div>
```

## 共有ビュー

#### File: web/view/shared_view.ex

```elixir
defmodule SampleApp.SharedView do
  use SampleApp.Web, :view
end
```

#### File: web/view/user_view.ex

```elixir
defmodule SampleApp.UserView do
  use SampleApp.Web, :view

  def is_empty_list?(list) when is_list(list) do
    list == []
  end
end
```

#### Directory: web/templates/shared

#### File: web/templates/shared/user_info.html.eex

```html
<a href="<%= user_path(@conn, :show, @user) %>">
  <img src="<%= gravatar_for(@user) %>" class="gravatar">
</a>
<h1><%= @user.name %></h1>
```

#### File: lib/helpers/view_helper.ex

```elixir
defmodule SampleApp.Helpers.ViewHelper do
  ...
  alias SampleApp.Helpers.Gravatar

  ...

  def gravatar_for(%{email: email}) do
    Gravatar.get_gravatar_url(email, 50)
  end
end
```

#### File: web/templates/shared/microposts.html.eex

```html
<ol class="microposts">
  <li>
    <%= for post <- @posts.entries do %>
      <span class="content"><%= post.content %></span>
      <span class="timestamp">
        Posted <%= post.inserted_at %> ago.
      </span>
      <%= if current_user?(@conn, @user) do %>
        <%= if @user.id == post.user_id do %>
          <%= link "Delete", to: micropost_path(@conn, :delete, post),
                             method: :delete,
                             class: "btn btn-danger btn-xs" %>
        <% end %>
      <% end %>
    <% end %>
  </li>
</ol>
```

#### File: web/templates/user/show.html.eex

```elixir
<div class="row">
  <aside class="col-md-4">
    <section>
      <%= render SampleApp.SharedView, "user_info.html",
                                       conn: @conn,
                                       user: @user %>
    </section>
    ...
  </aside>

  <div class="col-md-8">
    <%= unless is_empty_list?(@posts.entries) do %>
      <h3>Microposts</h3>
      <%= render SampleApp.SharedView, "microposts.html",
                                       conn: @conn,
                                       posts: @posts,
                                       user: @user %>
      
      <%= Scrivener.HTML.pagination_links @conn, @posts, [@user],
                                          view_style: :bootstrap,
                                          path: &user_path/4,
                                          action: :show %>
    <% end %>
  </div>
</div>
```

## おわりに

#### Example:

```cmd
$ git add -A
$ git commit -m "Finish user_microposts"
$ git checkout master
$ git merge user_microposts
```

## 参考



