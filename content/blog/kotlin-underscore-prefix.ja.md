---
title: "[Kotlin] underscore prefixに関するコーディング規約"
date: 2022-02-02T22:39:00+09:00
description: "Kotlinのunderscore prefixに関するコーディング規約について。"
categories: ["Kotlin"]
draft: false
---

Kotlinのunderscore prefixに関するコーディング規約について書きます。

## KotlinのCoding Conventions

Kotlin公式のCoding conventionsには、「_If a class has two properties which are conceptually the same but one is part of a public API and another is an implementation detail, use an underscore as the prefix for the name of the private property_」との記述があります。

つまり、外部にはinterfaceのみを公開し、privateなプロパティとしてその実装クラスのインスタンスを持っている場合です。この場合に、interfaceに対してunderscore prefix無しの名前を、実装クラスのインスタンスに対してunderscore prefix有りの名前をつけるとしています。

サンプルコードを以下に示します。 `_awesomeStrings` が実装クラスのインスタンス、 `awesomeStrings` がinterfaceです。interfaceに対してgetterを定義し、getterが実装クラスのインスタンスを返す、というKotlinでよく見る実装です。

```kotlin
class AwesomeClass {
  private val _awesomeStrings = mutableListOf<String>()
  val awesomeStrings: List<String> get() = _awesomeStrings
}
```

Kotlin公式のCoding Conventionsでは、上記のような場合に、privateプロパティにunderscore prefixを付けるとありました。他にはunderscore prefixに関する記述は見つけられませんでした。

## AndroidのKotlin Style Guide

AndroidのKotlin style guideにも、同様の記述がありました。「_When a backing property is needed, its name should exactly match that of the real property except prefixed with an underscore._」という記述があります。表現は異なりますが、指している内容は同等です。

こんな感じのサンプルコードが示されています。

```kotlin
  private var _table: Map? = null

  val table: Map
    get() {
      if (_table == null) {
        _table = HashMap()
      }
      return _table ?: throw AssertionError()
    }
```

## Kotlin Design Patterns and Best Practices

ここからが本題です。Kotlin Design Patterns and Best Practices Second Edition (Soshin (2022))を読んでいたら、以下の記述を見つけました。

> Using underscores for private variables is a common convention in Kotlin. (Soshin (2022)、P.59)

以下のようなコードを書く時に、 `this.awesome = awesome` というような冗長なコードを書かなくて済むというメリットと、 `awesome = awesome` と書いてしまうミスを無くすことができるメリットがある、と言うのですね。

```kotlin
class AnotherClass {
  private var _awesome: AwesomeClass = AwesomeClass()

  fun setAwesome(awesome: AwesomeClass) {
    _awesome = awesome
  }
}
```

そのメリットについてはよく理解できるのですが、Kotlinのcommon conventionだという認識はなかったのでちょっとびっくりしました。自分の調べた限りではKotlin公式のCoding conventionには上記のようなconventionは記載されておらず、common conventionなのかどうかは自分はわかりません。提示されているメリットについては理解できます。

一方で、この書き方は一種の（システム）ハンガリアン記法です。例えば「ハンガリアン記法は用いない」とするコーディング規約を設けているプロジェクトにおいてこういったunderscore prefixを付けることにした場合は、コーディング規約に例外が生まれることになります。

まあ、KotlinのCoding Conventionsの項で書いた以下の書き方を認める時点で、例外が生まれている感はありますが。

```kotlin
class AwesomeClass {
  private val _awesomeStrings = mutableListOf<String>()
  val awesomeStrings: List<String> get() = _awesomeStrings
}
```

そのあたりのメリット・デメリットをチームで話して、どの書き方は認めてどの書き方は認めない、ということを決めておくといいのかなと思いました。

## Reference
1. [Coding conventions | Kotlin](https://kotlinlang.org/docs/coding-conventions.html#names-for-backing-properties)（最終アクセス日：2022年2月2日）
2. [Kotlin style guide | Android Developers](https://developer.android.com/kotlin/style-guide#backing_properties)（最終アクセス日：2022年2月2日）
3. Kotlin Design Patterns and Best Practices Second Edition、Alexey Soshin（2022）、Packt Publishing
4. [ハンガリアン記法 - Wikipedia](https://ja.wikipedia.org/wiki/%E3%83%8F%E3%83%B3%E3%82%AC%E3%83%AA%E3%82%A2%E3%83%B3%E8%A8%98%E6%B3%95)
