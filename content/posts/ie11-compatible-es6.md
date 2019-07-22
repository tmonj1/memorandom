---
title: "IE 11 対応の ES6 コードを書くための準備"
date: 2018-12-30T23:40:33+09:00
draft: false
---

## 何が問題か?

ES6 で書いても Babel でトランスパイルし、polyfill を使えば IE 11 でも実行できます。多くの場合は。

ただ残念ながらこれだけでは動かないこともあります。たとえば、ES6 の新機能である Proxy を使っていたりすると IE では動作しません。なぜなら、Babel の polyfill は Proxy には対応していないからです[^1]。

Babel は ES6 以降の新しい JavaScript のシンタックスを ES5 に変換してくれますが、残念ながら Proxy には非対応なので放置されます。Proxy を使っている箇所はそのまま Babel を素通りして IE 11 で実行され、当然ながら IE 11 は Proxy など知らないのでエラーになります。

Proxy はほぼ唯一の例外のようなので、これだけであれば Proxy は使わないようにコードを書けばよいのですが、このほかにブラウザがサポートしている機能の違いもあります。たとえば fetch API は IE 11 ではサポートされていませんが、これは JavaScript のシンタックスの問題ではなくブラウザの機能の問題であるため Babel は感知しません。このため fetch メソッドを使ったコードも Babel と Polyfill を素通りして IE 11 でエラーになります。

というわけで IE 11 向けにコードを書くときは、単に Babel を使うだけでなく、IE 11 が非対応の機能を使わないように気をつける必要があります。

