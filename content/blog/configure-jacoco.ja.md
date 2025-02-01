---
title: "[Android] build.gradle.ktsでJaCoCoを動かす"
date: 2023-01-22T05:58:00+09:00
description: "AndroidでGradleでbuild.gradle.ktsを使っている場合にJaCoCoをセットアップする方法です。"
categories: ["Android"]
draft: false
---

※ 本記事で紹介している[arturdm/jacoco-android-gradle-plugin](https://github.com/arturdm/jacoco-android-gradle-plugin)を用いる方法は、Gradle 8では動かないようです。[Fixed build crash in gradle 8.0](https://github.com/arturdm/jacoco-android-gradle-plugin/pull/103)に修正のPRが作成されていますが、本日現在、まだマージはされておりません。代わりに、[thsaravana/jacoco-android-playground](https://github.com/thsaravana/jacoco-android-playground)を参考にしてGradleのタスクを作成する方法があります。

AndroidのGradleの設定ファイルを `build.gradle` から `build.gradle.kts` に書き換えた時、JaCoCoをセットアップする方法がわからなかったので頑張って動かしてみました。GradleとJaCoCoについてはよくわかっていないことが多いです。プロジェクトを以下のリポジトリにpushしてあります。

- [okuzawats/android-jacoco-test-report: measure test coverage of android app with JaCoCo](https://github.com/okuzawats/android-jacoco-test-report)

JaCoCoのセットアップのために、Android用のプラグインを作ってくれている方がいたので、利用させていただきました。とても助かりました。ありがとうございます。

- [arturdm/jacoco-android-gradle-plugin: Gradle plugin that creates JaCoCo test reports for Android unit tests](https://github.com/arturdm/jacoco-android-gradle-plugin)

## プロジェクトのsettings.gradle.kts

Maven Repositoryを追加します。プロジェクトによっては、プロジェクトの `build.gradle.kts` に追加する場合もあると思います。

```kotlin
dependencyResolutionManagement {
  // 略
  repositories {
    // 略
    maven(url = "https://plugins.gradle.org/m2/")
  }
}
```

## プロジェクトのbuild.gradle.kts

プラグインをdependenciesに追加します。

```kotlin
buildscript {
  dependencies {
    // 追加
    classpath("com.dicedmelon.gradle:jacoco-android:0.1.5")
  }
}

plugins {
  // 略
}
```

## モジュールのbuild.gradle.kts

プラグインを適用し、JaCoCoのバージョン、出力するカバレッジレポートの種別、カバレッジレポートの有効化を行います。

```kotlin
plugins {
  // 略
  id("com.dicedmelon.gradle.jacoco-android")
}

jacoco {
    toolVersion = "0.8.8"
}

tasks.withType<JacocoReport> {
    reports {
        csv.required.set(false)
        html.required.set(true)
        xml.required.set(true)
    }
}

android {
  // 略
  buildTypes {
    debug {
      enableUnitTestCoverage = true
    }
    // 略
  }
  // 略
}

dependencies {
  // 略
}
```

## カバレッジレポートの出力

ユニットテスト実行後、コマンドを実行します。`jacoco～` のコマンドは、プラグインが作ってくれるタスクです。

```shell
$ ./gradlew testDebugUnitTest
$ ./gradlew jacocoDebugUnitTest
```

`app` モジュールの場合は、`app/build/jacoco/` にカバレッジレポートが出力されます。

## カバレッジレポートのマージ

モジュールごとに出力されたカバレッジレポートをマージする方法がわからなくて困っていたのですが、[@t179a](https://twitter.com/t179a)さんが教えてくれました。ありがとうございました。

- [マルチモジュールアプリでJacocoのReportをまとめる方法 - Speaker Deck](https://speakerdeck.com/t179a/marutimoziyuruapuridejacoconoreportwomatomerufang-fa)

kotlinx-koverを使うと実現できました。

- [Kotlin/kotlinx-kover](https://github.com/Kotlin/kotlinx-kover)

まずはプロジェクトのbuild.gradleにkoverを追加して、有効化します。

```kotlin
plugins {
    id("org.jetbrains.kotlinx.kover") version "0.6.1" // 追加
}

koverMerged {
    enable() // 追加
}
```

次に、各モジュールのbuild.gradleにkoverを追加します。

```kotlin
plugins {
    id("org.jetbrains.kotlinx.kover") // 追加
}
```

以下のコマンドで `build/reports/kover/merged/` にマージされたテストレポートが出力されました。

```shell
% ./gradlew koverMergedReport
```

## References

- [arturdm/jacoco-android-gradle-plugin: Gradle plugin that creates JaCoCo test reports for Android unit tests](https://github.com/arturdm/jacoco-android-gradle-plugin)
- [JaCoCoカバレッジレポートをPRにコメントしてくれる機能が欲しい！（Codecov使えない人向け） - Qiita](https://qiita.com/tarumzu/items/d1e8fa611d966412c097)
- [マルチモジュールアプリでJacocoのReportをまとめる方法 - Speaker Deck](https://speakerdeck.com/t179a/marutimoziyuruapuridejacoconoreportwomatomerufang-fa)
- [Kotlin/kotlinx-kover](https://github.com/Kotlin/kotlinx-kover)
