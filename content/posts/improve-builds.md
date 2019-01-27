# ビルド改善

## TODOs

1. production build 対応 [0.5]  
  複合コントロールを minify かつ map ファイルなしでビルドするスクリプトを package.json に追加。
1. 全種類のビルドを実行するスクリプトを追加  [1.0]  
  browserslist 対応の検証で多数のビルドを繰り返すことになるので、作業効率を上げるため定義しておく (NextG、複合)。
1. browserlists 対応 []  
  ターゲットブラウザを明確に指定してビルドする (複合)。
  - eslint-plugin-compat 対応
  - ES6 トランスパイル対応 (babel-preset-env)
  - モダン CSS 対応 (Autoprefixer, PostCSS)
1. package.json 整理 []  
  - webpack等のバージョンアップ (複合、SmartDevice)
  - 不要な依存パッケージの削除 (NextG、複合、SmartDevice)

## 事前調査
1. 複合コントロールをターゲットブラウザ指定でビルドしたとき、依存パッケージはどうビルドされる？
1. rollup と webpack のときの ES6 と CSS のトランスパイルの全体像を明らかにする。

## メモ
1. 本当は SmartDevice も browserlist 対応したほうがよいかもしれない。ほとんどトランスパイルされないかもしれないが、少しでもトランスパイルされるなら。
