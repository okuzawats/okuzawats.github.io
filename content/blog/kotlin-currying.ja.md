---
title: "[Kotlin] はじめてのカリー化"
description: "Kotlinでカリー化にチャレンジします。"
date: 2022-02-10T23:14:00+09:00
draft: false
categories: ["Kotlin"]
---

カリー化とは、

> 複数の引数をとる関数を、引数が「もとの関数の最初の引数」で戻り値が「もとの関数の残りの引数を取り結果を返す関数」であるような関数にすること（あるいはその関数のこと）である。（Wikipediaより引用）

ということです。これがどういうことか、ということなんですが、まずはこんな感じの関数を考えます。

```kotlin
fun add(x: Int, y: Int): Int = x + y
```

この関数は以下のように使えますね。

```kotlin
fun main() {
  val a = add(2, 4)
  println(a)  // 6
}
```

この関数をカリー化すると、以下のように書けるということだと思います。

```kotlin
fun main() {
  val a = add(2)(4)
  println(a) // 6
}
```

上記のKotlinのコードは、実際に動くコードです。ちょっと見ただけではなんで動くのか不思議なコードなんですが、わかってしまえば「なるほど」という感じです。

手始めに、以下のコードを考えます。Kotlinでは、関数は第一級オブジェクトなのでした。そのため、変数に関数を代入し、関数を呼び出すことができました。例えば以下のコードでは、関数 `fun(int: Int): Int = int + 1` を代入した `increment(int)` を呼ぶと、 `int` の実引数に対して `int + 1` を評価した結果が返ります。

```kotlin
fun main() {
  val increment: (Int) -> Int = fun(int: Int): Int = int + 1
  println(increment(42)) // 43
}
```

なお、上記のコードはラムダを用いて以下のように書くこともできます。

```kotlin
fun main() {
  val increment: (Int) -> Int = { int: Int -> int + 1 }
  println(increment(42)) // 43
}
```

Kotlinでは関数は第一級オブジェクトなので、関数の返り値とすることも可能です。先ほど定義した `fun(int: Int): Int = int + 1` を返す関数を以下のように定義することができます。返り値の型が `(Int) -> Int` となっていることに注意してください。

```kotlin
fun incrementer(): (Int) -> Int = fun(int: Int): Int = int + 1
```

この関数の返り値は関数になるので、返り値として返された関数を呼び出すことができます。

```kotlin
fun main() {
  val incrementer = incrementer()
  println(incrementer(42)) // 43
}
```

もちろん、一時変数に代入しなくても、返り値として返された関数を直接呼び出すことも可能です。

```kotlin
fun main() {
  println(incrementer()(42)) // 43
}
```

次に、 `Int` 型の引数を取り、 `(Int) -> Int` 型の関数を返す関数を考えます。

```kotlin
fun add(x: Int): (Int) -> Int = TODO()
```

`(Int) -> Int` 型の関数は、既に出てきました。 `fun(int: Int): Int = int + 1` という感じで書けるのでした。add関数の場合は、この関数の引数を `x` に加えて返す関数となります。

```kotlin
fun add(x: Int): (Int) -> Int = fun(y: Int): Int = x + y
```

add関数の引数 `x` の実引数が `fun(y: Int): Int = x + y` の `x` に束縛されて返されますので、例えば `add(42)` の返り値として返される関数は、42に対する加算（ `42 + y` ）を行う関数になります。

```kotlin
fun main() {
  val addTo42 = add(42)
  println(addTo42(1)) // 43
  println(addTo42(100)) // 142
}
```

もちろん、この場合も、関数を一時変数に代入せずに直接呼び出すことができます。それが、冒頭に示した、以下のコードです。

```kotlin
fun main() {
  val a = add(2)(4)
  println(a) // 6
}
```

add関数をラムダで書き直すと、以下のようになります。

```kotlin
fun add(x: Int): (Int) -> Int = { y: Int -> x + y }
```

シグネチャをきれいにできるともっといいですが、とりあえずはKotlinで関数のカリー化をすることができました。

## Reference
1. [カリー化 - Wikipedia](https://ja.wikipedia.org/wiki/%E3%82%AB%E3%83%AA%E3%83%BC%E5%8C%96)（最終アクセス日：2022年2月10日）
2. Kotlin Design Patterns and Best Practices Second Edition、Alexey Soshin（2022）、Packt Publishing
