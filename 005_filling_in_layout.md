# レイアウトを作成する 

## ゴール

この章で学べること...

- Phoenixフレームワークでのレイアウトの作り方
- PhoenixにBootstrapを導入する一例
- テンプレートの仕組み

## 概要



## 準備

#### Example:

```cmd
$ git branch
  demo_app
* master
  static_pages
$ git branch filling_in_layout
$ git checkout filling_in_layout
$ git branch
  demo_app
* filling_in_layout
  master
  static_pages
```

## BootstrapとカスタムCSSの導入

Bootstrapの導入
導入するのはBootstrap3
npmからインストールする都合でBootstrap4は断念

### Bootstrap3

Bootstrap3と必要になるパッケージの取得

#### Example:

```cmd
$ npm install --save-dev sass-brunch copycat-brunch
$ npm install --save bootstrap-sass jquery
```

#### Rename: web/static/css/app.css to web/static/css/app.scss

#### File: brunch-config.js

カンマ(,)忘れやすいんで注意。

```javascript
exports.config = {
  // See http://brunch.io/#documentation for docs.
  files: {
    ...
    stylesheets: {
      joinTo: "css/app.css",
      order: {
        after: ["web/static/css/app.scss"] // concat app.css last
      }
    },
    ...
  },

  ...

  // Configure your plugins
  plugins: {
    babel: {
      ...
    },
    sass: {
      options: {
        includePaths: ["node_modules/bootstrap-sass/assets/stylesheets"],
        precision: 8
      }
    },
    copycat: {
      "fonts": ["node_modules/bootstrap-sass/assets/fonts/bootstrap"]
    }
  },

  ...

  npm: {
    enabled: true,
    globals: {
      $: 'jquery',
      jQuery: 'jquery',
      bootstrap: 'bootstrap-sass'
    }
  }
};
```

下記追加。

#### web/static/css/app.scss

```css
$icon-font-path: "/font/";
@import "bootstrap";
```

#### Note: 静的資産の取り扱いについて

```txt
取扱説明書...(いいからさっさと書け)
```

```txt
もしjQueryが読み込まれているか心配な方がいれば、
web/templates/layout/app.html.eexのapp.jsを読み込んでいる下へ下記を追加してください。

<script>
  $(function(){
    alert('jQuery is ready.')
  });
</script>

jQueryの読み込みに成功していればダイアログが表示されます。
```

### カスタムCSS

カスタムCSSの導入について

後々SCSS/SASSのファイルは分割で管理するので調査。
参考: http://cartman0.hatenablog.com/entry/2015/07/17/060125

#### web/static/css/app.scss

```css
/* This file is for your main application css. */

$icon-font-path: "/font/";
@import "bootstrap";
@import "custom/custom";
```

#### web/static/css/custom/custom.scss

```css
/* custom main scss */
@import "base";
```

#### web/static/css/custom/_base.scss

```css
/* universal */
html {
  overflow-y: scroll;
}

body {
  padding-top: 60px;
}

section {
  overflow: auto;
}

textarea {
  resize: vertical;
}

.center {
  text-align: center;
}

.center h1 {
  margin-bottom: 10px;
}

/* typography */
h1, h2, h3, h4, h5, h6 {
  line-height: 1;
}

h1 {
  font-size: 3em;
  letter-spacing: -2px;
  margin-bottom: 30px;
  text-align: center;
}

h2 {
  font-size: 1.2em;
  letter-spacing: -1px;
  margin-bottom: 30px;
  text-align: center;
  font-weight: normal;
  color: #777777;
}

p {
  font-size: 1.1em;
  line-height: 1.7em;
}
```

## レイアウトテンプレート

#### web/templates/layout/app.html.eex

```html
<!DOCTYPE html>
<html lang="en">
  <head>
    <meta charset="utf-8">
    <meta http-equiv="X-UA-Compatible" content="IE=edge">
    <meta name="viewport" content="width=device-width, initial-scale=1">
    <meta name="description" content="">
    <meta name="author" content="">

    <title>Hello SampleApp!</title>
    <link rel="stylesheet" href="<%= static_path(@conn, "/css/app.css") %>">
    
    <!--[if lt IE 9]>
    <script src="http://html5shim.googlecode.com/svn/trunk/html5.js"></script>
    <![endif]-->
  </head>

  <body>
    <header class="header navbar navbar-inverse">
      <div class="navbar-inner">
        <div class="container">
          <a class="logo" href="<%= page_path(@conn, :index) %>"></a>
          <nav role="navigation">
            <ul class="nav nav-pills pull-right">
              <li><a href="<%= static_page_path(@conn, :home) %>">Home</a></li>
              <li><a href="<%= static_page_path(@conn, :help) %>">Help</a></li>
              <li><a href="<%= static_page_path(@conn, :about) %>">About</a></li>
              <li><a href=#>Sign in</a></li>
              <li><a href="http://www.phoenixframework.org/docs">Get Started</a></li>
            </ul>
          </nav>
        </div> <!-- container -->
      </div> <!-- navbar-inner -->
    </header>

    <div class="container">
      <h2>
        <p class="alert alert-info" role="alert"><%= get_flash(@conn, :info) %></p>
        <p class="alert alert-danger" role="alert"><%= get_flash(@conn, :error) %></p>
      </h2>

      <main role="main">
        <%= render @view_module, @view_template, assigns %>
      </main>
    </div> <!-- /container -->

    <script src="<%= static_path(@conn, "/js/app.js") %>"></script>
  </body>
</html>
```

