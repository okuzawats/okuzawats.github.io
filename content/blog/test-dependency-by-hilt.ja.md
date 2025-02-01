---
title: "[Android] 単体テスト用の依存関係をHiltで解決する"
date: 2023-06-29T17:52:00+09:00
description: "プロダクションとは異なる単体テストの依存関係をHiltで解決する方法です。"
categories: ["Android", "Test"]
draft: false
---

プロダクションとは異なる単体テストの依存関係をHiltで解決したい、ということってありますよね。僕はありました。

本記事では、単体テスト用の依存関係をHiltで解決する方法を示していきます。

## 単体テスト用の依存関係のセットアップ

Hiltのセットアップはできているものとして、さらに単体テスト用の依存関係を追加します。テスト用のHiltの依存関係の他、JUnit 4で使用するテストランナーとRobolectricを使います。

```groovy
// use Hilt in tests
testImplementation "com.google.dagger:hilt-android-testing:2.46.1"
kaptTest "com.google.dagger:hilt-android-compiler:2.46.1"

// TestRunner
testImplementation "androidx.test.ext:junit-ktx:1.1.5"

// Robolectric
testImplementation "org.robolectric:robolectric:4.10.3"
```

## テストクラスのセットアップ

テストクラスのセットアップにいくつかポイントがありますが、結論としては以下のように行いました。

```kotlin
import androidx.test.ext.junit.runners.AndroidJUnit4
import dagger.hilt.android.testing.HiltAndroidTest
import dagger.hilt.android.testing.HiltTestApplication
import org.junit.Test

import org.robolectric.annotation.Config
import javax.inject.Inject

@HiltAndroidTest
@Config(application = HiltTestApplication::class)
@RunWith(AndroidJUnit4::class)
class ExampleUnitTest {
  // 後述
}
```

ポイントは、以下です。

- `@HiltAndroidTest` を付与すること
  - プロダクションで使う `@AndroidEntryPoint` のようなアノテーションと同様に、Hiltが依存関係を解決するために必要です。
- `@Config` にHiltTestApplicationを指定すること
  - `@Config` 自体はRobolectricの仕組みです。
  - `@Config` にHiltが定義しているテスト用のApplicationクラスを指定しています。
- `@RunWith` にAndroidJUnit4を指定すること
  - `@RunWith` でテストランナーを指定します。[Robolectricとandroidx.test](https://okuzawats.com/blog/robolectric-vs-androidx-test/)で書いたように、ここで指定するのはRobolectricTestRunnerでも問題ありません。

## テストルールの設定

テストルールとしてHiltAndroidRuleを指定します。setupでHiltAndroidRule#injectを呼び出すことでInjectされます。

```kotlin
import dagger.hilt.android.testing.HiltAndroidRule

import org.junit.Before
import org.junit.Rule

@HiltAndroidTest
@Config(application = HiltTestApplication::class)
@RunWith(AndroidJUnit4::class)
class ExampleUnitTest {

  @get:Rule
  var hiltRule = HiltAndroidRule(this)

  @Inject lateinit var animal: Animal

  @Before
  fun setup() {
    hiltRule.inject()
  }
}
```

## Moduleの差し替え

`test` 内にテスト用のモジュールを定義して、`@TestInstallIn` でそのモジュールを指定します。これを行うことで、単体テストの依存解決時に使用されるモジュールを差し替えることができます。またここでは、単体テストの依存解決時に[Mockk](https://okuzawats.com/blog/mockk/)を利用したテストダブルを注入するように設定しています。

```kotlin
import dagger.Module
import dagger.Provides
import dagger.hilt.components.SingletonComponent
import dagger.hilt.testing.TestInstallIn
import io.mockk.every
import io.mockk.mockk

@Module
@TestInstallIn(
  components = [SingletonComponent::class],
  replaces = [AnimalModule::class]
)
class FakeAnimalModule {
  @Provides
  fun provideAnimal(): Animal {
    return mockk<Animal>().also {
      every { it.power() } returns 100
    }
  }
}
```

サンプルプロジェクトはGitHubに公開してあります。

- [okuzawats/android-hilt-testing: Hiltを単体テストに活用するための実験用プロジェクト](https://github.com/okuzawats/android-hilt-testing)

## テストダブルの活用

以下の記事では、例外のテストを行いたい場合にテストダブルを注入する方法を説明しています。

- [[Android] 単体テスト用の依存関係をHiltで解決し、かつ例外のテストを行いたい場合のパターン](https://okuzawats.com/blog/test-dependency-by-hilt-and-testing-exception/)
