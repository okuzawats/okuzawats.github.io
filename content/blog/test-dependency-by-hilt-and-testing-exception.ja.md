---
title: "[Android] 単体テスト用の依存関係をHiltで解決し、かつ例外のテストを行いたい場合のパターン🗡️"
date: 2023-11-21T05:48:00+09:00
description: "単体テスト用の依存関係をHiltで解決して、かつ例外のテストを行いたい時のパターンです。"
categories: ["Android", "Test"]
draft: false
---

[単体テスト用の依存関係をHiltで解決する](https://okuzawats.com/blog/test-dependency-by-hilt/)の続きです。Hiltで注入した依存オブジェクトが特定の状況でthrowする例外をcatchするケースのテストを書きたいので、MockKを使う方法を考えました。

MockKの使い方については、以下の記事もあわせて参照ください。

- [Mockk によるモック入門](https://okuzawats.com/blog/mockk/)

## はじめに

はじめに注意事項ですが、単体テスト用の依存関係をHiltで解決する理由は、単体テストにテストダブルを多用したくないためでした。本記事ではMockKを用いたテストダブルを活用しますが、例外のテストに用途を限定します。

また、テストダブルの中でもスパイを用いて、極力、依存オブジェクトを実オブジェクトに近いものにします。

## 依存性注入

まずは[単体テスト用の依存関係をHiltで解決する](https://okuzawats.com/blog/test-dependency-by-hilt/)でも書いたように、テスト用の依存関係を記述するHiltのモジュールを作成します。この辺りの説明については、上記の記事を参照してください。

```kotlin
@Module
@TestInstallIn(
  components = [SingletonComponent::class],
  replaces = [AwesomeDaoModule::class],
)
class SpyAwesomeDaoModule {
  // TODO
}
```

次に `@Provides` の付いたprovide〜関数を作成します。ここではMockKの `spyk` を利用して、実オブジェクトのスパイに置き換えます。スパイなので、テストで注入されるオブジェクトは実オブジェクトと同様に振る舞い、またMockKの機能を用いた検証を行うことができるようになります。

```kotlin
@Module
@TestInstallIn(
  components = [SingletonComponent::class],
  replaces = [AwesomeDaoModule::class],
)
class SpyAwesomeDaoModule {
  @Provides
  @Singleton
  fun provideAwesomeDao(database: AwesomeDatabase): AwesomeDao {
    return spyk(database.awesomeDao()) // スパイに置き換える。
  }
}
```

## テスト対象オブジェクトのセットアップ

テストで必要となる依存関係をHiltで解決します。以下の例では、テストで必要となる依存関係としてAwesomeDaoのオブジェクトを注入しています。このオブジェクトは、上述のモジュールで作成したスパイになります。

```kotlin
@HiltAndroidTest
@Config(
  application = HiltTestApplication::class,
  manifest = Config.NONE,
)
@RunWith(AndroidJUnit4::class)
class DefaultAwesomeRepositoryTest {

  @get:Rule
  var hiltRule = HiltAndroidRule(this)

  private lateinit var sut: AwesomeRepository

  @Inject
  lateinit var dao: AwesomeDao

  @Before
  fun setUp() {
    hiltRule.inject()

    sut = DefaultAwesomeRepository(dao)
  }

  @After
  fun tearDown() {
    // TODO
  }
}
```

## テストダブルを用いた例外のテスト

依存オブジェクトがMockKのスパイに置き換わっているので、MockKの `every-throws` を用いて例外の検証を行うことができます。

```kotlin
  @Test
  fun test_error() = runTest {
    every {
      dao.save(any())
    } throws SomeException()

    // TODO ここで例外発生時の検証を行う。
  }
```

## References

1. [単体テスト用の依存関係をHiltで解決する](https://okuzawats.com/blog/test-dependency-by-hilt/)
2. [Mockk によるモック入門](https://okuzawats.com/blog/mockk/)
