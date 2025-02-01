---
title: "[Kotlin] 関数定義が1つだけのinterfaceは `fun interface` で定義できる"
date: 2023-09-24T05:42:05+09:00
description: "Kotlinで関数定義が1つだけのinterfaceを定義する場合は、fun interfaceとして関数型のイターフェースとして宣言できます。"
categories: ["Kotlin"]
draft: false
---

表題のとおりですが、Kotlinで関数定義が1つだけのinterfaceを定義する場合は、`fun interface` として定義すると関数型のインターフェースとして宣言できます。

[Functional (SAM) interfaces | Kotlin Documentation](https://kotlinlang.org/docs/fun-interfaces.html)

どういうことでしょうか。まずは以下のようなinterfaceについて考えます。関数定義が1つだけのinterfaceです。

```kotlin
interface Logger {
  fun log(s: String)
}
```

この場合は、interfaceの宣言を `fun interface` にすることができます。

```kotlin
fun interface Logger {
  fun log(s: String)
}
```

`fun interface` にすることができるのは、関数定義が1つだけの時に限られます。つまり、以下のようなinterfaceの定義をすることはできません。

```kotlin
// コンパイルエラー
fun interface Logger {
  fun log(s: String)
  fun errorLog(e: Throwable)
}
```

さて、`fun interface` で定義したLoggerを実装してみます。オブジェクト式を用いた無名クラスを以下のように実装することができます。

```kotlin
val logger: Logger = object : Logger {
  override fun log(s: String) {
    println(s)
  }
}
```

`fun interface` の場合は以下のように書き直すことができます。オブジェクト式を用いる必要がなくなり、かなりスッキリと書くことができるようになります。

```kotlin
val logger: Logger = Logger { s ->
  println(s)
}
```

ここでは型推論が効きますし、ラムダ式の引数名も省略できますので、上記のコードはさらに短く書くことができます。また、この場合は関数参照も適用できます。

```kotlin
val logger = Logger(::println)
```

[[Android] UseCaseの実装](https://okuzawats.com/blog/implement-usecase-in-android/)に書いたようなUseCaseの実装についても `fun interface` を適用できます。`fun interface` を用いて、1 interfaceに1関数だけ定義することを保証するという使い方もできるかもしれません。

```kotlin
fun interface SomeUseCase {
  operator fun invoke()
}
```

## References

1. [Functional (SAM) interfaces | Kotlin Documentation](https://kotlinlang.org/docs/fun-interfaces.html)（最終アクセス日：2023/09/24）
