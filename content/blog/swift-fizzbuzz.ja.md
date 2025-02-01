---
title: "SwiftでFizzBuzz"
date: 2023-02-26T22:27:00+09:00
description: "SwiftでFizzBuzzを書いてみるテスト。"
categories: ["Swift", "GitHub Actions", "プログラミング"]
draft: false
---

Swiftに入門していきたいので、FizzBuzzを書いて肩慣らししていきます。FizzBuzzのルールについては[Wikipedia](https://ja.wikipedia.org/wiki/Fizz_Buzz)を参照してください。

## FizzBuzz

まずは普通に `switch` で書こうとしたわけですが、Swiftの `switch` は値を返す式ではなく、値を返さない文です。そのため、Fizzを返すパターン、Buzzを返すパターン、FizzBuzzを返すパターン、それ以外のパターンのすべてで `return` を書く必要があることに気付きます。悪くないですが、 `return` をもう少し減らしたいです。

ということで、`var` で文字列を宣言し、3で割り切れる場合は `Fizz` を、5で割り切れる場合は `Buzz` を文字列に追加していくアプローチを取ります。3で割り切れ、かつ5でも割り切れる場合は、`Fizz` と `Buzz` の両方が追加されて `FizzBuzz` となる仕組みです。最初に書いたものがこれです。

```swift
func fizzbuzz(n:Int) -> String {
  var s = ""
  if n % 3 == 0 { s += "Fizz" }
  if n % 5 == 0 { s += "Buzz" }
  return s.isEmpty ? String(n) : s
}

(1...100)
  .map { fizzbuzz(n:$0) }
  .forEach { print($0) }
```

悪くないです。

ここから文字列を削る作業を始めていきます（この文字列を削る作業はあくまで趣味の領域ですが、言語について調べることになるのでいい勉強になると思っています）。

まず `n % 3 == 0` としている箇所ですが、整数同士の剰余は常に0以上であるという性質を利用すると、`n % 3 < 1` と書いても良いことに気が付きます。5についても同様です。

```swift
func fizzbuzz(n:Int) -> String {
  var s = ""
  if n % 3 < 1 { s += "Fizz" }
  if n % 5 < 1 { s += "Buzz" }
  return s.isEmpty ? String(n) : s
}

// 略
```

`String(n)` は、文字列補完の記法を用いて `"\(n)"` と書いた方が短いです。

```swift
func fizzbuzz(n:Int) -> String {
  var s = ""
  if n % 3 < 1 { s += "Fizz" }
  if n % 5 < 1 { s += "Buzz" }
  return s.isEmpty ? "\(n)" : s
}

// 略
```

さらに、 `s.isEmpty` は空文字 `""` との比較で十分です（ `s.isEmpty` の方が読みやすいですが、長いです）。

```swift
func fizzbuzz(n:Int) -> String {
  var s = ""
  if n % 3 < 1 { s += "Fizz" }
  if n % 5 < 1 { s += "Buzz" }
  return s == "" ? "\(n)" : s
}

// 略
```

丁寧に `fizzbuzz` 関数を定義するのはやめて、ループの中ですべての処理を書くようにしてしまいましょう。

```swift
(1...100).forEach {
  var s = ""
  if $0 % 3 < 1 { s += "Fizz" }
  if $0 % 5 < 1 { s += "Buzz" }
  print(s != "" ? s : "\($0)")
}
```

`forEach` をやめて `for` を使えば、簡略引数名 `$0` の代わりにインデックスを使うことで文字数を少し減らせませす。

```swift
for i in 1...100 {
  var s = ""
  if i % 3 < 1 { s += "Fizz" }
  if i % 5 < 1 { s += "Buzz" }
  print(s != "" ? s : "\(i)")
}
```

自分にはこれ以上削れません。実行してみます。

```shell
% swift -v
Apple Swift version 5.7.2 (swiftlang-5.7.2.135.5 clang-1400.0.29.51)
% swift fizzbuzz.swift
1
2
Fizz
4
Buzz
Fizz
7
8
Fizz
Buzz
11
Fizz
13
14
FizzBuzz
16
以下略
```

## GitHub Actions

ついでに、GitHub Actionsで今回書いたFizzBuzzを実行してみます。GitHub Docsに、「特定のバージョンのSwiftを使用するには、[fwal/setup-swift](https://github.com/marketplace/actions/setup-swift)を使え」（Ref.2）と書いてあるので、使います。

```yaml
name: FizzBuzz

on:
  push:
    branches:
      - main

jobs:
  fizzbuzz:
    runs-on: macos-latest
    steps:
      - uses: actions/checkout@v3
      - uses: fwal/setup-swift@da0e3e04b5e3e15dbc3861bd835ad9f0afe56296
        with:
          swift-version: "5.7.2"
      - name: do fizzbuzz
        run: swift fizzbuzz.swift
```

プロジェクト全体はGitHubの以下のリポジトリに公開してあります。

- [okuzawats/swift-fizzbuzz: SwiftでFizzBuzzを書いてみるテスト](https://github.com/okuzawats/swift-fizzbuzz)

## Reference

1. [Fizz Buzz - Wikipedia](https://ja.wikipedia.org/wiki/Fizz_Buzz)（最終アクセス日：2023/02/26）
2. [Swift のビルドとテスト - GitHub Docs](https://docs.github.com/ja/actions/automating-builds-and-tests/building-and-testing-swift)（最終アクセス日：2023/02/26）
