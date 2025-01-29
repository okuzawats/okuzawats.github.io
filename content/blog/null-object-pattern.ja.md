---
title: "[Kotlin] nullオブジェクトパターン"
date: 2022-02-07T22:48:00+09:00
description: "Kotlinによるnullオブジェクトパターンのサンプルコードです。"
categories: ["Kotlin", "Design", "プログラミング"]
draft: false
---

Kotlinでnullオブジェクトパターンを実装する例です。こんな感じのinterfaceを考えます。

```kotlin
interface AwesomeRunnable {
  fun runSomething()
}
```

## nullオブジェクトパターンを使わない場合

nullオブジェクトパターンを使わない場合は、こんな感じのifチェックを呼び出し側で毎回書かないといけません。対象の変数がnullなのかどうかということに興味がある場合は毎回nullチェックをするのもやぶさかではありません。

```kotlin
fun main() {
  val awesomeRunnable: AwesomeRunnable? = null
  if (awesomeRunnable != null) {
    awesomeRunnable.runSomething()
  }
}
```

## nullオブジェクトパターンを使う場合

対象の変数がnullなのかどうかということに興味がない場合もあります。こういう場合はnullオブジェクトパターンの使用に一考の価値があります。

以下のコードは「何かを行う」関数を呼び出しています。

```kotlin
fun main() {
  val doSomethingRunnable: AwesomeRunnable = object : AwesomeRunnable {
    override fun runSomething() {
      println("do something")
    }
  }

  doSomethingRunnable.runSomething()
}
```

これに対して、以下のコードはnullオブジェクトパターンを用いた「何もしない」関数を呼び出しています。この場合は、インスタンスに対して `runSomething` 関数を呼び出しても、実際には何の処理も行われません[^1][^2]。

```kotlin
fun main(args: Array<String>) {
  val doNothingRunnable: AwesomeRunnable = object : AwesomeRunnable {
    override fun runSomething() {
      // NOP
    }
  }

  doNothingRunnable.runSomething()
}
```

nullオブジェクトパターンを用いたことで、「何かを行う」場合も「何もしない」場合も、処理を呼び出す側ではnullチェックが不要となっています。このように、オブジェクトがnullかどうかということには関心を持ちたくないが、実際には処理を呼び出した時に何もしないで欲しいことがある、という場合にnullオブジェクトパターンが使えることがあります。

上記の例ではオブジェクト式を用いた無名クラスでnullオブジェクトを作成していますが、以下のようにnullオブジェクトをひとつだけ定義しておけば、コードの重複を防ぐことができます[^3]。

```kotlin
object NullRunnable : AwesomeRunnable {
  override fun runSomething() {
    // NOP
  }
}
```

[^1]: `Unit` を返す、という処理は行われます。
[^2]: Kotlinの `object` キーワードを用いていますが、nullオブジェクトの「オブジェクト」とはあまり関係がありません。
[^3]: この例でもKotlinの `object` キーワードを用いていますが、こちらもnullオブジェクトの「オブジェクト」とはあまり関係がありません。
