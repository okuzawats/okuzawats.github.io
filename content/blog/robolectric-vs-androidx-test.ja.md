---
title: "[Android] Robolectricとandroidx.test"
date: 2022-08-03T22:03:45+09:00
description: "Robolectricとandroidx.testについて混乱しないためのメモです。"
categories: ["Android", "Test"]
draft: false
---

Robolectricと `androidx.test` のどちらを使えばいいのか混乱してしまったので、情報を整理します。

わかりやすい例としてSharedPreferenceを取り上げます。SharedPreferenceへのアクセサを提供する、こんな感じのクラスを考えます。SharedPreferenceへの文字列の読み書きをしているだけのクラスです。

```kotlin
import android.content.SharedPreferences
import androidx.core.content.edit

private const val KEY_SOME_STRING = "KEY_SOME_STRING"

class PreferenceDataSource(
  private val preference: SharedPreferences,
) {
  var someString: String
    get() = preference.getString(KEY_SOME_STRING, "")!!
    set(value) {
      preference.edit {
        putString(KEY_SOME_STRING, value)
      }
    }
}
```

一昔前は、Robolectric[^1]のRuntimeEnvironmentを使ってテストを書いてましたね。

```groovy
testImplementation("org.robolectric:robolectric:4.8")
```

```kotlin
import androidx.preference.PreferenceManager  
import org.junit.Test  
import org.junit.runner.RunWith  
import org.robolectric.RobolectricTestRunner  
import org.robolectric.RuntimeEnvironment  
  
@RunWith(RobolectricTestRunner::class)  
class PreferenceDataSourceTest {  
  @Test  
  fun test() {  
    val preferenceDataSource = PreferenceDataSource(  
      PreferenceManager.getDefaultSharedPreferences(  
        RuntimeEnvironment.getApplication().applicationContext  
      )  
    )  
  
    preferenceDataSource.someString = "hello world!"  
    assert(preferenceDataSource.someString == "hello world!")  
  }  
}
```

今は `androidx.test` [^2]を使って同様のテストを書くことができます。

```groovy
testImplementation("androidx.test:core-ktx:1.4.0")  
testImplementation("androidx.test.ext:junit:1.1.3")
```

```kotlin
import androidx.preference.PreferenceManager  
import androidx.test.core.app.ApplicationProvider  
import androidx.test.ext.junit.runners.AndroidJUnit4  
import org.junit.Test  
import org.junit.runner.RunWith  
  
@RunWith(AndroidJUnit4::class)  
class PreferenceDataSourceTest {  
  @Test  
  fun test() {  
    val preferenceDataSource = PreferenceDataSource(  
      PreferenceManager.getDefaultSharedPreferences(  
        ApplicationProvider.getApplicationContext()  
      )  
    )  
  
    preferenceDataSource.someString = "hello world!"  
    assert(preferenceDataSource.someString == "hello world!")  
  }  
}
```

この時に使用しているテストランナーの `AndroidJUnit4` は、Robolectric環境で実行された場合に、テストランナーを `RobolectricTestRunner` に委譲するようです[^3]。Robolectricを使っていても使っていなくても、 `AndroidJUnit4` を使っていれば問題ないということですね。

> Supports running on Robolectric. This implementation will delegate to RobolectricTestRunner if test is running in Robolectric enviroment. A custom runner can be provided by specifying the full class name in a 'android.junit.runner' system property.
> 
> https://developer.android.com/reference/androidx/test/ext/junit/runners/AndroidJUnit4

Robolectricの方では、Robolectric特有のAPI以外については、`androidx.test` のAPIを使うことを推奨しています。

> Robolectric 4.0 includes initial support for androidx.test APIs. We strongly recommend adding androidx.test:core:1.0.0 as a test dependency and using those APIs whenever possible rather than using Robolectric-specific APIs.
> 
> http://robolectric.org/migrating/

特段の理由がなければ、Robolectricよりも `androidx.test` を優先して使っていく方針で良さそうです。

## References

1. [AndroidJUnit4 | Android Developers](https://developer.android.com/reference/androidx/test/ext/junit/runners/AndroidJUnit4) （最終アクセス日：2022年8月3日）
2. [Migration Guide | Robolectric](http://robolectric.org/migrating/) （最終アクセス日：2022年8月3日）

[^1]: http://robolectric.org/
[^2]: https://developer.android.com/jetpack/androidx/releases/test
[^3]: つまり、テストランナーとして `AndroidJUnit4` を指定していても、RobolectricのRuntimeEnvironmentなどが使えるということです。
