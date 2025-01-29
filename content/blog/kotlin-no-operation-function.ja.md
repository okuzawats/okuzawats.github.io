---
title: "[Kotlin] 「何もしない」関数をいい感じに書きたい"
description: "Kotlinで「何もしない関数」をいい感じに定義するための試みについて書きました。"
date: 2022-01-22T22:06:36+09:00
draft: false
categories: ["Kotlin", "プログラミング"]
---

Kotlinで「何もしない」関数をきれいに書きたい、というtrialです。

## 返り値を持たない関数

返り値を持たない関数は、通常、何らかの副作用を持つ処理を実行する関数です。例えば、特定の文字列を標準出力する関数などですね。以下のようなコードです。

```kotlin
fun doSomething() {
  println("Hello World")
}
```

Kotlinでは、このような関数は `Unit` 型を返します。Kotlinにおいては、「返り値を持たない関数」というのは、実は「返り値が `Unit` である関数」であると言えそうです。

```kotlin
fun main() {
  val value = doSomething()
  value is Unit // true
}
```

そのため、最初のサンプルコードに示した `doSomething` 関数の正確なシグネチャは以下のようになります。もちろん、返り値の型が `Unit` の場合には、通常関数の返り値の型を省略します。

```kotlin
fun doSomething(): Unit {
  println("Hello World")
}
```

## 何もしない関数

ところで、何もしない関数を定義することがあります。

例えば、こういう `interface` があったとします。

```kotlin
interface AwesomeInterface {
  fun doSomething()
}
```

この `interface` を実装するけど、実際には何もしない、という場合があります（「[nullオブジェクトパターン](https://okuzawats.com/blog/null-object-pattern/)」というやつでしょうか）。こんな感じです。

```kotlin
fun main() {
  val awesome: AwesomeInterface = object : AwesomeInterface {
    override fun doSomething() {
      // NOP
    }
  }

  awesome.doSomething() // do nothing
}
```

こういう時は `// 何もしない` とか `// NOP` とか `// NOOP` とかのコメントを書いたブロックを置くことが多いと思います。ここではもうひと工夫してみます。「返り値を持たない関数」で書いたように、こういった関数は、実は `Unit` を返しているのでした。ということは、ブロックの代わりに、関数本体として `Unit` を返せばよいということになります。以下のコードです。

```kotlin
  val awesome: AwesomeInterface = object : AwesomeInterface {
    override fun doSomething() = Unit
  }
```

このコードはKotlinのコードとして100%正しいですが、Kotlinに習熟していない人にとっては意図を理解しにくい記法であるとも思います。

そこでさらにひと工夫してみます。 `Unit` 型に `NOP` という名前でtypealiasを付けてみます。

```kotlin
typealias NOP = Unit
```

そうすると、先ほどのコードを以下のように修正することができます。

```kotlin
  val awesome: AwesomeInterface = object : AwesomeInterface {
    override fun doSomething() = NOP
  }
```

このように書けば、 `doSomething` 関数が「何もしない」関数であることをコードで表現できる...と思うのですがどうでしょうか。typealiasを付けたり、関数本体を返したりといった細かいテクニックを駆使しているので、Kotlinに習熟していない人にとってはもしかしたら逆に馴染みにくい記法になっている、という可能性はあります。

そうだとすると、一周回ってコメントで愚直に書くのが良いのかもしれません。

```kotlin
  val awesome: AwesomeInterface = object : AwesomeInterface {
    override fun doSomething() {
      // NOP
    }
  }
```

自分は `fun doSomething() = NOP` と書くのが好みなので、自分の趣味100%で書くプロジェクトであれば、 `fun doSomething() = NOP` で書きます。そうでない時は...コメントで書くかもしれません。

## Reference
1. [Unit - Kotlin Programming Language](https://kotlinlang.org/api/latest/jvm/stdlib/kotlin/-unit/)（最終アクセス日：2022年1月22日）
