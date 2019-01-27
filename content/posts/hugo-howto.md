---
title: "Hugo Howto"
date: 2019-01-25T07:31:49+09:00
draft: true
---

* hasCJKLanguage = true にしないとサマリが長大になる
* langugeCode が未だに謎
  * theme の partials で <html xmlns="//www.w3.org/1999/xhtml" xml:lang="en" lang="en"> とベタ打ちされていた。
    layouts/partialsにファイルをコピーして来て "ja" に変えたら日本語になった。