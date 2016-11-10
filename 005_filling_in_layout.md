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
@import "custom";
```

#### web/static/css/custom.scss

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

## リンクとパスヘルパー

## レンダリングチェイン

## Contactページの追加

## サインアップへの仕込み

## おわりに




