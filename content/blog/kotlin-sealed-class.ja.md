---
title: "[Kotlin] sealed classに親しむ"
date: 2022-01-20T06:11:20+09:00
description: "Kotlinのsealed classについての記事です。"
categories: ["Kotlin"]
draft: false
---

Kotlinの `sealed class` に親しんでいきます。

`sealed class` は、継承に制約を課す言語機能です。Kotlinのdocsによれば、 `sealed class` として定義されたクラスのサブクラスは、すべてコンパイル時に定義されている必要があります。

> Sealed classes and interfaces represent restricted class hierarchies that provide more control over inheritance. All direct subclasses of a sealed class are known at compile time. No other subclasses may appear after a module with the sealed class is compiled. For example, third-party clients can't extend your sealed class in their code. Thus, each instance of a sealed class has a type from a limited set that is known when this class is compiled.
> [Sealed classes | Kotlin](https://kotlinlang.org/docs/sealed-classes.html)

## はじめてのKotlin `sealed class`

`sealed class` の簡単なサンプルコードを以下に示します。親の `sealed class` としてSealedClassというクラスを定義しています。また、そのサブクラス（ `object` ）として、A、B、Cを定義しています。 `sealed class` の付けられたクラスはabstractになります。そのため、SealedClass自身をインスタンス化することはできず、インスタンスとして存在し得るのは、この場合、そのサブクラスであるA、B、Cのみです。

```kotlin
sealed class SealedClass
object A : SealedClass()
object B : SealedClass()
object C : SealedClass()
```

`sealed class` のサブクラスは、コンパイル時に定義されている必要があるのでした。この性質は、 `when` によるパターンマッチングと相性がよく、サブクラスの型に応じた処理の分岐を容易に行うことができます。コンパイル時に全てのサブクラスがわかっているので、 `when` のブランチを包括的（exhaustive）に定義することができます。IntelliJ IDEAやAndroid Studioでは、 `when` のブランチがexhaustiveでない場合に警告が表示されます[^1]。

```kotlin
fun main() {
  val sealedClass: SealedClass = A

  when (sealedClass) {
    is A -> {
      // do something here
    }
    is B -> {
      // do something here
    }
    is C -> {
      // do something here
    }
  }
}
```

なお、Kotlin 1.7以降は、exhaustiveでない `when` がコンパイルエラーになることが予定されています。

また、Kotlinスタートブック（長澤（2016））などには、 `sealed class` のサブクラスは `sealed class` にネストされたサブクラスしか定義することができない、という記述がありました。下のサンプルコードのように、 `sealed class` 本体にサブクラスを定義するということです。

```kotlin
sealed class SealedClass {
  object A : SealedClass()
  object B : SealedClass()
  object C : SealedClass()
}
```

この制限については、Kotlin 1.1 Betaにて変更が入り、必ずしもサブクラスをネストして定義する必要はなくなったようです（調べてくれた `@tkhs0604` さん、ありがとうございました）。

サブクラスをネストして定義する場合は、通常、以下のサンプルコードのように、 `Parent.Sub` というシンタックスでサブクラスを使います。

```kotlin
fun main() {
  val sealedClass: SealedClass = SealedClass.A

  when (sealedClass) {
    is SealedClass.A -> {
      // do something here
    }
    is SealedClass.B -> {
      // do something here
    }
    is SealedClass.C -> {
      // do something here
    }
  }
}
```

サブクラスをネストして定義した場合の方がタイプ数は多くなってしまうのですが、こちらの方がクラスの継承に規律ができて、自分は好みです。

## Kotlin `sealed class` に親しむ

ここからが本題です。Kotlinの `sealed class` で遊びながら親しんでいきます。

### コンストラクタ

まずは `sealed class` のコンストラクタを作ってみます。サブクラスごとに特定の値を持つことができます。

```kotlin
sealed class SealedClass(val awesomeValue: Int)
object A : SealedClass(awesomeValue = 42)
object B : SealedClass(awesomeValue = 43)
object C : SealedClass(awesomeValue = 44)
```

```kotlin
  val sealedClass: SealedClass = SealedClass.A

  when (sealedClass) {
    is A -> {
      println("value is ${sealedClass.awesomeValue}") // 42
    }
    is B -> {
      println("value is ${sealedClass.awesomeValue}") // 43
    }
    is C -> {
      println("value is ${sealedClass.awesomeValue}") // 44
    }
  }
```

### サブクラスに `data class` を用いる

ここまでのサンプルコードでは、 `sealed class` のサブクラスは、すべてシングルトンになっていました。お気付きのとおり、これらのサンプルコードは `enum class` でもほぼ等価なコードを実装することができます。

Kotlinの `sealed class` では、サブクラスにシングルトンでない通常のクラスを定義することもできるので、 `enum class` と使い分けることが可能です。

以下のサンプルコードでは、 `sealed class` のサブクラスとして、 `data class` を定義しています。この場合は、サブクラスごとに異なるプロパティを持たせることができます。

```kotlin
sealed class SealedClass
data class A(val a: Int) : SealedClass()
data class B(val b: Int) : SealedClass()
```

```kotlin
  val sealedClass: SealedClass = A(42)

  when (sealedClass) {
    is A -> {
      println("value is ${sealedClass.a}")
    }
    is B -> {
      println("value is ${sealedClass.b}")
    }
  }
```

### サブクラスに `class` を用いる

`data class` でない通常の `class` を `sealed class` のサブクラスにすることも可能ですが、Android StudioやIntelliJ IDEAなどのIDEから、 `equals` と `hashCode` をoverrideしなさい、という警告が表示されるかと思います。個人的には、 `sealed class` を使いたい場合のサブクラスは `object` と `data class` で事足りる場合が多い気がしています。

以下のoverrideは、IDEが自動生成したコードです。

```kotlin
class C : SealedClass() {
  override fun equals(other: Any?): Boolean {
    return this === other
  }

  override fun hashCode(): Int {
    return System.identityHashCode(this)
  }
}
```

### `abstract` なプロパティを定義する

`sealed class` に `abstract` なプロパティを定義することで、サブクラスが特定のプロパティを持っていることを保証できます。以下のサンプルコードでは、 `sealed class` のサブクラスが必ずawesomeValueというInt型のプロパティを持ちます。

```kotlin
sealed class SealedClass {
  abstract val awesomeValue: Int
}
data class A(override val awesomeValue: Int) : SealedClass()
data class B(override val awesomeValue: Int) : SealedClass()
```

```kotlin
  val sealedClass: SealedClass = A(42)
  println("value is ${sealedClass.awesomeValue}")
```

### 拡張関数を定義する

`sealed class` に拡張関数を定義することもできます。

```kotlin
fun SealedClass.awesomeHello() = println("hello awesome world!")
```

`sealed class` （のサブクラス）のインスタンスに対して、拡張関数を呼び出します。

```kotlin
  val sealedClass: SealedClass = A(42)
  sealedClass.awesomeHello()
```

## Reference
1. [Sealed classes | Kotlin](https://kotlinlang.org/docs/sealed-classes.html)（最終アクセス日：2022年1月18日）
2. [KotlinのExhaustive when statementsについて - Kenji Abe - Medium](https://star-zero.medium.com/kotlin%E3%81%AEexhaustive-when-statements%E3%81%AB%E3%81%A4%E3%81%84%E3%81%A6-7127c01b5448)（最終アクセス日：2022年1月19日）
3. Kotlinスタートブック 新しいAndroidプログラミング、長澤太郎（2016）、リックテレコム 
4. [kenken🐶さんはTwitterを使っています 「@okuzawats 気になってサンクトペテルブルクの奥へ向かってみたところ、1.1 Bataで変更が入ったようでした👀 https://t.co/ghqz8q1PPi」 / Twitter](https://twitter.com/tkhs0604/status/1483436623518142465)（最終アクセス日：2022年1月19日）
5. [Kotlin 1.1 Beta Is Here! | The Kotlin Blog](https://blog.jetbrains.com/kotlin/2017/01/kotlin-1-1-beta-is-here/)（最終アクセス日：2022年1月19日）

[^1]: Kotlin 1.6から、exhaustiveでない `when` をオプションでエラーにすることができます。Kotlin 1.7からは標準でエラーになります（参考文献2を参照）。