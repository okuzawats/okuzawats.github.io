---
title: "短絡評価を乱用しないでください🙅‍♂️"
date: 2023-03-16T22:48:54+09:00
description: "短絡評価を乱用すると、思いがけない不具合を生みます。"
categories: ["Design", "プログラミング"]
draft: false
---

多くのプログラミング言語では、論理演算は短絡評価されます。Kotlinの文法であれば、 `a && b` で `a` がfalseであれば `b` は評価されず、 `a || b` で `a` がtrueであれば `b` は評価されないということです（無駄なので）。

これを利用して、ハック的なコードを書くことが可能です。

わかりやすい例を提示します。以下のコードでは、 `someBool()` がtrueを返した時は `some()` のブロック内の処理は短絡評価されるため、 `doSomething()` は評価されません。一方、 `someBool()` がfalseを返した時は `doSomething()` が評価されます。

```kotlin
fun some() {
  someBool() || doSomething()
}

fun someBool(): Boolean = Random.nextBoolean()

fun doSomething(): Boolean {
    println("do something here")
    return true
}
```

この `&&` `||` の短絡評価を利用すると、 `someBool()` がtrue / falseの時にだけ特定の処理を行う、という処理を書くことができるわけですが、あまり行儀が良くないので[^1]、基本的にはやめましょう。

今回の例はまだわかりやすいですが、 `if (someCondition() || anotherCondition()) {}` のように、 `if` の条件式内で何らかの副作用が発生するような短絡評価の乱用は、厳に慎むべきでしょう。

---

Kotlinには、 `and` 演算子と `or` 演算子が存在します。これらの演算子はそれぞれ `&&` 演算子と `||` 演算子と似ていますが、 `and` 演算子と `or` 演算子は短絡評価されず、完全評価されます。すなわち、演算子の左側の値がtrueだろうがfalseだろうが、右側の値も評価される、ということです。

こちらはこちらで、無駄に処理が行われてしまうことにつながるため、注意が必要です。

[^1]: 個人の見解です。
