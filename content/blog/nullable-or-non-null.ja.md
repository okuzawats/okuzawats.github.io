---
title: "[Kotlin] Non-Nullにこだわらず、適切にNullableを使う"
date: 2024-10-09T06:15:00+09:00
description: "KotlinではNon-NullとNullableを区別します。一般にNullableよりもNon-Nullとすることが好まれますが、適切に使い分けるべきと考えます。"
categories: ["Kotlin", "プログラミング"]
draft: false
---

Kotlinでは、Non-NullとNullableを区別します。Nullableな型は、型の末尾に `?` を付けます。

```kotlin
var nonNull: Int = 42
var nullable: Int? = null
```

一般に、「NullableよりもNon-Nullが好ましい」とされます。それについては自分も同意します。しかし、無条件に同意するものではありません。例えば以下のコードです。この `0` が何の意味もない、いわば `null` の代わりとして使われているのであれば、Nullableとするよりも悪いと思います。

```kotlin
var a: Int = 0
```

上記の変数がNullableであれば、KotlinはNon-NullとNullableを区別するため、以下のようにnullチェックが強制されます。

```kotlin
if (a != null) {
  // do something here
}
```

一方で、 `null` の代わりにデフォルト値（例えば `0` ）を使用した場合は、これは強制されません。簡単に言えば、このチェックは容易に忘れられます。

```kotlin
if (a != 0) { // <= このifは簡単に忘れ去れられる
  // do something here
}
```

デフォルト値が何らかの意味を持つ場合はNon-Nullとしてデフォルト値を用いることで問題ありません。一方で、型をNon-Nullにするためだけにデフォルト値を用いる場合、すなわちデフォルト値が何の意味も持たない場合は、Nullableよりも悪くなります。何の意味も持たないオブジェクトを容易に利用可能な状態となってしまうためです。

このような場合には、空の状態を表すためにNullableな変数を用いて `null` として状態を表す、または場合によって[nullオブジェクトパターン](https://okuzawats.com/blog/null-object-pattern/)の使用を考えるべきである、と思います。