#### Example:

```cmd
$ mix phoenix.server
```

#### URL: http://localhost:4000

BootstrapとjQueryはbrunch-config.jsで統合されている。

#### Example:

```html
<!--[if lt IE 9]>
<script src="http://html5shim.googlecode.com/svn/trunk/html5.js"></script>
<![endif]-->
```

#### Example:

```html
<header class="header navbar navbar-inverse">
  <div class="navbar-inner">
    <div class="container">
      <a class="logo" href="<%= page_path(@conn, :index) %>"></a>
      <nav role="navigation">
        <ul class="nav nav-pills pull-right">
          <li><a href="<%= static_page_path(@conn, :home) %>">Home</a></li>
          <li><a href="<%= static_page_path(@conn, :help) %>">Help</a></li>
          <li><a href="<%= static_page_path(@conn, :about) %>">About</a></li>
          <li><a href=#>Sign in</a></li>
          <li><a href="http://www.phoenixframework.org/docs">Get Started</a></li>
        </ul>
      </nav>
    </div> <!-- container -->
  </div> <!-- navbar-inner -->
</header>
```

#### Example:

```html
<%= render @view_module, @view_template, assigns %>
```

## リンクとパスヘルパー

#### Example:

```elixir
<%= static_page_path(@conn, :home) %>
```

#### Example:

```html
<%= if true do %>
  <p>trueだよ</p>
<% else %>
  <p>falseだよ</p>>
<% end %>
```

#### Example:

```cmd
$ mix phoenix.routes
       page_path  GET  /       SampleApp.PageController :index
static_page_path  GET  /home   SampleApp.StaticPageController :home
...
```

#### Example:

```html
<li><a href="<%= static_page_path(@conn, :home) %>">Home</a></li>
```

#### Example:

```html
<li><%= link "Home", to: static_page_path(@conn, :home) %></li>
```

## レンダリングチェイン

#### File: web/templates/layout/app.html.eex

```elixir
<!DOCTYPE html>
<html lang="en">
  <head>
    <meta charset="utf-8">
    <meta http-equiv="X-UA-Compatible" content="IE=edge">
    <meta name="viewport" content="width=device-width, initial-scale=1">
    <meta name="description" content="">
    <meta name="author" content="">

    <title>Hello SampleApp!</title>
    <link rel="stylesheet" href="<%= static_path(@conn, "/css/app.css") %>">

    <%= render "shim.html" %>
  </head>

  <body>
    <%= render "header.html", conn: @conn %>

    <div class="container">
      <h2>
        <p class="alert alert-info" role="alert"><%= get_flash(@conn, :info) %></p>
        <p class="alert alert-danger" role="alert"><%= get_flash(@conn, :error) %></p>
      </h2>

      <main role="main">
        <%= render @view_module, @view_template, assigns %>
      </main>

      <%= render "footer.html", conn: @conn %>
    </div> <!-- /container -->

    <script src="<%= static_path(@conn, "/js/app.js") %>"></script>
  </body>
</html>
```

#### Example:

```elixir
<%= render "テンプレート名", 引数 %>
```

#### Example:

```elixir
<%= render "shim.html" %>
<%= render "header.html", conn: @conn %>
<%= render "footer.html", conn: @conn %>
```

#### File: web/templates/layout/shim.html.eex

```html
<!--[if lt IE 9]>
<script src="http://html5shim.googlecode.com/svn/trunk/html5.js"></script>
<![endif]-->
```

#### File: web/templates/layout/header.html.eex

```html
<header class="header navbar navbar-inverse">
  <div class="navbar-inner">
    <div class="container">
      <a class="logo" href="<%= page_path(@conn, :index) %>"></a>
      <nav role="navigation">
        <ul class="nav nav-pills pull-right">
          <li><%= link "Home", to: static_page_path(@conn, :home) %></li>
          <li><%= link "Help", to: static_page_path(@conn, :help) %></li>
          <li><a href=#>Sign in</a></li>
        </ul>
      </nav>
    </div> <!-- container -->
  </div> <!-- navbar-inner -->
</header>
```

#### File: web/templates/layout/footer.html.eex

```html
<footer class="footer">
  <nav>
    <ul>
      <li><%= link "About", to: static_page_path(@conn, :about) %></li>
      <li><a href="http://www.phoenixframework.org/docs">Phoenix Get Started</a></li>
    </ul>
  </nav>
</footer>
```