では、IE 11 はどの機能をサポートし、どの機能をサポートしていないのでしょうか？これを調べるのはかなり大変そうですが、幸いなことに IE 11 のみならずさまざまなブラウザがどの機能をサポートしているかをまとめた [Can I Use](http://caniuse.com) というサイトがあります。これを参照すれば一件落着 ・・・とはいきません。

Web 標準の機能は HTML 5 や CSS 3 を含め大量にあります。IE 11 がサポートしているかどうかをいちいちこのサイトでチェックしていたらコードを書くどころではありません。では、どうすればよいのでしょうか？

## Browserslist

まさにこのような状況を救ってくれるのが Browserslist 対応のツール群です。

Browserslist とは、ターゲットブラウザ (=動作対象とするブラウザ ) の一覧を記述するための設定です。package.json に書く方法や .browserslistrc ファイルに記述する方法などいくつかあります。詳細は GitHub の [browserslist リポジトリの README](http://github.com/browserslist/browserslist) を参照してください。ここでは .browserslistrc ファイルに記述する場合を例にとって説明します。実際の記述例は以下の通りです。

```text:.browserslistrc
ie 11
last 2 edge versions
last 2 chrome versions
firefox ESR
```

上記の例は IE 11 に対応、かつ Edge と Chrome については最新版とその一つ前のバージョンに対応し、firefox は ESR のみ対応ということを表現しています。

    ※ IE 11 を指定した場合、その時点でほぼすべての polyfill が取り込まれてしまうので他のブラウザの指定は実質的にほとんど効かないと思われます。

## Browserslist 対応ツール

Browserslist に対応したツールを使うとブラウザごとの違いを気にすることなく ES6+ や モダン CSS を使ってコードを書くことができます。これらのツールは browserslist に記述されたターゲットブラウザと Can I Use のブラウザの機能サポート状況テーブルを突き合わせ、ブラウザがサポートしていない ES6+ やモダン CSS の構文があれば警告で教えてくれたり、ES5 や通常の CSS に自動で変換してくれたりします。

Browserslist に対応したツールには以下のものがあります。

| ツール名   |  説明 |
| :--- | :---- |
| [eslint-plugin-compat](https://www.npmjs.com/package/eslint-plugin-compat) | ESLint プラグイン。ターゲットブラウザがサポートしていない JavaScript の機能が使われていると警告を出力する。|
| [@babel](https://babeljs.io/) | ES6+ のコードを ES5 に変換する。実際の変換は 後述の @babel/preset-env などのプラグイン (またはプラグインを集約したプリセット) を呼び出して実行する。|
| [@babel/preset-env](https://babeljs.io/docs/en/babel-preset-env) | Babel のプリセット (プリセットとは事前に定義した Babel のプラグインのセットのこと)。ターゲットブラウザを見て ES6+ のどの機能を ES5 にトランスパイルすべきかを判断し、適切なコードを出力する。実際に使うときは @babel/polyfill も必要。 |
| [PostCSS](https://www.npmjs.com/package/postcss) | モダン CSS をターゲットブラウザが読み込める形式の CSS に変換する。実際の変換は後述の postcss-preset-env などのプラグインを呼び出しておこなう。|
| [postcss-preset-env](https://www.npmjs.com/package/postcss-preset-env) | PostCSS のプラグイン。モダン CSS をターゲットブラウザが読み込める形式の CSS に変換する。 |
| [Autoprefixer](https://www.npmjs.com/package/autoprefixer)  | ターゲットブラウザにしたがって CSS に各ターゲットブラウザ固有のベンダプレフィックスを追加する。postcss-preset-env に含まれるため、そちらをインストールすれば Autoprefixer を別途インストールする必要はない。|
| [postcss-normalize](https://www.npmjs.com/package/postcss-normalize) | ブラウザごとに異なる挙動を補正するのに使われる normalize.css から各ターゲットブラウザに必要な部分を取得して出力する。  |
| [stylelint-no-unsupported-browser-feature](https://www.npmjs.com/package/stylelint-no-unsupported-browser-features) | ターゲットブラウザでサポートされていない CSS を検知する。|

## JavaScript (ES6+) のツール

### (1) eslint-plugin-compat

この ESLint のプラグインは、JavaScript でターゲットブラウザでサポートされていない機能を使っていると警告を出力してくれます。上述の fetch メソッドの例で言うと、fetch メソッドを使っている箇所があれば警告してくれます。

Visual Studio Code で ESLint プラグインを使っていれば、非サポート機能を使ったコードを書いたそばから ESLint の警告が表示されます。動作イメージについてはデモ動画が[公式ページ](https://www.npmjs.com/package/eslint-plugin-compat)にあるのでそちらをみてください。インストール方法と設定の書き方も公式ページに記載されています。

### (2) @babel、@babel/preset-env

Babel は ES6 を ES5 に変換 (トランスパイル) してくれるので IE 11 のような非モダンブラウザをターゲットとする場合でも ES6+ でコードを書くことができます。

Babel はバージョン 7 でスコープ付きパッケージ形式の名前に変わりました。バージョン 7 以降では以下のように @babel/core、@babel/cli、@babel/preset-env を --save-dev でインストールしてください。それから @babel/polyfill をインストールしますが、こちらは --save でインストールしてください。

```shell:
$ npm install --save-dev @babel/core @babel/cli @babel/preset-env
$ npm install --save @babel/polyfill
```

インストールが終わったらソースコードのエントリーポイントのファイルの先頭で @babel/polyfill をインポートします。

```JavaScript:index.js
import '@babel/polyfill';
```

最後に設定ファイル (.babelrc) をルートディレクトリ (package.json があるディレクトリ) に追加し、下記のコード例のようにプリセットとして @babel/preset-env を使うように指定します。

```JavaScript:.babelrc
{
  "presets": [
    [
      "@babel/preset-env",
      {
        useBuiltIns: "entry"
      }
    ]
  ]
}
```

@babel/preset-env はターゲットブラウザと Can I Use を参照して変換が必要な ES6+ の構文を検出し、適切なプラグインをロードして ES5 コードに変換して出力します。さらに "useBuiltIns" に "entry" を指定すればトランスパイル時に "import '@babel/polyfill'" の行が必要な polyfill だけを取り込むコードに変換されます[^2]。

こうしておけば HTML に &lt;script src="babel-polyfill.js"> のようなコードを書く必要はありません。必要な polyfill コードはすべて取り込まれています。

* なお、useBuiltIns に "usage" を指定すると @babel/polyfill を import しなくても自動的に必要な polyfill を選別して入れてくれるようですが、これはまだ**実験的**な機能で必要な polyfill を取りこぼすケースがまだあるようなので通常は使わないほうがよいでしょう[^3]。

### (3) Webpack の設定

ビルドに Webpack を使うときは、babel-loader を使って babel を呼び出します。このため、babel-loader をインストールする必要があります。

```shell:
$ npm install --save-dev babel-loader
```

JavaScriptファイルのローダーとして babel を指定するため、webpack.config.js に以下のように記述します (entry に指定したエントリファイルと output の filename は実際のファイル構成に応じて変えてください)。

```javascript:webpack.config.js
module.exports = {
  entry: ['./index.js'],
  output: {
    path: `${__dirname}/dist`,
    filename: 'bundle.js'
  },
  module: {
    rules: [
      {
        test: /\.js$/,
        exclude: /node_modules/,
        use: {
          loader: 'babel-loader'
        }
      }
    ]
  }
};
```

これで Webpack でソースコードをバンドルしたとき、babel-loader が babel を呼び出されるようになりました。このとき .babelrc も読み込まれので .browserslistrc も有効になり、ここで指定したターゲットブラウザ向けのコードが生成されます。

useBuiltIns に false を指定し (useBuiltIns のデフォルト値は false なので useBuiltIns 指定をまったく書かなくても同じ)、 entry 属性に ['@babel/polyfill', './index.js'] のように指定することもできますが、こうすると polyfill がすべて取り込まれてしまうためbrowserslist の指定は無効になってしまいます。なので、この方法は基本的には使わないほうがよいでしょう。

eslint-loader を使えば Webpack のビルドステップの一つとして ESLint を実行することも出来ます。その場合、Webpack の設定ファイルで .js ファイルのローダとして eslint-loader を追加します。このとき enforce プロパティに "pre" を指定すれば、他の loader よりも eslint-loader が先に実行されるようにできます。

```JavaScfipt:webpack.config.js
  {
    enforce: 'pre',
    test: /\.js$/,
    exclude: /node_modules/,
    loader: 'eslint-loader'
  }
```

なお、webpackは "webpack --mode development" のように production モードで実行すればマップファイルの生成等が開発に必要な情報が出力されます。開発時は通常、development モードで実行します。一方、リリース版作成時は "webpack --mode production" で実行すればマップファイルは生成されず、ファイル圧縮等の最適化処理が実行されます。

    * development と production それぞれのモードのときのビルドの指定の仕方を webpack.config.js に記述し、細かく処理を制御することもできます。

### (4) Rollup の設定

ビルドに Rollup を使うときは、rollup-plugin-babel を使います。まずプラグインをインストールします。

```shell:
$ npm install --save-dev rollup-plugin-babel@latest
```

次に rollup.config.js ファイルを以下のように設定します。input に指定したエントリファイルと output に指定した出力ファイル名は必要に応じて変えてください。

```JavaScript:rollup.config.js
import babel from 'rollup-plugin-babel';
export default {
  input: 'src/index.js',
  output: {
    file: 'dist/bundle.js',
    format: 'iife'
  },
  plugins: [ babel() ]
};
```

これでトランスパイル時に babel が使われるようになりました。こちらも当然、.babelrc が読み込まれるため、browserslist の設定を通じてターゲットブラウザに応じたコードが出力されます。

## JavaScript (ES6+) のツール

### (1) stylelint-no-unsupported-browser-feature

stylelint-no-unsupported-browser-feature は eslint-plugin-compat の CSS 版です。Browserslist で指定されたターゲットブラウザでサポートされていない CSS の記述箇所を検知し、警告してくれます。

利用するにはまず CSS の lint ツール stylelint と stylelint-no-unsupported-browser-feature をインストールします。

```shell:
npm install stylelint stylelint-no-unsupported-browser-features
```

次に .stylelintrc を次のように書いて stylelint-no-unsupported-browser-features プラグインを有効化します。

```json:.stylelintrc
{
  "plugins": [
    "stylelint-no-unsupported-browser-features"
  ],
  "rules": {
    "plugin/no-unsupported-browser-features": [true, {
      "severity": "warning"
    }]
  }
}
```

### (2) PostCSS、postcss-preset-env, Autoprefixer

PostCSS はモダン CSS に変換をかけてターゲットブラウザで解釈できるようにするツールです。babel と同様、実際の変換はプラグインを呼び出して実行します。

postss-preset-env は PostCSS のプラグインで、babel で言えば @babel/preset-env に相当するものです。browserlist に記述されたターゲットブラウザと Can I Use のブラウザサポート機能テーブルを参照し、ターゲットブラウザが解釈できないモダン CSS で書かれた CSS を従来型の CSS に変換してくれます。

利用するには PostCSS と postcss-preset-env をインストールします。

```shell:
$ npm install --save-dev postcss postcss-preset-env
```

次に PostCSS の設定ファイルを追加します。最小限の設定ファイルは以下のとおりです。

```javascript:postcss.config.js
const postcssPresetEnv = require('postcss-preset-env')

const config = () => ({
  plugins: [
    postcssPresetEnv()
  ]
})
```

postcss-preset-env には Autoprefixer も含まれており、設定ファイルで明示的に無効化の設定をしない限り、自動で実行されます。このため、ターゲットブラウザに合わせて適切なベンダープレフィックスをつける変換も併せて実行されます。

### (3) Webpack の設定

ビルドに Webpack を使う場合は CSS のローダーとして style-loader、css-loader、postcss-loader を (この順に) 指定します (ローダーについては [^4]、[^5]、[^6]などを読むとわかりやすいです)。postcss-loader が PostCSS を実行し、その結果 postcss-preset-env が実行されます。

利用するにはまず上記 3 つのローダーをインストールします。

```shell:
$ npm install --save-dev css-loader style-loader postcss-loader
```

次に webpack.config.js のローダーの設定部分を次のように記述します。

```javascript:postcss.config.js
  test: /\.css$/,
  exclude: /node_modules/,
  use: [
    {
      loader: 'style-loader',
    },
    {
      loader: 'css-loader',
      options: {importLoaders: 1},
    },
    {
      loader: 'postcss-loader',
      options: {
        config: {
          path: __dirname + '/postcss.config.js'
        }
      },
    },
  ]
```

上記の style-loader を使う方法だと自動的に &lt;link> タグが生成されて CSS がブラウザにロードされますが、そうではなく CSS をファイルに書き出したいときは mini-css-extract-plugin を使います。コード例は Webpack 公式ホームページの [MiniCssExtractPlugin](https://webpack.js.org/plugins/mini-css-extract-plugin/)のページを参照してください。

MiniCssExtractPlugin の説明ページにも書いてありますが、上記に加えてさらに CSS を圧縮するときは、optimize-css-assets-webpack-pluginを使います。詳しくは MiniCssExtractPlugin の説明ページを参照してください。

### (4) Rollup の設定

Rollup を使った場合、preset によって選択的に CSS を取り込むのは難しいようです。現時点 (2019年2月) ではいろいろやってみた結果、

## React を使う場合

React を使う場合は、JSX をトランスパイルできるように @babel/preset-react も追加でインストールします。

```shell:
$ npm install --save-dev @babel/preset-react
# $$$ 設定方法をあとで記述
```

## まとめ

以上をすべて実行したときのファイル一式は以下にあります。本記事のタイトルは「IE 11 対応の ES6 コードを書くための準備」ですが、要は目的とするターゲットブラウザに合わせたコードを出力する方法の説明なので、browserslist の設定次第で IE 9 にも対応したり、逆にモダンブラウザだけに対応するなど、さまざまなケースに使えます。

$$$ transform-runtime も追加したほうがよい？
$$$ ミニマル設定のプロジェクトを作る

---
## 参考文献

[^1]: Babel の公式ページの Learn ES2015 の Proxy の節(https://Babeljs.io/docs/en/learn#proxies) の末尾に "Unsupported Feature" と書かれています。

[^2]: [恐竜に教える現代のCSS – Part 1](https://postd.cc/actualize-networkmodern-css-explained-for-dinosaurs/)

[^3]: [Babel7.x時代のpolyfillの設定方法とuseBuiltInsの仕組み] (https://aloerina01.github.io/blog/2018-11-29-1)

[^4]: [最新版で学ぶwebpack 4入門 – JavaScriptのモジュールバンドラ](https://ics.media/entry/12140)

[^5]: [最新版で学ぶwebpack 4入門 – スタイルシート(CSSやSass)を取り込む方法](https://ics.media/entry/17376)

[^6]: [最新版で学ぶwebpack 4入門 – Babel 7でES2018環境の構築(React, Vue, Three.js, jQueryのサンプル付き)](https://ics.media/entry/16028)

[^7]: [Webpack v3 → v4移行パッケージ対照表](https://qiita.com/shimarin/items/17707fa575744ca0bd89#fn1)
