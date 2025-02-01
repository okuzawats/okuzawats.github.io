---
title: "[Android] Uniflowを用いたMVI的なアーキテクチャを試してみる🦄"
date: 2023-08-05T22:54:20+09:00
description: "単一方向のデータフローをサポートするライブラリ「Uniflow」を味見してみます。"
categories: ["Android", "Design"]
draft: false
---

MVI的な単一方向のデータフローをサポートするライブラリ[Uniflow](https://github.com/uniflow-kt/uniflow-kt)を（しばらく前に）試してみました。その概要は[What is Uniflow?](https://github.com/uniflow-kt/uniflow-kt/blob/master/doc/what.md)に詳しいです。

> Uniflow 🦄 - Simple Unidirectional Data Flow for Android & Kotlin, using Kotlin coroutines and open to functional programming
> https://github.com/uniflow-kt/uniflow-kt

ここ1年ほどはコミットが進んでないようです。最新バージョンは1.1.2で、1.1.1とかなり違うような...。

```groovy
dependencies {
  implementation 'org.uniflow-kt:uniflow-android:1.1.2'
}
```

## Uniflowを味見してみる

`io.uniflow.android.AndroidDataFlow` を継承したクラスを作ります。便宜上ViewModelという名前を付けていましたが、公式のサンプルでは「〜DataFlow」という命名をしているようです。なお、AndroidDataFlowはAACの `androidx.lifecycle.ViewModel` を継承しているので、実装上は同じように扱うことができます。例えば、Hiltを用いたDIには `@HiltViewModel` を用いることができるということです。

```kotlin
import io.uniflow.android.AndroidDataFlow
import io.uniflow.core.flow.data.UIEvent
// 省略

class MainViewModel : AndroidDataFlow(
  defaultState = MainViewModelState.Initial,
) {
  // TODO
}
```

AndroidDataFlowを継承したクラス内では、`action` 内で `setState` を用いた状態の更新を、`sendEvent` を用いたOne Shot Eventの発火を行うことができます。Viewからアクションがトリガーされ、Stateの更新・Eventの発火処理が行われます。

```kotlin
fun onSomeAction() {
  action {
    setState(MainViewModelState.Loading)

    getSomeFlowUseCase()
      .map(/* map it here */)
      .onEach(::setState)
      .launchIn(viewModelScope)
  }
}

fun onAnotherAction() {
  action {
    sendEvent(MainViewModelEvent.SomeEvent)
  }
}
```

Stateについては `io.uniflow.core.flow.data.UIState` のサブクラスとします。sealed classなどでUIのStateをモデル化してあげると良さそうです。

```kotlin
import io.uniflow.core.flow.data.UIState

sealed class MainViewModelState : UIState() {
  object Initial : MainViewModelState()

  object Loading : MainViewModelState()

  data class ShowImage(val image: String) : MainViewModelState()

  object LoadFailed : MainViewModelState()
}
```

同様にEventについては `io.uniflow.core.flow.data.UIEvent` のサブクラスとします。

```kotlin
import io.uniflow.core.flow.data.UIEvent

sealed class MainViewModelEvent : UIEvent(){
  object SomeEvent : MainViewModelEvent()
}
```

View側では、LifecycleOwnerやFragmentに `onState` 、`onEvent` が生えていますので、UIの更新は `onState` で、One Shot Eventに対する処理の実行は `onEvent` で行います。ここでいうStateやEventの実態はLiveDataです。Kotlin Coroutines Flowをサポートしようとした痕跡をソースコードから読み取りましたが、どうも頓挫したように思われます。

```kotlin
onState(viewModel) { state ->
  // TODO
}
```

```kotlin
onEvents(viewModel) { event ->
  // TODO
}
```

## まとめ

Uniflowを用いた、MVI的なアーキテクチャを味見してみました。AndroidではAndroidDataFlowとそれが公開するStateとEventを用いることとなりますが、その実態はAACのViewModelとLiveDataでした。アーキテクチャとしては美しいですね。

ただ、ライブラリの更新が止まってしまっていることもありますので、ライブラリに頼らず、自分でUniflowのやっていることに相当するコードを書いて使ってもいいかな？とは思いました。ライブラリのソースコードもそんなに量はないです。

ライブラリのソースコードは読んでいて面白かったです🦄
