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

では、IE 11 はどの機能をサポートし、どの機能をサポートしていないのでしょうか？これを調べるのはかなり大変そうですが、幸いなことに IE 11 のみならずさまざまなブラウザがどの機能をサポートしているかをまとめた [Can I Use](http://caniuse.com) というサイトがあります。これを参照すれば一件落着 ・・・とはいきません。

Web 標準の機能は HTML 5 や CSS 3 を含め大量にあります。IE 11 がサポートしているかどうかをいちいちこのサイトでチェックしていたらコードを書くどころではありません。では、どうすればよいのでしょうか？

## Browserslist

まさにこのような状況を救ってくれるのが Browserslist 対応のツール群です。

Browserslist とは、ターゲットブラウザ (=動作対象とするブラウザ ) の一覧を記述するための設定です。package.json に書く方法や .browserslistrc ファイルに記述する方法などいくつかあります。詳細は GitHub の [browserslist リポジトリの README](http://github.com/browserslist/browserslist) を参照してください。ここでは .browserslistrc ファイルに記述する場合を例にとって説明します。実際の記述例は以下の通りです。

```text:.browserslistrc
ie 11
last 2 edge versions
last 2 chrome versions
firefox ESR
```

上記の例は IE 11 に対応、かつ Edge と Chrome については最新版とその一つ前のバージョンに対応し、firefox は ESR のみ対応ということを表現しています。

## Browserslist 対応ツール

Browserslist に対応したツールを使うとブラウザごとの違いを気にすることなく ES6+ や モダン CSS を使ってコードを書くことができます。これらのツールは Can I Use を参照して browserslist に記述された各ターゲットブラウザの機能サポート状況を取得し、ES6+ や CSS のコードを適切な形式に変換してくれます。それができない場合でも、非サポート機能を検知し、警告してくれます。

Browserslist に対応したツールには以下のものがあります。

| ツール名   |  説明 |
| :--- | :---- |
| [eslint-plugin-compat](https://www.npmjs.com/package/eslint-plugin-compat) | ESLint プラグイン。ターゲットブラウザがサポートしていない JavaScript の機能が使われていると警告を出力する。|
| [@babel/preset-env](https://babeljs.io/docs/en/babel-preset-env) | Babel のプリセット (プリセットとは事前に定義した Babel のプラグインのセットのこと)。ターゲットブラウザを見て ES6+ のどの機能を ES5 にトランスパイルすべきかを判断し、適切なコードを出力する。実際に使うときは @babel/polyfill も必要。 |
| [postcss-preset-env](https://www.npmjs.com/package/postcss-preset-env) | モダン CSS をターゲットブラウザが読み込める形式の CSS に変換する。 |
| [Autoprefixer](https://www.npmjs.com/package/autoprefixer)  | ターゲットブラウザにしたがって CSS に各ターゲットブラウザ固有のベンダプレフィックスを追加する。postcss-preset-env に含まれるため、そちらをインストールすれば Autoprefixer を別途インストールする必要はない。|
| [postcss-normalize](https://www.npmjs.com/package/postcss-normalize) | normalize.css から各ターゲットブラウザに必要な部分を取得する。  |
| [stylelint-no-unsupported-browser-feature](https://www.npmjs.com/package/stylelint-no-unsupported-browser-features) | ターゲットブラウザでサポートされていない CSS を検知する。|

### (1) eslint-plugin-compat

まず eslint-plugin-compat を使いましょう。この ESLint のプラグインは、JavaScript でターゲットブラウザでサポートされていない機能を使っていると警告を出力してくれます。上述の例で言うと、fetch メソッドを使っている箇所があれば警告してくれます。

Visual Studio Code で ESLint プラグインを使っていれば、非サポート機能を使ったコードを書いたそばから ESLint の警告が表示されます。動作イメージについてはデモ動画が[公式ページ](https://www.npmjs.com/package/eslint-plugin-compat)にあるのでそちらをみてください。インストール方法と設定の書き方も公式ページに記載されています。

### (2) Babel

つぎに Babel をセットアップしましょう。Babel は ES6 を ES5 に変換 (トランスパイル) してくれるので IE 11 のような非モダンブラウザをターゲットとする場合でも ES6+ でコードを書くことができます。

Babel はバージョン 7 でスコープ付きパッケージ形式の名前に変わりました。バージョン 7 以降では以下のように @babel/core、@babel/cli、@babel/preset-env を --save-dev でインストールしてください。それから @babel/polyfill をインストールしますが、こちらは --save でインストールしてください。

```shell:
$ npm install --save-dev @babel/core @babel/cli @babel/preset-env
$ npm install --save @babel/polyfill
```

インストールが終わったらソースコードのエントリーポイントのファイルの先頭で @babel/polyfill をインポートします。

```JavaScript:index.js
import '@babel/polyfill';
```

最後に設定ファイル (.babelrc) を追加します。最小限の設定は以下のとおりです。"useBuiltIns" に "entry" を指定すればトランスパイル時に "import '@babel/polyfill'" の行が必要な polyfill だけを取り込むコードに変換されます[^2]。

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

こうしておけば HTML に &lt;script src=babel-polyfill> のようなコードを書く必要はありません。必要な polyfill コードはすべて取り込まれています。

useBuiltIns に "usage" を指定すると @babel/polyfill を import しなくても自動的に必要な polyfill を選別して入れてくれるようですが、これはまだ**実験的**な機能で必要な polyfill を取りこぼすケースがまだあるようなので通常は使わないほうがよいでしょう[^3]。

### (3) Webpack 

Webpack を使うときは、babel-loader を使って babel を呼び出します。このため、babel-loader をインストールする必要があります。

```shell:
$ npm install --save-dev babel-loader
```

jsファイルのローダーとして babel を指定するため、webpack.config.js に以下のように記述します (entry に指定したエントリファイルと output の filename は実際のファイル構成に応じて変えてください)。

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

これで Webpack でソースコードをバンドルしたとき、babel-loader が babel を呼び出されるようになりました。このとき .babelrc も読み込まれので .browserslistrc も有効になり、ここで指定したターゲットブラウザ向けのコードが生成されます。

.babelrc で useBuiltIns に entry を指定した場合、webpack.config.js の entry プロパティに @babel/polyfill を指定してはいけません。これを指定すると polyfill 全体が取り込まれてしまうため、せっかく browserslist で指定したターゲットブラウザの指定が無効になってしまいます。

### (5) Rollup

Rollup でバンドルするときは、rollup-plugin-babel を使います。まずプラグインをインストールします。

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

これでトランスパイル時に babel が使われるようになりました。こちらも当然、.babelrc が読み込まれるため、browserslist の設定を通じてターゲットブラウザに応じたコードが出力されます。

### (6) CSS

ここからは JavaScript ではなく CSS 向けの設定です。CSS 用の変換・チェックツールは PostCSS のプラグインとして提供されているので、まず PostCSS をインストールします。

```shell:
$ npm install --save-dev postcss
```

次に postcss-preset-env をインストールします。postcss-preset-env を使えばモダン CSS で記述しても非モダンブラウザ向けに自動で変換してくれます (以前は postcss-cssnext が使われていましたが、)。

```shell:
$ npm install --save-dev postcss-preset-env
```

最後に PostCSS の設定ファイルを追加します。最小限の設定ファイルは以下のとおりです。

```JSON
```


## React を使う場合

React を使う場合は、JSX をトランスパイルできるように @babel/preset-react も追加でインストールします。

```shell:
$ npm install --save-dev @babel/preset-react
# $$$ 設定方法をあとで記述
```

## まとめ

以上をすべて実行したときのファイル一式は以下にあります。本記事のタイトルは「IE 11 対応の ES6 コードを書くための準備」ですが、要は目的とするターゲットブラウザに合わせたコードを出力する方法の説明なので、browserslist の設定次第で IE 9 にも対応したり、逆にモダンブラウザだけに対応するなど、さまざまなケースに使えます。

$$$ transform-runtime も追加したほうがよい？
$$$ ミニマル設定のプロジェクトを作る

---
## 参考文献

[^1]: Babel の公式ページの Learn ES2015 の Proxy の節(https://Babeljs.io/docs/en/learn#proxies) の末尾に "Unsupported Feature" と書かれています。

[^2]: [恐竜に教える現代のCSS – Part 1](https://postd.cc/actualize-networkmodern-css-explained-for-dinosaurs/) 

[^3]: [Babel7.x時代のpolyfillの設定方法とuseBuiltInsの仕組み] (https://aloerina01.github.io/blog/2018-11-29-1)

[^4]: [ES2015&#40;ES6)+webpack+babel-loaderで開発環境を構築](https://qiita.com/d-dai/items/23410fa7d84fb1020ea8)