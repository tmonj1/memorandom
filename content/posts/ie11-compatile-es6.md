---
title: "IE 11 対応の ES6 コードを書くための準備"
date: 2018-12-30T23:40:33+09:00
draft: false
---

## 何が問題か?

ES6 で書いても Babel でトランスパイルし、polyfill を使えば IE 11 でも実行できます。多くの場合は。

ただ残念ながらこれだけでは動かないこともあります。たとえば、ES6 の新機能である Proxy を使っていたりすると IE では動作しません。なぜなら、Babel の polyfill は Proxy には対応していないからです[^1]。

Proxy を使っている箇所はそのまま Babel と polyfill を素通りして IE 11 で実行され、当然ながら IE 11 は Proxy など知らないのでエラーになります。

なので IE 11 向けにコードを書くときは、単に Babel を使うだけでなく、IE 11 が非対応の機能を使わないように気をつける必要があります。

では、IE 11 はどの機能をサポートし、どの機能をサポートしていないのでしょうか？これを調べるのはかなり大変そうですが、幸いなことに IE 11 のみならずさまざまなブラウザがどの機能をサポートしているかをまとめた [Can I Use](http://caniuse.com) というサイトがあります。これを参照すれば一件落着 ・・・とはいきません。

Web 標準の機能は HTML 5 や CSS 3 を含め大量にあります。IE 11 がサポートしているかどうかをいちいちこのサイトでチェックしていたらコードを書くどころではありません。では、どうすればよいのでしょうか？

## Browserslist

まさにこのような状況を救ってくれるのが Browserslist 対応のツール群です。

Browserslist とは、ターゲットブラウザ (=動作対象とするブラウザ ) の一覧を記述するための設定です。package.json に書く方法や .browserslistrc ファイルに記述する方法などいくつかあります。詳細は GitHub の [browserslist リポジトリの README](http://github.com/browserslist/browserslist) を参照してください。ここでは .browserslistrc ファイルに記述することにします。実際の記述例は以下の通りです。

```text:.browserslistrc
ie 11
ast 2 edge versions
last 2 chrome versions
last 2 safari versions
```

上記の例は IE 11 に対応、かつ Edge、Chrome、Safari は最新版とその一つ前のバージョンに対応ということを表現しています。

## Browserslist 対応ツール

Browserlist に対応したツールを使うと、ターゲットブラウザのサポート状況を気にすることなく ES 6 や モダン CSS を使ってコードを書くことができます。これらのツールは Can I Use を参照することで browserslist に記述した各ターゲットブラウザの機能サポート状況を取得し、ES 6 や CSS のコードを適切な形式に変換してくれます。それができない場合でも、非サポート機能を検知し、警告してくれます。

Browserslist に対応したツールとしは以下のものがあります。

| ツール名   |  説明 |
| :--- | :---- |
| [eslint-plugin-compat](https://www.npmjs.com/package/eslint-plugin-compat) | ESLint プラグイン。ターゲットブラウザでサポートされてない JavaScript の機能を検知する。|
| [Babel](https://babeljs.io) | ES6 から ES5 へのトランスパイラ。ターゲットブラウザを見て ES6 のどの機能を ES5 にトランスパイルすべきかを判断する。 |
| [postcss-preset-env](https://www.npmjs.com/package/postcss-preset-env) | モダン CSS をターゲットブラウザが読み込める形式の CSS に変換する。 |
| [Autoprefixer](https://www.npmjs.com/package/autoprefixer)  | ターゲットブラウザにしたがって CSS に各ターゲットブラウザ固有のベンダプレフィックスを追加する。postcss-preset-env に含まれるため、そちらをインストールすれば Autoprefixer を別途インストールする必要はない。|
| [postcss-normalize](https://www.npmjs.com/package/postcss-normalize) | normalize.css から各ターゲットブラウザに必要な部分を取得する。  |
| [stylelint-no-unsupported-browser-feature](https://www.npmjs.com/package/stylelint-no-unsupported-browser-features) | ターゲットブラウザでサポートされていない CSS を検知する。|

### (1) eslint-plugin-compat

まず eslint-plugin-compat を使いましょう。この ESLint のプラグインは、JavaScript でターゲットブラウザでサポートされていない機能を使っていると警告を出力してくれます。

Visual Studio Code で ESLint プラグインを使っていれば、非サポート機能を使ったコードを書いたそばから ESLint の警告が表示されます。動作イメージについては[デモ動画](https://www.npmjs.com/package/eslint-plugin-compat)が公式ページにあるので、そちらをみてください。

### (2) Babel

つぎに Babel をセットアップしましょう。Babel は ES6 を ES5 に変換 (トランスパイル) してくれるので IE 11 のような非モダンブラウザをターゲットとする場合でも ES 6 でコードを書くことができます。

Babel はバージョン 7 でスコープ付きパッケージ形式の名前に変わりました。スコープなしの古い形式で説明したものがたくさん出回っているので注意してください。バージョン 7 以降では以下のように "@" が入る名称でインストールしてください。

```shell:
$ npm install --save-dev @babel/core @babel/cli @babel/preset-env
npm install --save @babel/polyfill
```

インストールが終わったら設定ファイル (.babelrc) を追加します。最小限の設定は以下のとおりです。"useBuiltIns" に "usage" を指定すれば必要な polyfill だけを取り込んでトランスパイルしてくれるようになります。

```JavaScript:.babelrc
{
  "presets": [
    [
      "@babel/preset-env",
      {
        useBuiltIns: "usage"
      }
    ]
  ]
}
```
### (3) PostCSS プラグイン

ここからは JavaScript ではなく CSS 向けの設定です。CSS 用の変換・チェックツールは PostCSS のプラグインとして提供されているので、まず PostCSS をインストールします。

```shell:
$ npm install --save-dev postcss
```

次に postcss-preset-env をインストールします。postcss-preset-env を使えばモダン CSS で記述しても非モダンブラウザ向けに自動で変換してくれます。

```shell:
$ npm install --save-dev postcss-preset-env
```

最後に PostCSS の設定ファイルを追加します。最小限の設定ファイルは以下のとおりです。

```JSON
```

## まとめ

以上をすべて実行したときのファイル一式は以下にあります。本記事のタイトルは「IE 11 対応の ES6 コードを書くための準備」ですが、要は目的とするターゲットブラウザに合わせたコードを出力する方法の説明なので、browserslist の設定次第で IE 9 にも対応したり、逆にモダンブラウザだけに対応するなど、さまざまなケースに使えます。

$$$ transform-runtime も追加したほうがよい？
$$$ ミニマル設定のプロジェクトを作る

---
## 参考文献

[^1]: Babel の公式ページの Learn ES2015 の Proxy の節(https://Babeljs.io/docs/en/learn#proxies) の末尾に "Unsupported Feature" と書かれています。

[^2]: 恐竜に教える現代のCSS – Part 1 (https://postd.cc/actualize-networkmodern-css-explained-for-dinosaurs/)
