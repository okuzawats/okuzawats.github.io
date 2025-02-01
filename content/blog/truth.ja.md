---
title: "Truth によるアサーション入門"
date: 2022-10-08T10:30:00+09:00
description: "Google が中心となって開発している、Java / Android 向けのアサーションライブラリ Truth の使い方の紹介です。"
categories: ["Android", "Test"]
draft: false
---

Truth は、Google が中心となって開発している、Java / Android 向けのアサーションライブラリです。本記事では、Truth を用いてアサーションを行う方法を紹介します。Truth のすべての API に触れることはできませんので、筆者が代表的な API と考える API について触れていきます。より詳しい内容については、Truth の Web サイトを参照してください。

- [Truth - Fluent assertions for Java and Android](https://truth.dev/)

androidx.test による Truth の拡張については本記事では触れませんので、androidx.test のドキュメントを参照してください。

- [Package Index | Android Developers](https://developer.android.com/reference/androidx/test/packages)

## アサーションライブラリとは

アサーションライブラリとは、テストコードにおけるアサーションの可読性を上げ、またアサーションを書きやすくしてくれるライブラリです。アサーションに失敗した時のメッセージもわかりやすくしてくれるため、どういう理由でテストが失敗したのかも把握しやすくなります。

Java 向けのアサーションライブラリも種々ありますが、その中で Truth の特徴は、シンプルな API を提供していること、Android のサポートがあることがあげられます。前者の「シンプルな API を提供していること」は、テストコードを書く場合に考えることを減らして、機能の実装に集中することができるというメリットがあります。後者の「Android のサポートがあること」については、Android のプロジェクトで Truth を導入することを後押ししてくれます。

本書では、Android のプロジェクトのユニットテストに Truth を導入するケースを想定し、Truth による基本的なアサーションについて紹介します。

## Android のプロジェクトへの Truth の導入

Android のプロジェクトに Truth を導入するには、`build.gradle` または `build.gradle.kts` に以下のような記述を追加します（以下の例は `build.gradle` の場合です）。Truth のバージョンは、執筆時点の最新版を使用しています。

```groovy
dependencies {
  testImplementation "com.google.truth:truth:1.1.3"
}
```

Android Studio 上で Gradle Sync を行えば、Truth の導入は完了です。

## Boolean のアサーション

はじめに、Boolean 型の値に対して利用可能な Truth のアサーションを紹介します。

Truth での Boolean 型のアサーションについて、覚えておくべきことはあまり多くありません。Boolean 型の値は `true` と `false` のどちらかのみとなります。そのため、Boolean 型に対するアサーションで主に使用する関数は 2 つのみです。

そのうちの 1 つは `isTrue` です。Truth の `isTrue` を用いた Boolean 型の値に対するアサーションは、以下のように行います。 `assertThat` は Truth の提供する関数で、引数としてテスト対象となる実測値（actual）を渡します。実測値が `true` の場合は、テストが成功します。逆に実測値が `false` の場合はテストが失敗します。言い方を変えれば、このアサーションにおける期待値（expected）が `true` であるということを表しています。

```kotlin
@Test
fun test_isTrue() {
  val actual: Boolean = true
  assertThat(actual).isTrue()
}
```

もう 1 つは `isFalse` です。こちらは `isTrue` と逆に、実測値が `false` の時にテストが成功します。このアサーションの期待値は `false` です。

```kotlin
@Test
fun test_isFalse() {
  val actual: Boolean = false
  assertThat(actual).isFalse()
}
```

以上です！

Truth を用いることで、 `assertThat(actual).isTrue()` というようにアサーションを書くことができました。このコードは、"assert that actual is true."というように、自然言語（英語）のように読むことができます。筆者は、とても可読性の高いテストコードであると思います。Truth を用いることで、このように読みやすいアサーションを書けるようになります。次節以降、他のアサーションについて紹介していきます。

本節のコードを以下にまとめます。

```kotlin
package com.okuzawats.truth

import com.google.common.truth.Truth.assertThat
import org.junit.Test

class BooleanTest {
  @Test
  fun test_isTrue() {
    val actual: Boolean = true
    assertThat(actual).isTrue()
  }

  @Test
  fun test_isFalse() {
    val actual: Boolean = false
    assertThat(actual).isFalse()
  }
}
```

## Nullable のアサーション

本節では、Nullable な型に対して利用可能な Truth のアサーションを紹介します。

一口に Nullable と言っても、様々な型に対する Nullable があります。Kotlin では、任意の `T` 型に対する Nullable を `T?` として表現します。この `T?` 型に対するアサーションは、 `T` がどのような型かによってどのようなアサーションが適用可能かが決まり、一概にどのようなアサーションを適用できるとは言えません。しかしながら、Nullable な型の値が `null` なのかそうでないかということに着目すれば、2 通りのアサーションを考えればことが足ります。

以下、Boolean? 型を例としたサンプルコードを示します。

Nullable な型の値が `null` か否か、ということに着目した 1 つめのアサーションは、 `isNull` です。以下のテストケースでは、実測値が `null` の時、テストが成功します。すなわち、このアサーションの期待値は `null` です。

```kotlin
@Test
fun test_isNull() {
  val actual: Boolean? = null
  assertThat(actual).isNull()
}
```

2 つめは、 `isNotNull` です。このアサーションは、実測値が Non-Null であることを期待します。

```kotlin
@Test
fun test_isNotNull() {
  val actual: Boolean? = true
  assertThat(actual).isNotNull()
}
```

`null` かそうでないかに着目したアサーションは、以上の 2 種類です。型 `T` がどのような型なのかに応じて他に必要となるテストケースは様々あるとおもいますが、実測値が `null` なのかそうでないかという観点では、この 2 種類のアサーションを利用することができます。

本節のコードを以下にまとめます。

```kotlin
package com.okuzawats.truth

import com.google.common.truth.Truth.assertThat
import org.junit.Test

class NullableTest {
  @Test
  fun test_isNull() {
    val actual: Boolean? = null
    assertThat(actual).isNull()
  }

  @Test
  fun test_isNotNull() {
    val actual: Boolean? = true
    assertThat(actual).isNotNull()
  }
}
```

## String のアサーション

本節では、String 型に対して利用可能な Truth のアサーションを紹介します。

はじめに、`isEqualTo` を紹介します。`isEqualTo` はまた後で出てきますが、少し使い方に注意が必要なアサーションです。しかし、String 型に対しては完全に機能します。以下のサンプルコードは、実測値と期待値の文字列が文字列として等しいことを検証しています。

```kotlin
@Test
fun test_isEqualTo() {
  val actual: String = "hello world"
  assertThat(actual)
    .isEqualTo("hello world")
}
```

文字列の大文字・小文字を気にしなくて良い場合は、以下のサンプルコードのように `ignoringCase` を用いることができます。以下のサンプルコードでは、`hello world` という文字列と `HELLO WORLD` という文字列の大文字・小文字を無視して比較しています。

```kotlin
@Test
fun test_ignoreCase_isEqualTo() {
  val actual: String = "hello world"
  assertThat(actual)
    .ignoringCase()
    .isEqualTo("HELLO WORLD")
}
```

`hasLength` は、文字列の文字数を検証するために利用できます。以下のサンプルコードでは、実測値の文字数が 11 文字ですので、アサーションに成功します。

```kotlin
@Test
fun test_hasLength() {
  val actual: String = "hello world"
  assertThat(actual)
    .hasLength(11)
}
```

`startsWith` は、実測値が特定の文字列から開始されていることを検証します。以下のサンプルコードでは、実測値が `hello` から始まっているため、アサーションに成功します。

```kotlin
@Test
fun test_startsWith() {
  val actual: String = "hello world"
  assertThat(actual)
    .startsWith("hello")
}
```

`endsWith` は、`startsWith` とは逆に、実測値が特定の文字列で終了していることを検証します。

```kotlin
@Test
fun test_endsWith() {
  val actual: String = "hello world"
  assertThat(actual)
    .endsWith("world")
}
```

`isEmpty` は、実測値が空文字（ Kotlin では `""` ）であることを検証します。実測値が 1 文字以上の文字列だった場合、`null` の場合にはアサーションが失敗します。

```kotlin
@Test
fun test_isEmpty() {
  val actual: String = ""
  assertThat(actual)
    .isEmpty()
}
```

`isNotEmpty` は、実測値が空文字でない、すなわち 1 文字以上の文字列であることを検証します。

```kotlin
@Test
fun test_isNotEmpty() {
  val actual: String = "hello world"
  assertThat(actual)
    .isNotEmpty()
}
```

`contains` は、実測値に特定の文字列が含まれていることを検証します。以下のサンプルコードでは、実測値である `hello world` という文字列の中に `wor` という文字列が含まれています。そのため、このアサーションに成功します。

```kotlin
@Test
fun test_contains() {
  val actual: String = "hello world"
  assertThat(actual)
    .contains("wor")
}
```

`doesNotContain` は、`contains` とは逆に、実測値に特定の文字列が含まれていないことを検証します。以下のサンプルコードでは、実測値である `hello world` という文字列の中に `zzz` という文字列はふくまれていないため、アサーションに失敗します。

```kotlin
@Test
fun test_doesNotContains() {
  val actual: String = "hello world"
  assertThat(actual)
    .doesNotContain("zzz")
}
```

`containsMatch` は、正規表現を用いて文字列の検証を行います。実測値が正規表現にマッチした場合に検証が成功します。正規表現についてはここでは触れませんが、`containsMatch` を使いこなすことで、非常にパワフルな文字列の検証を行うことができます。

```kotlin
@Test
fun test_containsMatch() {
  val actual: String = "hello world"
  assertThat(actual)
    .containsMatch("^hello [a-z]+")
}
```

`doesNotContainMatch` は、`containsMatch` とは逆に、実測値が正規表現にマッチしないことを検証します。

```kotlin
@Test
fun testDoesNotContainMatch() {
  val actual: String = "hello world"
  assertThat(actual)
    .doesNotContainMatch("^[A-Z]+")
}
```

本節のコードを以下にまとめます。

```kotlin
package com.okuzawats.truth

import com.google.common.truth.Truth.assertThat
import org.junit.Test

class StringTest {
  @Test
  fun test_isEqualTo() {
    val actual: String = "hello world"
    assertThat(actual)
      .isEqualTo("hello world")
  }

  @Test
  fun test_ignoreCase_isEqualTo() {
    val actual: String = "hello world"
    assertThat(actual)
      .ignoringCase()
      .isEqualTo("HELLO WORLD")
  }

  @Test
  fun test_hasLength() {
    val actual: String = "hello world"
    assertThat(actual)
      .hasLength(11)
  }

  @Test
  fun test_startsWith() {
    val actual: String = "hello world"
    assertThat(actual)
      .startsWith("hello")
  }

  @Test
  fun test_endsWith() {
    val actual: String = "hello world"
    assertThat(actual)
      .endsWith("world")
  }

  @Test
  fun test_isEmpty() {
    val actual: String = ""
    assertThat(actual)
      .isEmpty()
  }

  @Test
  fun test_isNotEmpty() {
    val actual: String = "hello world"
    assertThat(actual)
      .isNotEmpty()
  }

  @Test
  fun test_contains() {
    val actual: String = "hello world"
    assertThat(actual)
      .contains("wor")
  }

  @Test
  fun test_doesNotContains() {
    val actual: String = "hello world"
    assertThat(actual)
      .doesNotContain("zzz")
  }

  @Test
  fun test_containsMatch() {
    val actual: String = "hello world"
    assertThat(actual)
      .containsMatch("^hello [a-z]+")
  }

  @Test
  fun testDoesNotContainMatch() {
    val actual: String = "hello world"
    assertThat(actual)
      .doesNotContainMatch("^[A-Z]+")
  }
}
```

## Collection のアサーション

Truth では、Collection に対するアサーションが豊富に用意されています。ここですべてを紹介するのは難しいため、一部の代表的なアサーションの紹介に留めます。

`contains` は、String のアサーションの節でも出てきました。Collection に特定の値が含まれていることを検証します。以下のサンプルコードでは、実測値の `List` に `42` が含まれているため、アサーションが成功します。

```kotlin
@Test
fun test_contains() {
  val actual: List<Int> = listOf(1, 2, 3, 42, 100)
  assertThat(actual)
    .contains(42)
}
```

`containsExactly` は、実測値の Collection と `containsExactly` の可変長引数の要素がすべて 1 対 1 で対応していることを検証します。この時、順序は問いません。以下の例では、実測値の Collection と `containsExactly` の可変長引数は、共に `1` `2` `3` `42` `100` であり、すべて 1 対 1 で対応しているため、アサーションが成功します。

```kotlin
@Test
fun test_containsExactly() {
  val actual: List<Int> = listOf(1, 2, 3, 42, 100)
  assertThat(actual)
    .containsExactly(100, 42, 3, 2, 1)
}
```

`isInOrder` は、実測値の Collection が順序通りに並んでいることを検証します（後述する Comparable とも関連します）。ラムダを用いてテストケース内で順序の比較方法を指定することが可能な `isInOrder` も用意されています。以下のサンプルコードでは、実測値の Collection が、整数値で小さいものから大きいものの順に並んでいるため、アサーションに成功します。

```kotlin
@Test
fun test_isInOrder() {
  val actual: List<Int> = listOf(1, 2, 3, 42, 100)
  assertThat(actual)
    .isInOrder()
}
```

本節のコードを以下にまとめます。

```kotlin
package com.okuzawats.truth

import com.google.common.truth.Truth.assertThat
import org.junit.Test

class CollectionTest {
  @Test
  fun test_contains() {
    val actual: List<Int> = listOf(1, 2, 3, 42, 100)
    assertThat(actual)
      .contains(42)
  }

  @Test
  fun test_containsExactly() {
    val actual: List<Int> = listOf(1, 2, 3, 42, 100)
    assertThat(actual)
      .containsExactly(100, 42, 3, 2, 1)
  }

  @Test
  fun test_isInOrder() {
    val actual: List<Int> = listOf(1, 2, 3, 42, 100)
    assertThat(actual)
      .isInOrder() // ラムダなどのパターンあり
  }
}
```

## Comparable のアサーション

本節では、Comparable な型に対して利用可能な Truth のアサーションを紹介します。すなわち、Comparable インターフェースを実装した型（代表的なものは、Int 型です）の比較に関するアサーションです。ここでは、わかりやすく Int 型を用いてサンプルコードを示します。

`isGreaterThan` は、実測値が `isGreaterThan` の引数よりも大きいこと、つまり「実測値 >  `isGreaterThan` の引数」となっていることを検証します。以下のサンプルコードにおいて、1 つめのアサーションで 42 > 0 であることを検証しています。これは正しいためアサーションに成功します。2 つめのアサーションでも、42 > 41 は正しいため、アサーションに成功します。一方、コメントアウトした 3 つめのアサーション 42 > 42 は正しくないため、アサーションに失敗します。

```kotlin
@Test
fun test_isGreaterThan() {
  val actual: Int = 42
  assertThat(actual).isGreaterThan(0)
  assertThat(actual).isGreaterThan(41)
  // assertThat(actual).isGreaterThan(42) // 失敗
}
```

`isLessThan` は `isGreaterThan` と逆に、「実測値 <  `isLessThan` の引数」となっていることを検証します。以下のサンプルコードでは、42 < 100、42 < 43 は正しいためアサーションに成功しますが、42 < 42 は正しくないためアサーションに失敗します。

```kotlin
@Test
fun test_isLessThan() {
  val actual: Int = 42
  assertThat(actual).isLessThan(100)
  assertThat(actual).isLessThan(43)
  // assertThat(actual).isLessThan(42) // 失敗
}
```

`isAtLeast` は `isGreaterThan` と似ていますが、こちらはいわゆる「大なりイコール」です。つまり、「実測値 >= `isAtLeast` の引数」の時にアサーションが成功します。`isGreaterThan` との違いは、実測値と `isAtLeast` の引数の比較結果が同等である場合（ここで難しい言い回しをしている理由は、この記事を最後まで読むとわかると思います）にもアサーションが成功するという点にあります。

```kotlin
@Test
fun test_isAtLeast() {
  val actual: Int = 42
  assertThat(actual).isAtLeast(0)
  assertThat(actual).isAtLeast(42)
}
```

`isAtMost` は、いわゆる「小なりイコール」です。「実測値 <=  `isAtMost` の引数」の時にアサーションが成功します。`isAtMost` と `isLessThan` の違いは、`isAtLeast` と `isGreaterThan` の場合の違いと同じです。

```kotlin
@Test
fun test_isAtMost() {
  val actual: Int = 42
  assertThat(actual).isAtMost(100)
  assertThat(actual).isAtMost(42)
}
```

`isEquivalentAccordingToCompareTo` は、実測値と `isEquivalentAccordingToCompareTo` の引数の比較結果が同等である時にアサーションが成功します。つまり、ふたつの間に大小関係がない場合です。例として Int 型を用いているためにサンプルコードが難解になってしまっているのですが、Int 型の場合は `isEquivalentAccordingToCompareTo` ではなく `isEqualTo` を使うべきです。`isEquivalentAccordingToCompareTo` は、Comparable 間の大小関係がないことの検証に用いることができます。

```kotlin
@Test
fun test_isEquivalentAccordingToCompareTo() {
  val actual: Comparable<Int> = 42 // Intの場合はisEqualToを使うのがよい
  assertThat(actual).isEquivalentAccordingToCompareTo(42)
}
```

`isIn` は、実測値が範囲内に含まれていることを検証します。以下のサンプルコードでは、「0 <= 実測値 <= 100」であることを検証しています（ `..` は Range を表す Kotlin の演算子です）。

```kotlin
@Test
fun test_isIn() {
  val actual: Int = 42
  assertThat(actual).isIn(0..100)
}
```

`isNotIn` も `isIn` と同様に範囲の検証に用いられるアサーションですが、こちらは実測値が範囲内に含まれていないことを検証します。以下のサンプルコードでは、「-100 <= 実測値 < 0」でないことを検証しています（ `until` も Range を表す Kotlin の演算子です。こちらは 2 つめの引数に対して開区間となります）。

```kotlin
@Test
fun test_isNotIn() {
  val actual: Int = 42
  assertThat(actual).isNotIn(-100 until 0)
}
```

本節のコードを以下にまとめます。

```kotlin
package com.okuzawats.truth

import com.google.common.truth.Truth.assertThat
import org.junit.Test

class ComparableTest {
  @Test
  fun test_isGreaterThan() {
    val actual: Int = 42
    assertThat(actual).isGreaterThan(0)
    assertThat(actual).isGreaterThan(41)
    // assertThat(actual).isGreaterThan(42) // 失敗
  }

  @Test
  fun test_isLessThan() {
    val actual: Int = 42
    assertThat(actual).isLessThan(100)
    assertThat(actual).isLessThan(43)
    // assertThat(actual).isLessThan(42) // 失敗
  }

  @Test
  fun test_isAtLeast() {
    val actual: Int = 42
    assertThat(actual).isAtLeast(0)
    assertThat(actual).isAtLeast(42)
  }

  @Test
  fun test_isAtMost() {
    val actual: Int = 42
    assertThat(actual).isAtMost(100)
    assertThat(actual).isAtMost(42)
  }

  @Test
  fun test_isEquivalentAccordingToCompareTo() {
    val actual: Comparable<Int> = 42 // Intの場合はisEqualToを使うのがよい
    assertThat(actual).isEquivalentAccordingToCompareTo(42)
  }

  @Test
  fun test_isIn() {
    val actual: Int = 42
    assertThat(actual).isIn(0..100)
  }

  @Test
  fun test_isNotIn() {
    val actual: Int = 42
    assertThat(actual).isNotIn(-100 until 0)
  }
}
```

## isEqualTo 、isNotEqualTo を用いたアサーション

「Comparable のアサーション」の節で少し触れましたが、例えば Int 型の実測値が期待値と一致することの検証は、以下のサンプルコードのように、`isEqualTo` を用いて行うことができます。

```kotlin
@Test
fun test_isEqualTo() {
  val actual = 40 + 2
  assertThat(actual).isEqualTo(42)
}
```

この `isEqualTo` 、簡単そうに見えますが、使い方に少し注意が必要です。

例えば、実測値と期待値が共に SomeClass のインスタンスである時、以下のサンプルコードのアサーションは成功しそうに見えますが、実際には失敗します。

```kotlin
@Test
fun test_someClass_isEqualTo() {
  val actual = SomeClass()
  val expected = SomeClass()
  assertThat(actual).isEqualTo(expected) // 失敗
}
```

これは何故かというと、`isEqualTo` は期待値と実測値の `equals` の結果をアサーションに用いているからです（ライブラリの内部ではもう少し複雑なことをやっていますが、ライブラリを使う側ではそれを意識する必要はないと思います）。

そのため、例えば以下のサンプルコードのように、乱暴に `equals` をオーバーライドすれば、上記のアサーションが成功するようになります。

```kotlin
class SomeClass {
  override fun equals(other: Any?): Boolean {
    return other is SomeClass
  }
}
```

プリミティブな値の検証や、Kotlin のデータクラスの検証を行う場合は、`isEqualTo` は非常に便利に利用できます。それ以外の場合については、上記の内容に注意しましょう。

`isEqualTo` の逆に、`isNotEqualTo` は、実測値と期待値の `equals` が false を返すことを検証します。

```kotlin
@Test
fun test_otherClass_isNotEqualTo() {
  val actual = OtherClass()
  val expected = OtherClass()
  assertThat(actual).isNotEqualTo(expected)
}
```

本節のコードを以下にまとめます。

```kotlin
package com.okuzawats.truth

import com.google.common.truth.Truth.assertThat
import org.junit.Test

class SomeClass {
  override fun equals(other: Any?): Boolean {
    return other is SomeClass
  }
}

class OtherClass

class EqualToTest {
  @Test
  fun test_isEqualTo() {
    val actual = 40 + 2
    assertThat(actual).isEqualTo(42)
  }

  @Test
  fun test_someClass_isEqualTo() {
    val actual = SomeClass()
    val expected = SomeClass()
    // SomeClassは同じ型同士で比較した時に `equalTo` がtrueを返す
    assertThat(actual).isEqualTo(expected)
  }

  @Test
  fun test_otherClass_isNotEqualTo() {
    val actual = OtherClass()
    val expected = OtherClass()
    assertThat(actual).isNotEqualTo(expected)
  }
}
```

## isInstanceOf を用いた型のアサーション

`isInstanceOf` を用いて、型の検証を行うことができます。

以下のサンプルコードでは、実測値が Comparable 型であることの検証を行っています。Kotlin の Int 型は Comparable interface を実装していますので、アサーションに成功します。

```kotlin
@Test
fun test_instanceOf() {
  val actual: Int = 42
  assertThat(actual).isInstanceOf(Comparable::class.java)
}
```

本節のコードを以下にまとめます。

```kotlin
package com.okuzawats.truth

import com.google.common.truth.Truth.assertThat
import org.junit.Test

class InstanceOfTest {
  @Test
  fun test_instanceOf() {
    val actual: Int = 42
    assertThat(actual).isInstanceOf(Comparable::class.java)
  }
}
```

## 例外送出のアサーション

Truth は、直接的に例外が送出される処理の検証を行う API を提供していないようです。

そのため、Truth の API を用いて例外送出の検証をするためには、少し冗長ではありますが以下のようなコードを書く必要があると思います。前述の `isInstanceOf` で例外型の検証を行うこともできるでしょう。

```kotlin
@Test
fun test_throwable() {
  var throwable: Throwable? = null
  try {
    ThrowException()
      .throwException()
  } catch (e: Throwable) {
    throwable = e
  }
  assertThat(throwable)
    .isNotNull()
}
```

または、例外送出の検証については Truth を用いることを諦め、テスティングフレームワークの機能で行う方法もあると思います。

本節のコードを以下にまとめます。

```kotlin
package com.okuzawats.truth

import com.google.common.truth.Truth.assertThat
import org.junit.Test

class ThrowException {
  fun throwException() {
    throw Throwable()
  }
}

class ExceptionTest {
  @Test
  fun test_throwable() {
    var throwable: Throwable? = null
    try {
      ThrowException()
        .throwException()
    } catch (e: Throwable) {
      throwable = e
    }
    assertThat(throwable)
      .isNotNull()
  }
}
```
