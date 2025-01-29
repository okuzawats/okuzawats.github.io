---
title: "[Android] SQLiteのインメモリデータベースで単体テストを書く"
date: 2023-12-22T00:00:00+09:00
description: "Android Advent Calendar 2023の22日目の記事「SQLiteのインメモリデータベースで単体テストを書く」です。"
categories: ["Android", "Test"]
draft: false
---

この記事は[Android Advent Calendar 2023](https://qiita.com/advent-calendar/2023/android)の22日目の記事です。21日目は `@usuiat` による「[意外と知らないModifier.clickable #Android](https://qiita.com/usuiat/items/8b37a0ea56fcc47cb943)」でした。

Androidアプリ開発におけるローカルデータベースは、Android JetpackのRoomを使うケースが一般的になってきたかと思います。Roomを用いる場合は、`inMemoryDatabaseBuilder` を呼び出してインメモリデータベースを作り、単体テストを書くことができました。詳細は、[GitHub ActionsでRoom / Realmの自動テスト](https://okuzawats.com/blog/room-or-realm-test-on-github-actions/)に書いています。

Roomが一般的になってきたとはいえ、まだまだRoomを使わずにSQLiteを使ってローカルデータベースの処理を書く機会はあるかと思います。自分はありました。本記事では、SQLiteを使っている場合の単体テストについて書きます。　　　

## インメモリデータベースの生成

結論から言うと、`SupportSQLiteOpenHelper.Configuration` の `name` に `null` を渡すとインメモリデータベースになります。こうして作ったインメモリデータベースを用いて、Robolectric環境下で単体テストを書くことができます。

「`name` に `null` を渡すとインメモリデータベースになる」ということを調べて知った時は衝撃を受けましたが、事実です。例えば以下のドキュメントにも記載があります。

> name	String: of the database file, or null for an in-memory database
> [SQLiteOpenHelper  | Android Developers](https://developer.android.com/reference/android/database/sqlite/SQLiteOpenHelper)

具体的なコードを以下に示します。`android.content.Context` が必要なので、引数で受け取ります。このContextについては後述します。

```kotlin
import android.content.Context
import androidx.sqlite.db.SupportSQLiteDatabase
import androidx.sqlite.db.SupportSQLiteOpenHelper
import androidx.sqlite.db.framework.FrameworkSQLiteOpenHelperFactory

fun inMemoryDb(context: Context): SupportSQLiteDatabase {
  val configuration: SupportSQLiteOpenHelper.Configuration =
    SupportSQLiteOpenHelper.Configuration
      .builder(context)
      .name(null) // nameにnullを指定するとインメモリデータベースになる
      .callback(callback)
      .build()

  val inMemoryDb: SupportSQLiteDatabase =
    FrameworkSQLiteOpenHelperFactory()
      .create(configuration)
      .writableDatabase

  return inMemoryDb
}
```

`callback` は `SupportSQLiteOpenHelper.Callback` です。本記事では省略しますが、`onCreate` には単体テストに必要なテーブルを作るコードを書きます。

```kotlin
val callback: SupportSQLiteOpenHelper.Callback =
  object : SupportSQLiteOpenHelper.Callback(version = 1) {
    override fun onCreate(
      db: SupportSQLiteDatabase,
    ) {
      // TODO テーブルを作る
    }

    override fun onUpgrade(
      db: SupportSQLiteDatabase,
      oldVersion: Int,
      newVersion: Int,
    ) {
      // TODO
    }
  }
```

## Contextの提供

単体テスト用のContextは、`androidx.test` の `AndroidJUnit4` ランナーを用いて取得します。このランナーについての詳細は、[Robolectricとandroidx.test](https://okuzawats.com/blog/robolectric-vs-androidx-test/)に書いています。

具体的なコードを以下に示します。また、`tearDown` でデータベースをcloseします。

```kotlin
import androidx.sqlite.db.SupportSQLiteDatabase
import androidx.test.core.app.ApplicationProvider
import androidx.test.ext.junit.runners.AndroidJUnit4
import org.junit.After
import org.junit.Test
import org.junit.runner.RunWith

@RunWith(AndroidJUnit4::class)
class InMemoryDbTest {

  private val db: SupportSQLiteDatabase = inMemoryDb(
    context = ApplicationProvider.getApplicationContext(),
  )

  @After
  fun tearDown() {
    db.close()
  }

  // 省略
}
```

## まとめ

本記事では、AndroidアプリのSQLiteの単体テストのためにインメモリデータベースを使う方法について書きました。Android Advent Calender 23日目は、 `@fztkm` で「**何か書きます**」です。楽しみですね。

## References

1. [[Android] GitHub ActionsでRoom / Realmの自動テスト | okuzawatsの日記](https://okuzawats.com/blog/room-or-realm-test-on-github-actions/)
2. [SQLiteOpenHelper  | Android Developers](https://developer.android.com/reference/android/database/sqlite/SQLiteOpenHelper)
3. [[Android] Robolectricとandroidx.test | okuzawatsの日記](https://okuzawats.com/blog/robolectric-vs-androidx-test/)