#### File: web/static/css/custom/_header.scss

```css
/* header */
#logo {
  float: left;
  margin-right: 10px;
  font-size: 1.7em;
  color: #ffffff;
  text-transform: uppercase;
  letter-spacing: -1px;
  padding-top: 9px;
  font-weight: bold;
}

#logo:hover {
  color: #ffffff;
  text-decoration: none;
}
```

#### File: web/static/css/custom/_header.scss

```css
/* footer */
footer {
  margin-top: 45px;
  padding-top: 5px;
  border-top: 1px solid #eaeaea;
  color: #777777;
}

footer a {
  color: #555555;
}

footer a:hover {
  color: #222222;
}

footer small {
  float: left;
}

footer ul {
  float: right;
  list-style: none;
}

footer ul li {
  float: left;
  margin-left: 15px;
}
```

## Contactページの追加

#### File: web/router.ex

```elixir
defmodule SampleApp.Router do
  ...

  scope "/", SampleApp do
    pipe_through :browser # Use the default browser stack

    ...
    get "/contact", StaticPageController, :contact
  end

  ...
end
```

#### File: web/controllers/static_page_controller.ex

```elixir
defmodule SampleApp.StaticPageController do
  ...

  def contact(conn, _params) do
    render conn, "contact.html"
  end
end
```

#### File: web/templates/static_page/contact.html.eex

```html
<div class="jumbotron">
  <h1>Contact</h1>
</div>
```

#### File: web/templates/layout/footer.html.eex

```html
<footer class="footer">
  <nav>
    <ul>
      <li><%= link "About", to: static_page_path(@conn, :about) %></li>
      <li><%= link "Contact", to: static_page_path(@conn, :contact) %></li>
      <li><a href="http://www.phoenixframework.org/docs">Phoenix Get Started</a></li>
    </ul>
  </nav>
</footer>
```

## SCSSの記述へ

SCSSファイルの内容をCSS形式で記述している。
せっかくSCSSを使用しているのにもったいないので記述を修正する。

#### File: web/static/css/custom/custom.scss

```css
/* custom main scss */

@import "mixins";
@import "base";
@import "header";
@import "footer";
```

#### File: web/static/css/custom/_mixins.scss

```css
/* mixins */

$grayMediumLight: #eaeaea;
$grayLight: #999;
```

#### File: web/static/css/custom/_base.scss

```css
/* universal */

html {
  overflow-y: scroll;
}

body {
  padding-top: 60px;
}

section {
  overflow: auto;
}

textarea {
  resize: vertical;
}

.center {
  text-align: center;
  h1 {
    margin-bottom: 10px;
  }
}

/* typography */

h1, h2, h3, h4, h5, h6 {
  line-height: 1;
}

h1 {
  font-size: 3em;
  letter-spacing: -2px;
  margin-bottom: 30px;
  text-align: center;
}

h2 {
  font-size: 1.2em;
  letter-spacing: -1px;
  margin-bottom: 30px;
  text-align: center;
  font-weight: normal;
  color: $grayLight;
}

p {
  font-size: 1.1em;
  line-height: 1.7em;
}
```

#### File: web/static/css/custom/_header.scss

```css
/* header */

#logo {
  float: left;
  margin-right: 10px;
  font-size: 1.7em;
  color: white;
  text-transform: uppercase;
  letter-spacing: -1px;
  padding-top: 9px;
  font-weight: bold;
  line-height: 1;
  &:hover {
    color: white;
    text-decoration: none;
  }
}
```

#### File: web/static/css/custom/_footer.scss

```css
/* footer */

footer {
  margin-top: 45px;
  padding-top: 5px;
  border-top: 1px solid $grayMediumLight;
  color: $grayLight;
  a {
    color: $gray;
    &:hover { 
      color: $grayDarker;
    }
  }  
  small { 
    float: left; 
  }
  ul {
    float: right;
    list-style: none;
    li {
      float: left;
      margin-left: 10px;
    }
  }
}
```

## サインアップへの仕込み

#### File: web/templates/static_page/home.html.eex

```html
<div class="jumbotron">
  <h1>Welcome to the Sample App</h1>

  <h2>
    This application is
    <a href="http://railstutorial.jp/">Ruby on Rails Tutorial</a>
    for Phoenix sample application.
  </h2>

  <a href="#" class="btn btn-large btn-primary">Sign up now!</a>
</div>
```

#### File: web/controllers/static_page_controller.ex

```elixir
defmodule SampleApp.StaticPageController do
  ...

  def home(conn, _params) do
    render conn, "home.html"
  end

  ...
end
```

## おわりに

#### Example:

```cmd
$ git add -A
$ git commit -m "Finish filling_in_layout"
$ git checkout master
$ git merge filling_in_layout
```


