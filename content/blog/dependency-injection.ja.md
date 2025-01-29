---
title: "何故、依存性注入（DI）するのか"
date: 2022-01-24T22:34:00+09:00
description: "依存性注入（DI）パターンを用いる理由について書きます。"
categories: ["Design"]
draft: false
aliases:
    - /blog/why-we-inject-dependency/
---

DI（Dependency Injection、依存性注入）とは何かということと、何故DIを行うのかということをここに書きます。

## DI（Dependency Injection、依存性注入）とは

DIは、理解してしまえばシンプルな考え方です。

まずはDIを行わないパターンのコードを見てみます。プログラミング言語はKotlinですが、DIの考え方についてはプログラミング言語によらないので、コードについては参考程度に考えてください。

以下のコードでは、AwesomeClassがSomeClassを持っています。SomeClassのインスタンスは、AwesomeClassが生成しています。Kotlinの入門書に出てきそうな、普通のコードです。

```kotlin
class AwesomeClass() {
  val someClass: SomeClass = SomeClass()
}
```

次に、DIを行うパターンのコードを見てみます。あまりKotlinらしくないコードな気はしますが、Kotlinに慣れていない方への説明のためにこうしています。

```kotlin
class AwesomeClass constructor(val someClass: SomeClass)
```

1個目のパターンではクラスの内部でSomeClassのインスタンスを生成しているのに対して、2個目のパターンではAwesomeClassのコンストラクタでSomeClassのインスタンスを受け取っているのがわかるかと思います。1個目の例の「生成している」に対する2個目の例の「受け取っている」という表現がミソで、これを「SomeClassのインスタンスを外部から注入している」と考えるのがDIです。DIというのは、突き詰めるとこれだけのことです。

補足としては、上記2個目のパターンのようにコンストラクタでインスタンスを渡すことを「コンストラクタインジェクション（Constructor Injection）」と呼びます。その他に、外部からsetterを呼ぶことでインスタンスを渡す場合がありますが、このことは「セッターインジェクション（Setter Injection）」と呼びます。

DIとはこれだけのことなのですが、これだけのことが、プログラムを驚くほど柔軟にしてくれるのです。

## DIの活用

こんな感じのinterfaceがあったとします。

```kotlin
interface Greetable {
  fun greet()
}
```

Greetableを使うHelloクラスを定義します。以下のHelloクラスは、Helloクラス自身がGreetableのインスタンスを生成しています。GreetableのインスタンスをDIしないパターンの実装です。

```kotlin
class Hello {
  private val greetable: Greetable = object : Greetable {
    override fun greet() {
      println("Hello!")
    }
  }

  fun greet() {
    greetable.greet()
  }
}
```

コードから想像がつくと思いますが、このクラスは以下のように用います。

```kotlin
fun main(args: Array<String>) {
  val hello = Hello()
  hello.greet() // Hello!
}
```

このHelloクラスの `greet` 関数に「Hello!」ではなく「Good evening!」と出力させたい、となったらどうでしょうか。この場合は、 `greet` 関数の挙動を変えた、新たにクラスを作成する必要があります。

```kotlin
class GoodEvening {
  private val greetable: Greetable = object : Greetable {
    override fun greet() {
      println("Good evening!")
    }
  }

  fun greet() {
    greetable.greet()
  }
}
```

```kotlin
fun main(args: Array<String>) {
  val hello = Hello()
  hello.greet() // Hello!

  val goodEvening = GoodEvening()
  goodEvening.greet() // Good evening!
}
```

次に、GreetableをDI（コンストラクタインジェクション）するパターンを考えます。Greeterクラスを作成し、コンストラクタでGreetableをDIします。

```kotlin
class Greeter constructor(
  private val greetable: Greetable,
) {
  fun greet() {
    greetable.greet()
  }
}
```

DIを用いるパターンでは、外部からGreeterを注入しているので、`greet` 関数の挙動を変更することも容易です。Greeterクラスを使う箇所で、注入されるインスタンスの具象実装を定義することができるためです。

```kotlin
fun main(args: Array<String>) {
  val hello = Greeter(
    greetable = object : Greetable {
      override fun greet() {
        println("Hello!")
      }
    }
  )
  hello.greet() // Hello!

  val goodEvening = Greeter(
    greetable = object : Greetable {
      override fun greet() {
        println("Good evening!")
      }
    }
  )
  goodEvening.greet() // Good evening!
}
```

DIを用いないパターンでは2つのクラスを定義する必要がありましたが、DIを用いるパターンでは1つのクラスで事が足りました。これは、クラスが「インスタンスを生成する責務」と「インスタンスを使う責務」という、2つの異なる責務を持っていたところを、DIパターンを用いることでインスタンスを生成する責務をこのクラスから分離することができた、ということに起因します。

DIパターンによってプログラムに柔軟性がもたらされたわけですが、この柔軟性がさらに役に立つのが、ユニットテストにおいてです。

## ユニットテスト

DIによって、インスタンスを生成する責務と使う責務を分離することができるのでした。これによってもたらされる最大のメリットは、ユニットテストが書きやすくなることであると思います。

Booleanを生成する関数を持つinterfaceと、その関数の返り値がtrueの時にYesの文字列を、falseの時にNoの文字列を出力する関数を持つクラスを考えます。こんな感じです。

```kotlin
interface BooleanGenerator {
  fun generate(): Boolean
}
```

```kotlin
class YesOrNo constructor(
  private val booleanGenerator: BooleanGenerator,
) {
  fun yesOrNo(): String {
    return if (booleanGenerator.generate()) {
      "Yes"
    } else {
      "No"
    }
  }
}
```

BooleanGeneratorはYesOrNoクラスの外部から注入しているので、YesOrNoクラスを使う側が、BooleanGeneratorの挙動を自由に変えることができるのでした。この柔軟性が、ユニットテストを書くために役立ちます。

つまり、テストケースごとにBooleanGenerator#generateの返り値を自由に制御できる、ということです。これにより、YesOrNoの挙動をBooleanGeneratorの挙動から切り離して、YesOrNoクラス単体のテストを行うことが容易になります。

```kotlin
internal class YesOrNoTest {

  @Test
  fun yesOrNo_returnsYesIfTrue() {
    val target = YesOrNo(
      booleanGenerator = object : BooleanGenerator {
        override fun generate(): Boolean = true
      }
    )

    assertEquals("Yes", target.yesOrNo())
  }

  @Test
  fun yesOrNo_returnsNoIfFalse() {
    val target = YesOrNo(
      booleanGenerator = object : BooleanGenerator {
        override fun generate(): Boolean = false
      }
    )

    assertEquals("No", target.yesOrNo())
  }
}
```

ただし、実際にユニットテストを書く時には、モックライブラリを用いて依存先の挙動を制御することが多いと思います。モックライブラリを用いる場合でも、DIを行うことで依存先の挙動を制御することが容易になります。モックライブラリについては、以下の書籍・記事にまとめましたので、参照いただければと思います。

- [Android ユニットテスト ヒッチハイク・ガイド](https://syntaxsugar.booth.pm/items/4186246)
- [Mockk によるモック入門](https://okuzawats.com/blog/mockk/)