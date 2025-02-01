---
title: "Truthのカスタムサブジェクトを定義する"
date: 2023-03-25T23:24:08+09:00
description: "アサーションライブラリTruthのカスタムサブジェクトを定義する方法です。"
categories: ["Android", "Test"]
draft: false
---

Truthのカスタムサブジェクトを新たに定義して、独自のアサーションを追加してみます。[公式の解説](https://truth.dev/extension.html)を参考に進めました。Javaで書かれたサンプルコードをKotlinに読み替えるのに若干の苦労をしましたが、なんとかなりました。

今回は、サンプルとしてKotlinのUnit型に対するアサーションを追加します。つまり、以下のようなアサーションができるようにします。

```kotlin
@Test
fun test_isNotUnit() {
  val actual: Unit? = null
  assertThat(actual).isNotUnit()
}

@Test
fun test_isUnit() {
  val actual: Unit? = Unit
  assertThat(actual).isUnit()
}
```

余談ですが、KotlinのUnitのインスタンスをJavaで使うには、 `Unit.INSTANCE` と書くことを知りました。またひとつ賢くなりました。

## カスタムサブジェクトの定義

`com.google.common.truth.Subject` （以下Subjectと呼ぶ）を継承して、カスタムサブジェクトを作ります。Subjectのコンストラクタは、引数としてFailureMetadataと検査対象となるObjectをとります。カスタムサブジェクトのコンストラクタ引数もこれに準じます。コンストラクタはprivateにしておきます（理由は後述します）。

```kotlin
class UnitSubject private constructor(
  metadata: FailureMetadata,
  private val actual: Unit?,
) : Subject(metadata, actual) {
  // TODO
}
```

次に、 `assertThat` をstaticな関数として定義します。 `assertThat` は検査対象となる値（actual）を引数にとります。この中で呼んでいる `assertAbout` はSubjectのFactoryを引数にとり、SubjectBuilderを返します。さらにSubjectBuilderに対して `that` を呼び出し、actualを渡すことでSubjectが作られます。ここでは独自に定義したUnitSubjectを返しています（ `assertThat` を経由してUnitSubjectのインスタンスを作りたいため、コンストラクタをprivateにしています）。

```kotlin
class UnitSubject private constructor(
  metadata: FailureMetadata,
  private val actual: Unit?,
) : Subject(metadata, actual) {

  // TODO

  companion object {
    /**
     * start assertion of an object with type Unit?
     */
    fun assertThat(actual: Unit?): UnitSubject = assertAbout(unit()).that(actual)

    private fun unit(): Factory<UnitSubject, Unit?> = Factory(::UnitSubject)
  }
}
```

UnitSubjectにアサーションを実装します。今回はSubjectに生えている `isEqualTo` と `isNotEqualTo` を使うだけで済みました。

```kotlin
class UnitSubject private constructor(
  metadata: FailureMetadata,
  private val actual: Unit?,
) : Subject(metadata, actual) {

  /**
   * assert that actual is Unit.
   */
  fun isUnit() = unit().isEqualTo(Unit)

  /**
   * assert that actual is NOT Unit.
   */
  fun isNotUnit() = unit().isNotEqualTo(Unit)

  private fun unit(): Subject = check("unit()").that(actual)

  // 省略
}
```

## カスタムサブジェクトを用いたアサーション

カスタムサブジェクトを用いてアサーションをしてみます。

```kotlin
package com.okuzawats.truth

import com.okuzawats.truth.UnitSubject.Companion.assertThat
import org.junit.Test

class UnitTest {
  @Test
  fun test_isNotUnit() {
    val actual: Unit? = null
    assertThat(actual).isNotUnit()
  }

  @Test
  fun test_isUnit() {
    val actual: Unit? = Unit
    assertThat(actual).isUnit()
  }
}
```

動きました🎉

## ソースコード

Truthのバージョンは1.1.3を用いています。

```kotlin
package com.okuzawats.truth

import com.google.common.truth.FailureMetadata
import com.google.common.truth.Subject
import com.google.common.truth.Subject.Factory
import com.google.common.truth.Truth.assertAbout

class UnitSubject private constructor(
  metadata: FailureMetadata,
  private val actual: Unit?,
) : Subject(metadata, actual) {

  /**
   * assert that actual is Unit.
   */
  fun isUnit() = unit().isEqualTo(Unit)

  /**
   * assert that actual is NOT Unit.
   */
  fun isNotUnit() = unit().isNotEqualTo(Unit)

  private fun unit(): Subject = check("unit()").that(actual)

  companion object {
    /**
     * start assertion of an object with type Unit?
     */
    fun assertThat(actual: Unit?): UnitSubject = assertAbout(unit()).that(actual)

    private fun unit(): Factory<UnitSubject, Unit?> = Factory(::UnitSubject)
  }
}
```

GitHubのpublicリポジトリにもソースコードを公開します。

- [okuzawats/android-truth-samples: アサーションライブラリ Truth の使い方のサンプル](https://github.com/okuzawats/android-truth-samples)

## References

1. [Extensions to Truth](https://truth.dev/extension.html) （最終アクセス日：2023年3月25日）
2. [okuzawats/android-truth-samples: アサーションライブラリ Truth の使い方のサンプル](https://github.com/okuzawats/android-truth-samples) （最終アクセス日：2023年3月25日）
