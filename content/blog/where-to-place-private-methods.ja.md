---
title: "privateメソッドをどこに書くか"
date: 2024-10-30T09:26:00+09:00
description: "privateメソッドをどこに書くのか、考えます。"
categories: ["プログラミング"]
draft: true
---

privateメソッドをソースコードのどこに書くべきか？ということを考えてみます。これはソースコードのファイルにおける位置の話です。プログラミング言語は説明に都合の良いように使える擬似言語を用いますので、適宜実在するプログラミング言語に読み替えていただくと良いかと思います。

### 下の方にまとめて書く

クラスやファイルの下の方にまとめて書くことがあると思います。例えばこんな感じです。

```
public function a() {
  x()
  y()
  ...
}

public function b() {
  x()
  z()
  ...
}

public function c() {
  z()
  ...
}

// public関数が続く

private function x() {}

private function y() {}

private function z() {}
```

この場合、publicやprivateといったコンテキストに応じて、メソッドが並びます。publicな振る舞いが分かりやすことがメリットでしょうか。一方で、 `function a` や `function b` のソースコードを読む時に `function x` `function y` `function z` にジャンプすることが若干面倒です。

### 使う場所の近くに書く

TODO
