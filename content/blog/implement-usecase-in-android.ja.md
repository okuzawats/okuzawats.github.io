---
title: "[Android] UseCaseの実装"
date: 2022-01-05T08:46:49+09:00
description: "Android Developersのアプリアーキテクチャガイドに示された、AndroidにおけるUseCase層の実装方針についてのメモです。"
categories: ["Android", "Design"]
aliases:
    - /posts/implement-usecase-in-android/
draft: false
---

昨年（2021年）末にAndroid Developersに掲載されているアプリアーキテクチャガイドが更新されていましたね。更新版の内容で参考になったことの一つが、UseCaseの実装についてでした。Domain layerに関する以下のページです。

- [Domain layer | Android Developers](https://developer.android.com/jetpack/guide/domain-layer)

ドメインとは何か、UseCaseとは何か、ということについては上記のリンクに譲ることにして、ここではその実装方法を見ていきたいと思います。

リンク先では、以下のようにUseCaseの `invoke` を実装することで、UseCaseがただ一つの機能のみを持っていることを強制できるようにしています。以下のサンプルコードは上記のリンク先のものです。

```kotlin
operator fun invoke(date: Date): String {
  return formatter.format(date)
}
```

例えばコーディング規約などでUseCaseは `invoke` のみを実装していることを定めれば、一つのUseCaseがただ一つの機能のみを提供していることが保証できるようになるかと思います[^a]。 `invoke` の引数と返り値は自由に定義できるので、実装上も不便はなさそうです。

UseCaseの命名は、アプリアーキテクチャガイドにあるように、「_動詞＋名詞＋UseCase_」というように命名します。例えば、LogOutUserUseCaseというような命名です。このように命名した場合は、インスタンスに対して `logOutUserUseCase()` というように書けば `invoke` に実装した処理を実行できます。この命名規則に従うことにより、このUseCaseが「ユーザーのログアウトを行う」という処理を実行していることがわかりやすくなると思います。

また、 `invoke` を実装する場合も、UseCaseのinterfaceを切ったり、 `suspend` を付けることもできます。例えばUseCaseの `invoke` でFlowを返す、という時にはこんな感じで実装できます。 `@Inject` はDaggerのアノテーションです。

```kotlin
interface LogOutUserUseCase {
  operator fun invoke(): Flow<LogOutState>
}
```

```kotlin
class LogOutUserUseCaseImpl @Inject constructor() : LogOutUserUseCase {
  override operator fun invoke(): Flow<LogOutState> =
    flow {
      // do something here
    }
}
```

`invoke` をsuspend funにする時はこんな感じで書けます。

```kotlin
suspend operator fun invoke(): AwesomeType
```

```kotlin
override suspend operator fun invoke(): AwesomeType {
  // do something here
}
```

UseCaseを実行する時は、UseCaseのインスタンスに `()` を付けて `invoke` を呼び出せば良いです。

```kotlin
viewModelScope.launch {
  logOutUserUseCase()
    .map(/* do something here */)
    .onEach(/* do something here */)
    .launchIn(this)
  }
}
```

UseCaseをいい感じに実装できて捗りそうです。

余談ですが、UseCaseを1関数にすることでKotlinの `fun interface` を適用して関数型インターフェースにできますので（参考：[[Kotlin] 関数定義が1つだけのinterfaceは `fun interface` で定義できる](https://okuzawats.com/blog/functional-interface/)）、こんな感じで書くこともできます。

```kotlin
fun interface SomeUseCase {
  operator fun invoke()
}
```

```kotlin
val useCase = SomeUseCase { /* Not yet implemented */ }
```

`fun interface` は関数定義が1つのみであることがコンパイラによって検証されますので、UseCaseが1関数のみであることが保証できて嬉しいかもしれません。

[^a]: 一つのUseCaseに複数の機能が生えてくることが稀によくある。

## Reference
1. [Domain layer | Android Developers](https://developer.android.com/jetpack/guide/domain-layer)（最終アクセス日：2022年1月5日）
2. [Operator overloading | Kotlin](https://kotlinlang.org/docs/operator-overloading.html#invoke-operator)（最終アクセス日：2022年1月5日）
