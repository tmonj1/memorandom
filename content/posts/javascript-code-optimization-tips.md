---
title: "JavaScript パフォーマンス Tips"
date: 2019-01-24T11:40:33+09:00
draft: false
---

以下の内容は、下記の Web 記事の簡単な要約です。

出典: [How JavaScript works: inside the V8 engine + 5 tips on how to write optimized code] (https://blog.sessionstackcom/how-javascript-works-inside-the-v8-engine-5-tips-on-how-to-write-optimized-code-ac089e62b12e)

## パフォーマンス Tips のサマリ (全 5 件)

1. オブジェクトプロパティの順序

    オブジェクトにプロパティを追加するときは、常に同じ順番でプロパティを初期化する。下記のコード例のように同一クラスのオブジェクトでもプロパティの追加順番が異なると、性能が劣化する。

    ```JavaScript
    function Point(x, y) {
        this.x = x;
        this.y = y;
    }
    var p1 = new Point(1, 2);
    p1.a = 5;
    p1.b = 6;
    var p2 = new Point(3, 4);
    p2.b = 7;
    p2.a = 8;
    ```

1. 動的プロパティ

    あとからプロパティを追加するのはコストが高い。プロパティはすべてコンストラクタ内で追加しておく。

1. メソッド

    同じコードを何回も実行させるほうが、異なるメソッドを実行させるより速い
    →   つまり、共通化できるメソッドはきちんと共通化したほうが高速になる可能性が高い。
        コード行数の多いメソッドは、保守性が落ちるだけでなく性能も劣化する。

1. 配列

    不連続な配列 (sparse array) は使用しない。配列は常にインデックスが0から始まる連続領域で確保・使用する。JavaScriptの配列は不連続なインデックスでも使えるが、実質 hash table となるので性能が劣化する。またサイズの大きな配列を使うとき、事前に領域を確保するのではなく、push() で随時追加するほうがよい (←普通の言語とは逆）。

1. タグ付きの値

    V8 ではオブジェクトと数値を 32 bit で表現している。そのうち 1 bit は値がオブジェクトか数値かのは別に使用している。整数値が 31 bit を超えると boxing が発生して性能が劣化するため、できるだけ 31 bit に収まる範囲で使うのが良い。

## V8 エンジンについて

* V8 は Google Chrome に搭載されている JavaScript エンジン。Node.js も V8 を使っている。

* v8 version 5.9 (2017年5月15日リリース) 以降とそれ以前とで v8 の JavaScript 実行エンジンが使用するコンパイラが変わっており、性能もだいぶ向上した模様。

    * v5.8 以前: full-codegen (baseline compiler) + Crankshaft (optimized JIT compiler)
    * v5.9 以降: Ignition (interpreter) + TurboFan (optimized JIT compiler)
