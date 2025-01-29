---
title: "[Android] Fragmentに値を渡す"
description: "AndroidのFragmentについて、Bundleを用いて値を渡す方法とNavigation Safe Argsを用いて値を渡す方法についてまとめます。"
date: 2019-09-28T21:41:10+09:00
categories: ["Android"]
aliases:
    - /posts/pass-value-to-fragment/
    - /blog/pass-value-with-navigation-safe-args/
draft: false
---

Navigation Safe Argsを使わない場合と使う場合に分けて書きます。

## Navigation Safe Argsを使わず、Fragmentに値を渡す

Navigation Safe Argsを使わずにFragmentに値を渡す時は、Bundleを使って渡します。Fragmentのインスタンスを生成するstaticなメソッドを作るのは良くあるパターンだと思いますが、この時にFragmentのargumentsにBundleをセットします。

```kotlin
companion object {
    private const val ARG_TEXT = "arg_text"

    fun newInstance(text: String) = MainFragment()
        .apply {
            arguments = Bundle().apply {
                putString(ARG_TEXT, text)
            }
        }
}
```

Bundleから値を取り出す時は、 `by lazy` の中で取り出すのが個人的には気に入っています。

```kotlin
private val text: String by lazy {
    arguments?.getString(ARG_TEXT) ?: ""
}
```

必ずFragmentに値が渡ってくるのであれば、 `arguments` の代わりに `requireArguments` が使えます。つまり、 `requireArguments().getString(ARG_TEXT)` というように書けます。

## Navigation Safe Argsを使って値を渡す

Navigation Safe Argsを使って値を渡す方法について以下に示します。

`app/build.gradle` に追加します。

```groovy
apply plugin: 'androidx.navigation.safeargs.kotlin'
```

`build.gradle` にも追加します。

```groovy
buildscript {
        // Navigation Safe Args
        def nav_version = "2.1.0"
        classpath "androidx.navigation:navigation-safe-args-gradle-plugin:$nav_version"
    }
}
```

これだけだと「*Cannot inline bytecode built with JVM target 1.8 into bytecode that is being built with JVM target 1.6*」みたいな感じで怒られるので、さらに `app/build.gradle` に以下を追加します。

```groovy
android {
    compileOptions {
        sourceCompatibility = 1.8
        targetCompatibility = 1.8
    }
    kotlinOptions {
        jvmTarget = "1.8"
    }
}
```

Navigation Graphに `argument` を追加して、名前と型を指定します。 `android:defaultValue="John Doe"` のようにすればデフォルト値が設定できます。

```xml
<fragment
        android:id="@+id/output_fragment"
        android:name="com.okuzawats.helloworld.fragment.OutputFragment"
        android:label="fragment_output"
        tools:layout="@layout/fragment_output">

    <argument
        android:name="name"
        app:argType="string"/>

</fragment>
```

遷移元では、Actionに対象となる引数を渡します。一回、ビルドすると必要なクラスが生成されるかと思います。

```kotlin
val inputText = input_edit_text.text.toString()
val action = InputFragmentDirections.actionInputToOutput(inputText)
findNavController().navigate(action)
```

遷移先では、 `navArgs()` で `argument` を取得し、プロパティ名で中に入っている値を取り出します。

```kotlin
private val args: OutputFragmentArgs by navArgs()

private val name: String by lazy {
    args.name
}
```
