---
title: "Mockk によるモック入門"
date: 2022-10-22T22:50:00+09:00
description: "Kotlin 製のモックライブラリ MockK の使い方の紹介です。"
categories: ["Android", "Test"]
draft: false
---

MockK は、オープンソースで開発されている、Kotlin 製のモックライブラリです。本記事では、MockK を用いてテストコードを書く方法を紹介します。MockK のすべての API に触れることはできませんので、筆者が代表的な API と考える API について触れていきます。より詳しい内容については、MocKK の Web サイトを参照してください。

- [MockK | mocking library for Kotlin](https://mockk.io/)

## モックライブラリとは

モックライブラリとは、ユニットテストで用いるためのテストダブル（テストのための代役）を便利に扱うためのライブラリです。モックライブラリを用いることで、わざわざ自分でテストダブルを作るためのコードを書かなくとも、快適にテストコードを書くことが可能となります。

Kotlin から便利に利用できるモックライブラリには、本記事で紹介する MockK の他に、Java 向けのモックライブラリである Mockito の Kotlin 向けラッパーである Mockito-Kotlin が存在します。

- [mockito/mockito-kotlin: Using Mockito with Kotlin](https://github.com/mockito/mockito-kotlin)

MockK と Mockito-Kotlin のどちらを選んでも基本的な機能には大きな差はありませんが、MockK は Mockito-Kotlin にない便利・強力な機能が存在します。「Android ユニットテスト ヒッチハイク・ガイド」の本での結論とは異なってしまいますが、本記事執筆時現在、筆者個人的には MockK を選ぶことを推奨したいと思います。

本書では、Android のプロジェクトのユニットテストに MockK を導入するケースを想定し、MockK の基本的な機能について紹介します。

## Android のプロジェクトへの MockK の導入

Android のプロジェクトに MockK を導入するには、build.gradle または build.gradle.kts に以下のような記述を追加します（以下の例は build.gradle の場合です）。MockK のバージョンは、執筆時点の最新版を使用しています。

```groovy
dependencies {
  testImplementation "com.google.truth:truth:1.1.3"
  testImplementation "io.mockk:mockk:1.13.2" // 追加
}
```

ここで、アサーションライブラリとして Truth を用いています。Truth の使い方については、[Truth によるアサーション入門](https://okuzawats.com/blog/truth/)を参照してください。

Android Studio 上で Gradle Sync を行えば、MockK の導入は完了です。

## スタブ化

まずは以下のようなクラスをテストすることを考えます。FortuneCookie クラスの draw 関数は占いを行う関数で、占い結果として Fortune 型を返します。また、FortuneCookie クラスのコンストラクタ引数として、サイコロを表す Dice 型を取ります。

```kotlin
/**
 * 占いを行うClass
 *
 * @property dice 占いに必要なサイコロ
 */
class FortuneCookie(
  private val dice: Dice,
) {
  /**
   * 占いを行い、結果を[Fortune]として返す
   *
   * @return [Fortune]
   * @throws [FortuneException]
   */
  fun draw(): Fortune {
    // サイコロの出目
    val dots = dice.roll()

    // サイコロの出目が1ならGood、2から6ならBadを返す。
    // それ以外の出目が出た場合はFortuneExceptionをthrowする。
    return if (dots == 1) {
      Fortune.Good
    } else if(dots in 2..6) {
      Fortune.Bad
    } else {
      throw FortuneException("can not calculate fortune.")
    }
  }
}
```

Fortune 型の定義を以下に示します。Good 、Bad の列挙子を持つ列挙型です。

```kotlin
/**
 * 運勢を表す列挙型
 */
enum class Fortune {
  /**
   * 吉
   */
  Good,

  /**
   * 凶
   */
  Bad,
}
```

Dice 型は interface として定義されています。roll 関数は、1 から 6 までの整数値をランダムに返すように実装します。

```kotlin
/**
 * サイコロを表すinterface
 */
interface Dice {
  /**
   * 1から6までの整数値をランダムに返す。
   */
  fun roll(): Int
}
```

また、FortuneCookie#draw を呼び出した際、占いの実行に失敗した場合に throw される例外型を定義します。

```kotlin
/**
 * 運勢を占うことができなかった場合の例外
 *
 * @param message エラーメッセージ
 */
class FortuneException(
  override val message: String,
): Throwable(message)
```

次に、FortuneCookie クラスに対するテストを行う際に、MockK を用いて Dice 型のモックを用意します。書き方はいくつかありますが、ここでは `@MockK` アノテーションを用いて、`lateinit var` としてプロパティを宣言します。

```kotlin
@MockK
lateinit var dice: Dice
```

`@MockK` アノテーションを付与したプロパティにインスタンスを注入するため、`MockKAnnotations.init` を呼び出します。引数にはテストクラスのインスタンス（`this`）を渡しています。この処理により、MockK がモックオブジェクトの注入を行います。また、テストケースの実行のたびに `MockKAnnotations.init` を呼び出すため、`@Before` の中で実行しています。

```kotlin
@Before
fun setUp() {
  MockKAnnotations.init(this)
}
```

次に、テストの対象となる FortuneCookie クラスのインスタンスを用意します。こちらも `lateinit var` で宣言しておき、 `@Before` でインスタンスの生成を行います。FortuneCookie クラスはコンストラクタ引数として Dice 型の `dice` を取るので、インスタンス化する際に MockK によって生成されたモックオブジェクトを渡します。

```kotlin
lateinit var fortuneCookie: FortuneCookie
```

```kotlin
@Before
fun setUp() {
  MockKAnnotations.init(this)
  fortuneCookie = FortuneCookie(dice)
}
```

ここまでのコードを以下にまとめます。これでモックオブジェクトの準備ができましたので、次節以降、テストケースを作成していきます。

テストケースを作成します。まずは、作成したモックオブジェクト（`dice`）の roll 関数にテストに都合の良い値を返させます。これには、MockK の `every` が利用できます。`every` の引数のブロック内でモックオブジェクトの関数呼び出しを行い、さらに `returns` （Kotlin の `return` ではないことに注意してください）、モックオブジェクトの関数に返させたい値、というように続けて書きます。具体的には、以下のとおりです。

```kotlin
every { dice.roll() } returns 1
```

これで、モックオブジェクトの `dice` に対して `roll` を呼び出した場合に、返り値として 1 を返してくれるようになりました。これを利用したテストを書いてみます。

ここで、Fortune クラスの `draw` 関数は、`dice` の `roll` 関数が 1 を返す時に Good を返します。これをテストコードで検証します。これを検証するためには、（Truth の isEqualTo を用いて）以下のように書くことができます。`actual` は実測値、`expected` は期待値です。

```kotlin
val actual = fortuneCookie.draw()
val expected = Fortune.Good

assertThat(actual)
  .isEqualTo(expected)
```

ここまでの内容をまとめると、以下のようなコードになります。これは、`dice` の `roll` 関数が 1 を返す時、FortuneCookie クラスのインスタンスの `draw` 関数は Good を返す、ということを検証するテストケースです。

```kotlin
@Test
fun draw_returnsGoodIfDiceRollIsOne() {
  every { dice.roll() } returns 1

  val actual = fortuneCookie.draw()
  val expected = Fortune.Good

  assertThat(actual)
    .isEqualTo(expected)
}
```

Dice interface の具象実装を行っているクラスがひとつも無いにも関わらず、Dice 型のインスタンスの `roll` 関数の返り値に依存する FortuneCookie#draw に対するテストコードを書くことができました。これが MockK によるモックオブジェクトを利用することで得られる力です。

テストコードを仕上げます。`dice` の `roll` 関数が 2 を返す時、及び 6 を返す時に `draw` 関数が Bad を返すことも検証しておきます（1 ～ 6 以外が返された時に FortuneException が throw されることのテストは、読者への課題としておきます）。

```kotlin
@Test
fun draw_returnsBadIfDiceRollIsTwo() {
  every { dice.roll() } returns 2

  val actual = fortuneCookie.draw()
  val expected = Fortune.Bad

  assertThat(actual)
    .isEqualTo(expected)
}

@Test
fun draw_returnsBadIfDiceRollIsSix() {
  every { dice.roll() } returns 6

  val actual = fortuneCookie.draw()
  val expected = Fortune.Bad

  assertThat(actual)
    .isEqualTo(expected)
}
```

本節のテストコードを以下にまとめます。

```kotlin
package com.okuzawats.mockk

import com.google.common.truth.Truth.assertThat
import io.mockk.MockKAnnotations
import io.mockk.every
import io.mockk.impl.annotations.MockK

import org.junit.After
import org.junit.Before
import org.junit.Test

class FortuneCookieTest {
  @MockK
  lateinit var dice: Dice

  lateinit var fortuneCookie: FortuneCookie

  @Before
  fun setUp() {
    MockKAnnotations.init(this)
    fortuneCookie = FortuneCookie(dice)
  }

  @After
  fun tearDown() {
  }

  @Test
  fun draw_returnsGoodIfDiceRollIsOne() {
    every { dice.roll() } returns 1

    val actual = fortuneCookie.draw()
    val expected = Fortune.Good

    assertThat(actual)
      .isEqualTo(expected)
  }

  @Test
  fun draw_returnsBadIfDiceRollIsTwo() {
    every { dice.roll() } returns 2

    val actual = fortuneCookie.draw()
    val expected = Fortune.Bad

    assertThat(actual)
      .isEqualTo(expected)
  }

  @Test
  fun draw_returnsBadIfDiceRollIsSix() {
    every { dice.roll() } returns 6

    val actual = fortuneCookie.draw()
    val expected = Fortune.Bad

    assertThat(actual)
      .isEqualTo(expected)
  }
}
```

## Kotlin Coroutines への対応

前節では、サイコロを表す Dice interface が登場しました。Dice interface の roll 関数はすぐに値を返してくれるものでしたが、実際のサイコロは、転がしてすぐに結果が出るものではなく、少しの時間をおいて回転が止まってから結果がでるものですね。この現実世界でのサイコロの挙動を考慮して、roll 関数を Kotlin Coroutines を用いた `suspend fun` として定義し直した AsyncDice のユニットテストについて考えます（前節の Dice interface と名前が同じになってしまうことを避けて、このような名前にしています）。

```kotlin
package com.okuzawats.mockk

/**
 * サイコロを表すinterface
 */
interface AsyncDice {
  /**
   * 1から6までの整数値をランダムに返す。
   */
  suspend fun roll(): Int
}
```

あわせて、FortuneCookie クラスの Kotlin Coroutines 対応版の AsyncFortuneCookie クラスを定義します。

```kotlin
package com.okuzawats.mockk

/**
 * 占いを行うClass
 *
 * @property dice 占いに必要なサイコロ
 */
class AsyncFortuneCookie(
  private val dice: AsyncDice,
) {
  /**
   * 占いを行い、結果を[Fortune]として返す
   *
   * @return [Fortune]
   * @throws [FortuneException]
   */
  suspend fun draw(): Fortune {
    // サイコロの出目
    val dots = dice.roll()

    // サイコロの出目が1ならGood、2から6ならBadを返す。
    // それ以外の出目が出た場合はFortuneExceptionをthrowする。
    return if (dots == 1) {
      Fortune.Good
    } else if(dots in 2..6) {
      Fortune.Bad
    } else {
      throw FortuneException("can not calculate fortune.")
    }
  }
}
```

次に、Coroutines 部分のテストコードを書いていきます。モックオブジェクトとテスト対象の準備については、前節の内容と変わりません。

```kotlin
@MockK
lateinit var dice: AsyncDice

lateinit var fortuneCookie: AsyncFortuneCookie

@Before
fun setUp() {
  MockKAnnotations.init(this)
  fortuneCookie = AsyncFortuneCookie(dice)
}
```

次にテスト部分ですが、テスト対象の関数が `suspend fun` であるため、今までのようにテストを書いても正しい結果が得られません。そこで、`kotlinx.coroutines.runBlocking` を用いるか、新たにテスト用のライブラリを追加して `kotlinx.coroutines.test.runTest` を用いるか、といった手段を取ることができます。ここでは後者の方法を用いますが、後者の方法は、執筆時現在、まだ Experimental な API であることに注意してください。

`app/build.gradle` に以下の記述を加え、テスト用のライブラリを追加します。

```groovy
dependencies {
  testImplementation "org.jetbrains.kotlinx:kotlinx-coroutines-test:1.6.4"
}
```

次に、テストケース全体を `runTest` で囲みます。関数名と `runTest` の間に `=` があることに注意してください。また、`runTest` ではなく `runBlocking` を用いる場合も書き方は同じです。

```kotlin
@Test
fun draw_returnsGoodIfDiceRollIsOne() = runTest {
  // TODO
}
```

MockK を用いて `suspend fun` をスタブ化するためには、前節で用いた `every` ではなく、`coEvery` を用います。MockK の使い方で異なるのは、この点のみです。

```kotlin
coEvery { dice.roll() } returns 1
```

テストケースの全体は以下のようになります。

```kotlin
@Test
fun draw_returnsGoodIfDiceRollIsOne() = runTest {
  coEvery { dice.roll() } returns 1

  val actual = fortuneCookie.draw()
  val expected = Fortune.Good

  assertThat(actual)
    .isEqualTo(expected)
}
```

本節のテストコードを以下にまとめます。

```kotlin
package com.okuzawats.mockk

import com.google.common.truth.Truth.assertThat
import io.mockk.MockKAnnotations
import io.mockk.coEvery
import io.mockk.impl.annotations.MockK
import kotlinx.coroutines.ExperimentalCoroutinesApi
import kotlinx.coroutines.test.runTest

import org.junit.After
import org.junit.Before
import org.junit.Test

@ExperimentalCoroutinesApi
class AsyncFortuneCookieTest {
  @MockK
  lateinit var dice: AsyncDice

  lateinit var fortuneCookie: AsyncFortuneCookie

  @Before
  fun setUp() {
    MockKAnnotations.init(this)
    fortuneCookie = AsyncFortuneCookie(dice)
  }

  @After
  fun tearDown() {
  }

  @Test
  fun draw_returnsGoodIfDiceRollIsOne() = runTest {
    coEvery { dice.roll() } returns 1

    val actual = fortuneCookie.draw()
    val expected = Fortune.Good

    assertThat(actual)
      .isEqualTo(expected)
  }

  @Test
  fun draw_returnsBadIfDiceRollIsTwo() = runTest {
    coEvery { dice.roll() } returns 2

    val actual = fortuneCookie.draw()
    val expected = Fortune.Bad

    assertThat(actual)
      .isEqualTo(expected)
  }

  @Test
  fun draw_returnsBadIfDiceRollIsSix() = runTest {
    coEvery { dice.roll() } returns 6

    val actual = fortuneCookie.draw()
    val expected = Fortune.Bad

    assertThat(actual)
      .isEqualTo(expected)
  }
}
```

## 部分引数マッチング

MockK の `every` では、モックオブジェクトの関数呼び出しに対して、引数がマッチした時に特定の値を返すようにできます。

以下のような Computer interface のモックオブジェクトを作ることを考えます。`isAnswer` は、Int 型の引数をとり、Boolean 型の返り値を返します。

```kotlin
/**
 * コンピュータの動作を表すinterface
 */
interface Computer {
  /**
   * [int]が答えと等しいときにtrueを返す
   */
  fun isAnswer(int: Int): Boolean
}
```

次に、以下のような Human クラスを考えます。Human クラスは、Computer 型のコンストラクタ引数をとります。Human クラスの `ask` 関数は Int 型の引数を取り、その引数を `computer.isAnswer(int)` に渡し、返される Boolean 型の値に応じた文字列を返します。

```kotlin
/**
 * 人間を表すクラス
 *
 * @property computer 人間の持つコンピュータ
 */
class Human(
  private val computer: Computer,
) {
  /**
   * [number]に対するComputerの答えがtrueなら"correct answer"、
   * falseなら"wrong answer"を返す。
   */
  fun ask(
    number: Int,
  ): String =
    if (computer.isAnswer(number)) {
      "correct answer"
    } else {
      "wrong answer"
    }
}
```

`every` の部分引数マッチングを用いて、Human クラスのテストを書く際に Computer 型のモックオブジェクトを渡し、特定の引数で `isAnswer` を呼んだ時にのみ true を返すようにしてみます。以下のように書けます。

```kotlin
every { computer.isAnswer(any()) } returns false
every { computer.isAnswer(42) } returns true
```

上記のコードでは、引数として 42 を受け取ったときに true 、それ以外の値であれば false を返すようにしています。すなわち、引数が 42 にマッチした時は true を返しています。

この状態で、モックオブジェクトが意図した通りに動いていることを確かめてみます。

```kotlin
  @Test
  fun ask_returnsWrongAnswerAskedWith0() {
    assertThat(human.ask(0))
      .isEqualTo("wrong answer")
  }

  @Test
  fun ask_returnsWrongAnswerAskedWith100() {
    assertThat(human.ask(100))
      .isEqualTo("wrong answer")
  }

  @Test
  fun ask_returnsCorrectAnswerAskedWith42() {
    assertThat(human.ask(42))
      .isEqualTo("correct answer")
  }
```

ただし、上記のテストコードはあまり良いテストコードではありません。Human クラスは、Computer#isAnswer が true を返すのか、または false を返すのかには興味はありますが、引数にどのような整数値を渡したら true が返るのか、ということには興味を持たないはずです。上記のコードはあくまで、MockK の挙動を知るためのコードであるということを念頭に置いていただければ幸いです。

本節のテストコードをまとめると以下のようになります。

```kotlin
import com.google.common.truth.Truth.assertThat
import io.mockk.MockKAnnotations
import io.mockk.every
import io.mockk.impl.annotations.MockK
import org.junit.After
import org.junit.Before
import org.junit.Test

class HumanTest {
  @MockK
  lateinit var computer: Computer

  lateinit var human: Human

  @Before
  fun setUp() {
    MockKAnnotations.init(this)

    // Computer#isAnswerの引数が42の時はtrue、それ以外の時はfalseを返す
    every { computer.isAnswer(any()) } returns false
    every { computer.isAnswer(42) } returns true

    human = Human(computer)
  }

  @After
  fun tearDown() {
  }

  @Test
  fun ask_returnsWrongAnswerAskedWith0() {
    assertThat(human.ask(0))
      .isEqualTo("wrong answer")
  }

  @Test
  fun ask_returnsWrongAnswerAskedWith100() {
    assertThat(human.ask(100))
      .isEqualTo("wrong answer")
  }

  @Test
  fun ask_returnsCorrectAnswerAskedWith42() {
    assertThat(human.ask(42))
      .isEqualTo("correct answer")
  }
}
```

## ネストされたスタブ

「スタブ化」の節では、`@MockK` アノテーションを用いてスタブ化を行いました。MockK では他にもスタブ化する方法が用意されています。それが `mockk` 関数です。

`mockk` 関数は、以下のコードのように書いた場合は型推論され SomeMock 型のモックオブジェクトを返します。

```kotlin
val someMock: SomeMock = mockk()
```

または、以下のコードのように型引数を用いてモックオブジェクトを生成する方法もあります。

```kotlin
val someMock = mockk<SomeMock>()
```

本節では、以下のようなクラスのテストを考えます。コンストラクタ引数として `ProviderFactory` 型を受け取ります。

```kotlin
/**
 * 値を返す関数の集まり
 */
class Values(
  private val providerFactory: ProviderFactory,
) {
  /**
   * Int型の値を返す
   */
  fun intValue(): Int =
    providerFactory
      .intProvider()
      .get()

  /**
   * String型の値を返す
   */
  fun stringValue(): String =
    providerFactory
      .stringProvider()
      .get()
}
```

さらにこの `ProviderFactory` 型は、intProvider 関数が IntProvider 型を、stringProvider 関数が StringProvider 型を返します。

```kotlin
interface ProviderFactory {
  fun intProvider(): IntProvider
  fun stringProvider(): StringProvider
}
```

IntProvider 型は Int 型の、StringProvider 型は String 型の値を返す関数をそれぞれ持ちます。

```kotlin
interface IntProvider {
  fun get(): Int
}
```

```kotlin
interface StringProvider {
  fun get(): String
}
```

設計上の是非は置いておくとして、このような少し複雑な状況でのテストを考えます。

まず、Values クラスをインスタンス化する際、引数に `mockk` 関数の返すモックオブジェクトを渡すようにしてみましょう。

```kotlin
val values = Values(
  providerFactory = mockk<ProviderFactory>()
)
```

`providerFactory` は、IntProvider 型を返す `intProvider` 関数と、StringProvider 型を返す `stringProvider` 関数を持ちます。これらもモックオブジェクトを返すようにしたいのですが、`providerFactory` は private なため、 `intProvider` 関数と `stringProvider` 関数のそれぞれの返り値を簡単にモックオブジェクトに差し替えることができません。

そこで、Values のコンストラクタ呼び出し時に一工夫して、あらかじめモックオブジェクトを用意しておきます。

```kotlin
val values = Values(
  providerFactory = mockk<ProviderFactory> {
    every {
      intProvider()
    } returns mockk<IntProvider> {
      every {
        get()
      } returns 42
    }

    every {
      stringProvider()
    } returns mockk<StringProvider> {
      every {
        get()
      } returns "Hello World"
    }
  }
)
```

こういったテクニックが必要になることはあまり多くないかもしれませんが、MockK のモックオブジェクトはネストすることが可能である、ということを覚えておくと役に立つことがあります。

本節のテストコードを以下にまとめます。

```kotlin
package com.okuzawats.mockk

import com.google.common.truth.Truth.assertThat
import io.mockk.every
import io.mockk.mockk
import org.junit.After
import org.junit.Before
import org.junit.Test

class ValuesTest {
  lateinit var values: Values

  @Before
  fun setUp() {
    values = Values(
      providerFactory = mockk<ProviderFactory> {
        every {
          intProvider()
        } returns mockk<IntProvider> {
          every {
            get()
          } returns 42
        }

        every {
          stringProvider()
        } returns mockk<StringProvider> {
          every {
            get()
          } returns "Hello World"
        }
      }
    )
  }

  @After
  fun tearDown() {
  }

  @Test
  fun test_intValue() {
    val actual = values.intValue()
    val expected = 42

    assertThat(actual)
      .isEqualTo(expected)
  }

  @Test
  fun test_stringValue() {
    val actual = values.stringValue()
    val expected = "Hello World"

    assertThat(actual)
      .isEqualTo(expected)

  }
}
```

## relaxed

モックオブジェクトの関数が返す値に関心がない場合、いちいち `every` を呼び出してスタブ化するのは面倒ですし、テストコードの意図がわかりにくくなってしまいます。

MockK には、relaxed という機能があります。モックオブジェクトの生成時に relaxed を設定しておくと、`every` を呼び出してスタブ化しなくても、自動的に適当な値を返してくれるようになります。

以下の例は、relaxed を設定していないテストコードです。この場合は、モックオブジェクトの `intProvider` の get 関数がスタブ化されていませんので、モックオブジェクトから値が返されず、テストは失敗します。

```kotlin
// このテストは失敗します
@Test
fun test_not_relaxed() {
  val intProvider: IntProvider = mockk()

  assertThat(intProvider.get())
    .isLessThan(Int.MAX_VALUE)
}
```

これに対して、以下の例ではモックオブジェクトの生成時に relaxed を設定しています。この場合、モックオブジェクトの `intProvider` の get 関数が自動的にスタブ化され、適当な値が返されるようになります。そのため、このテストは成功します。

```kotlin
@Test
fun test_relaxed() {
  val intProvider: IntProvider = mockk(relaxed = true)

  assertThat(intProvider.get())
    .isLessThan(Int.MAX_VALUE)
}
```

relaxed を乱用すると意味のないテストコードにつながる場合もあるので注意が必要ですが、relaxed を正しく活用できれば、効果的なテストコードを書くことができるようになります。

## object のスタブ化

このような object を考えます。

```kotlin
import kotlin.random.Random

/**
 * 信号機の色をランダムに生成する
 */
object SignalGenerator {
  /**
   * 信号機の色をランダムに生成して返す
   */
  fun randomColor(): String {
    // 0 - 2 の乱数を返す。
    val int = Random.nextInt(from = 0, until = 3)

    return when (int) {
      0 -> "Green"
      1 -> "Yellow"
      2 -> "Red"
      else -> throw Throwable()
    }
  }
}
```

信号機を表す Signal クラスでは、現在の信号機の色を表す `currentColor` を上記の SignalGenerator を用いて初期化しています。

```kotlin
/**
 * 信号機
 */
class Signal {
  /**
   * 現在の信号の色
   */
  val currentColor: String =
    SignalGenerator.randomColor()
}
```

このコードは、`currentColor` の初期値をテストしようとすると大変です。外部から SignalGenerator の挙動を差し替えることができないためです。設計を改善する、というのが正攻法だとは思いますが、今回は MockK の力を借りてこのまま Signal クラスのテストを書いてみます。

MockK には、非常に強力な機能が実装されています。Kotlin の object をモック化する機能です。それには、`mockkObject` を用います。

```kotlin
mockkObject(SignalGenerator)
```

JUnit 4 の場合、`@Before` の付与されたセットアップ関数は、`mockkObject` を呼び出すのに適当です。

```kotlin
@Before
fun setUp() {
  mockkObject(SignalGenerator)
}
```

あわせて、`@After` の付与されたティアダウン関数で、モック化を解除しておきます。

```kotlin
@After
fun tearDown() {
  unmockkAll()
}
```

これで、`SignalGenerator` をモックすることができるようになります。実際にテストコードを書いてみます。

```kotlin
  @Test
  fun test_green() {
    every {
      SignalGenerator.randomColor()
    } returns "Green"

    val signal = Signal()

    assertThat(signal.currentColor)
      .isEqualTo("Green")
  }
```

今回のテストコードの場合、SignalGenerator をスタブ化せずとも、ユニットテストがたまたま成功してしまう確率があります。心配な方は、複数回テストを回してみて、必ずテストが成功することを確かめましょう。

本節のテストコードを以下にまとめます。

```kotlin
package com.okuzawats.mockk

import com.google.common.truth.Truth.assertThat
import io.mockk.every
import io.mockk.mockkObject
import io.mockk.unmockkAll
import org.junit.After
import org.junit.Before
import org.junit.Test

class SignalTest {

  @Before
  fun setUp() {
    mockkObject(SignalGenerator)
  }

  @After
  fun tearDown() {
    unmockkAll()
  }

  @Test
  fun test_green() {
    every {
      SignalGenerator.randomColor()
    } returns "Green"

    val signal = Signal()

    assertThat(signal.currentColor)
      .isEqualTo("Green")
  }

  @Test
  fun test_yellow() {
    every {
      SignalGenerator.randomColor()
    } returns "Yellow"

    val signal = Signal()

    assertThat(signal.currentColor)
      .isEqualTo("Yellow")
  }

  @Test
  fun test_red() {
    every {
      SignalGenerator.randomColor()
    } returns "Red"

    val signal = Signal()

    assertThat(signal.currentColor)
      .isEqualTo("Red")
  }
}
```

## 処理の呼び出しの検証

ここからは、テスト対象のコードが特定の処理を呼び出していることの検証について考えます。MockK では、このような場合に `verify` が利用できます。テスト対象のクラスとして、「スタブ化」の節で登場した FortuneCookie クラスを再度取り上げます。

FortuneCookie クラスでは、draw 関数内でサイコロを表す Dice クラスのインスタンスが出力するサイコロの出目に応じた結果を返すのでした。

```kotlin
/**
 * 占いを行うClass
 *
 * @property dice 占いに必要なサイコロ
 */
class FortuneCookie(
  private val dice: Dice,
) {
  /**
   * 占いを行い、結果を[Fortune]として返す
   *
   * @return [Fortune]
   * @throws [FortuneException]
   */
  fun draw(): Fortune {
    // サイコロの出目
    val dots = dice.roll()

    // サイコロの出目が1ならGood、2から6ならBadを返す。
    // それ以外の出目が出た場合はFortuneExceptionをthrowする。
    return if (dots == 1) {
      Fortune.Good
    } else if(dots in 2..6) {
      Fortune.Bad
    } else {
      throw FortuneException("can not calculate fortune.")
    }
  }
}
```

FortuneCokie クラスが、きちんとサイコロを振っていることを検証します。MockK の `verify` は、モックオブジェクトの処理が呼び出されていることを検証します。

```kotlin
fortuneCookie.draw()

verify {
  dice.roll()
}
```

サイコロをただ一回だけ振っていることも検証してみましょう。`verify` は、いくつかの引数をとることができます。そのうちのひとつが `exactly` です。`exactly` を用いた場合、検証対象の処理が呼ばれた回数についても検証できます。

```kotlin
verify(exactly = 1) {
  dice.roll()
}
```

`verify` には、他にも便利な引数があります。使用する際は、是非ドキュメントにあたってみてください。

本節のテストコードを以下にまとめます（「スタブ化」の節で書いたテストコードについては省略します）。

```kotlin
package com.okuzawats.mockk

import com.google.common.truth.Truth.assertThat
import io.mockk.MockKAnnotations
import io.mockk.every
import io.mockk.impl.annotations.MockK
import io.mockk.verify

import org.junit.After
import org.junit.Before
import org.junit.Test

class FortuneCookieTest {
  @MockK
  lateinit var dice: Dice

  lateinit var fortuneCookie: FortuneCookie

  @Before
  fun setUp() {
    MockKAnnotations.init(this)
    fortuneCookie = FortuneCookie(dice)
  }

  @After
  fun tearDown() {
  }

  @Test
  fun draw_callsRoll() {
    every { dice.roll() } returns 1

    fortuneCookie.draw()

    verify {
      dice.roll()
    }
  }

  @Test
  fun draw_callsRoll_exactlyOnce() {
    every { dice.roll() } returns 1

    fortuneCookie.draw()

    verify(exactly = 1) {
      dice.roll()
    }
  }
}
```

## 処理の呼び出しの検証の Kotlin coroutines 対応

FortuneCookie クラスの処理呼び出しについての検証が済みましたので、その Kotlin coroutines 対応版の AsyncFortuneCookie クラスについても同様の検証を行います。AsyncFortuneCookie クラスは、「Kotlin Coroutines への対応」の節に登場しました。

`every` に coroutines 対応版の `coEvery` が存在したように、`verify` にも coroutines 対応版の `coVerify` が存在します。`coVerify` を用いた検証は、以下のように行います。

```kotlin
coVerify { 
  dice.roll()
}
```

`verify` と同様に、`coVerify` にも便利な引数が存在します。実際に `coVerify` を用いる場合は、是非ドキュメントにあたってみてください。

本節のテストコードを以下にまとめます（「Kotlin Coroutines への対応」の節で書いたテストコードについては省略します）。

```kotlin
package com.okuzawats.mockk

import com.google.common.truth.Truth.assertThat
import io.mockk.MockKAnnotations
import io.mockk.coEvery
import io.mockk.coVerify
import io.mockk.impl.annotations.MockK
import kotlinx.coroutines.ExperimentalCoroutinesApi
import kotlinx.coroutines.test.runTest

import org.junit.After
import org.junit.Before
import org.junit.Test

@ExperimentalCoroutinesApi
class AsyncFortuneCookieTest {
  @MockK
  lateinit var dice: AsyncDice

  lateinit var fortuneCookie: AsyncFortuneCookie

  @Before
  fun setUp() {
    MockKAnnotations.init(this)
    fortuneCookie = AsyncFortuneCookie(dice)
  }

  @After
  fun tearDown() {
  }

  @Test
  fun draw_callsRoll = runTest {
    coEvery { dice.roll() } returns 1

    fortuneCookie.draw()

    coVerify {
      dice.roll()
    }
  }
}
```

## 処理の呼び出しの検証における、パラメータの検証

本節では、前節の内容を発展させ、さらに処理を呼び出す際のパラメータについても検証します。ここでは、「部分引数マッチング」の節で登場した、Human クラスを再び考えます。

```kotlin
/**
 * 人間を表すクラス
 *
 * @property computer 人間の持つコンピュータ
 */
class Human(
  private val computer: Computer,
) {
  /**
   * [number]に対するComputerの答えがtrueなら"correct answer"、
   * falseなら"wrong answer"を返す。
   */
  fun ask(
    number: Int,
  ): String =
    if (computer.isAnswer(number)) {
      "correct answer"
    } else {
      "wrong answer"
    }
}
```

Human クラスの ask 関数は、受け取った引数を `computer` の isAnswer 関数に渡しています。Human クラスのインスタンスが ask 関数を呼ばれた時、引数で受け取った Int 型の値を加工せず、そのまま computer の isAnswer 関数に渡していることを検証してみます。

MockK を用いた場合、この検証は簡単です。`verify` を呼び出す際に、適切な引数を渡します。引数のマッチする処理がテストケース内で呼ばれていた場合は、テストに成功します。

```kotlin
human.ask(42)

verify {
  computer.isAnswer(42)
}
```

本節のテストコードを以下にまとめます（「部分引数ママッチング」の節で書いたテストコードについては省略します）。

```kotlin
package com.okuzawats.mockk

import com.google.common.truth.Truth.assertThat
import io.mockk.MockKAnnotations
import io.mockk.every
import io.mockk.impl.annotations.MockK
import io.mockk.verify
import org.junit.After
import org.junit.Before
import org.junit.Test

class HumanTest {
  @MockK
  lateinit var computer: Computer

  lateinit var human: Human

  @Before
  fun setUp() {
    MockKAnnotations.init(this)

    // Computer#isAnswerの引数が42の時はtrue、それ以外の時はfalseを返す
    every { computer.isAnswer(any()) } returns false
    every { computer.isAnswer(42) } returns true

    human = Human(computer)
  }

  @After
  fun tearDown() {
  }

  @Test
  fun ask_callsIsAnswer_withReceivedParameter() {
    human.ask(42)

    verify {
      computer.isAnswer(42)
    }
  }
}
```

## 発展的な内容

以下の記事では、Hiltを用いてMockKのスパイをテスト用のオブジェクトとして注入して、検証する方法を説明しています。

- [[Android] 単体テスト用の依存関係をHiltで解決し、かつ例外のテストを行いたい場合のパターン](https://okuzawats.com/blog/test-dependency-by-hilt-and-testing-exception/)
