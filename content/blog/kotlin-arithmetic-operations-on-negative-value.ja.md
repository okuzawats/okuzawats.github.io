---
title: "[Kotlin] 負数に対する除算の商と剰余"
date: 2022-02-20T22:05:00+09:00
description: "Kotlinでの負数に対する除算の商と剰余についての記事です。"
categories: ["Kotlin"]
draft: false
---

AtCoder Beginner Contest 239のB問題を解いていてWAになり、解説を読んでいたら勉強になったのでメモ。負の値に対する除算の商と剰余について。

> 実は負数の除算はいくつか定義があり、プログラム言語によって定義が異なる という大きな問題があります。詳しい話は以下で説明しますが、例えば −24 を割られる数、 10 を割る数として整数除算した時の答えは言語によって −2 になったり −3 になったりします。
> [解説 - デンソークリエイトプログラミングコンテスト2022（AtCoder Beginner Contest 239）](https://atcoder.jp/contests/abc239/editorial/3390)

Kotlinの（Int型やLong型に対する）除算のドキュメントを探すと、例えばInt型なら `div` 関数を見れば良さそうでしょうか（Kotlinの算術演算子は `operator fun` でオーバーロードされる）。

- [div - Kotlin Programming Language](https://kotlinlang.org/api/latest/jvm/stdlib/kotlin/-int/div.html)

ここには、0に近い方に丸めると記載されてます。

> Divides this value by the other value, truncating the result to an integer that is closer to zero.

例えば -1 に対して 2 で除算を行うと、（ -0.5 が 0 に近い方に丸められて） 0 となります。同様に -3 に対して 2 で除算を行うと、（ -1.5 が 0 に近い方に丸められて） -1 となります。

```kotlin
fun main() {
  println(-1 / 2) // 0, not -1
  println(-2 / 2) // -1
  println(-3 / 2) // -1, not -2
}
```

剰余がどうなるかというと、上記の除算の結果と整合する自然な値になります。

```kotlin
fun main() {
  println(-1 % 2) // -1
  println(-2 % 2) // 0
  println(-3 % 2) // -1
}
```

負の除算については注意して扱わないと思わぬバグを生みそうです。数年来プログラミングをしてきましたが、初歩的なところに理解のあやふやな知識があることに気付けて良かったです。

## Reference
1. [解説 - デンソークリエイトプログラミングコンテスト2022（AtCoder Beginner Contest 239）](https://atcoder.jp/contests/abc239/editorial/3390)（最終アクセス日：2022年2月20日）
2. [div - Kotlin Programming Language](https://kotlinlang.org/api/latest/jvm/stdlib/kotlin/-int/div.html)（最終アクセス日：2022年2月20日）
