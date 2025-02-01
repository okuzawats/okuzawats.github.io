---
title: "[Android] GitHub ActionsでRoom / Realmの自動テスト"
date: 2021-12-25T00:00:00+09:00
description: "フラー株式会社 Advent Calendar 2021の25日目の記事「GitHub ActionsでRoom / Realmの自動テスト」です。"
categories: ["Android", "GitHub Actions", "Test"]
aliases:
    - /posts/room-or-realm-test-on-github-actions/
draft: false
---

この記事はフラー株式会社 Advent Calendar 2021の25日目の記事です。

- [フラー株式会社 Advent Calendar 2021のカレンダー \| Advent Calendar 2021 \- Qiita](https://qiita.com/advent-calendar/2021/fuller-inc)

24日目の記事は `@su8` さんによる「[コンテナ遠洋航海 ~そこらへんの草でも食わせておけ プロセス分離の旅路 ~ - Qiita](https://qiita.com/su8/items/517b5a2de22973dc4513)」でした。

## Androidのローカルデータベースアクセスの自動テスト

2021年現在、Androidアプリ開発でローカルデータベース（SQLite）にデータを保存する場合は、Roomを使用することが多いと思います。数年前はRealmが流行っていたので、今もRealmを使用しているというプロジェクトも多いと思います。この記事では、RoomとRealmを用いたローカルデータベースアクセスの自動テストをGitHub Actionsで行う方法について書きます。

### Roomのテスト

Roomのテストは、Dao単位で行うことを想定します。Roomのインメモリデータベースを用いてテストを行います。エミュレータテストになるので、 `androidTest` にテストを書きます。

本記事の例では、Contextの取得に `androidx.test.core.app.ApplicationProvider` を、テストランナーとして `androidx.test.ext.junit.runners.AndroidJUnit4` を使用します。

- [androidx\.test\.core\.app \| Android Developers](https://developer.android.com/reference/androidx/test/core/app/package-summary)
- [AndroidJUnit4 \| Android Developers](https://developer.android.com/reference/androidx/test/ext/junit/runners/AndroidJUnit4)

`setup` と `tearDown` までは以下のように書けます。ここで、AwesomeDaoはRoomのDao、DatabaseはRoomDatabaseのサブクラスです。 `setup` でインメモリデータベースをビルドした後、テストのtargetとなるDaoを作ります。 `tearDown` では、データベースをcloseしています。

```kotlin
@RunWith(AndroidJUnit4::class)
class AwesomeDaoTest {

  private lateinit var target: AwesomeDao
  private lateinit var database: Database

  @Before
  fun setup() {
    val context = ApplicationProvider.getApplicationContext<Context>()
    database = Room
      .inMemoryDatabaseBuilder(context, Database::class.java)
      .setTransactionExecutor(StandardTestDispatcher().asExecutor())
      .setQueryExecutor(StandardTestDispatcher().asExecutor())
      .build()
    target = database.awesomeDao()
  }

  @After
  fun tearDown() {
    database.close()
  }
}
```

ここまでの準備ができたらあとはテストケースを書けば良いのですが、特にどうということもないので省略します。テストは `gradlew connectedAndroidTest` などのコマンドから実行することができます。実際には、Build Variantsを適切に設定してください。

### Realmのテスト

Roomと同様に、RealmのテストもRealmの提供するインメモリデータベースを用いてテストを行います。Realmのインメモリデータベースのインスタンスは、以下のように作成することができます。

```kotlin
RealmConfiguration.Builder()
  .inMemory()
  .name("your_name_here")
  .build()
```

データベースアクセスを行うクラスにこのRealmのインスタンスをDIできるようにしておけば、Realmを用いたデータベースアクセスのテストを書くことができます。やっていることはRoomのテストと同様です。

```kotlin
class AwesomeDataSourceTest {

  private lateinit var target: AwesomeDataSource
  private var realm: Realm? = null

  @Before
  fun setup() {
    val config = RealmConfiguration.Builder()
                  .inMemory()
                  .name("your_name_here")
                  .build()
    realm = Realm.getInstance(config)
    target = AwesomeDataSource(requireNotNull(realm))
  }

  @After
  fun tearDown() {
    realm?.close()
    realm = null
  }
}
```

### GitHub Actions

ここまでできれば、あとはGitHub Actionsでトリガーが引かれた時にテストを実行するだけです。Instrumented Testにする必要がありますが、GitHub Actions上でAndroidのエミュレータをセットアップするための素晴らしいアクション `malinskiy/action-android` が存在しますので、こちらを使えば苦労はいりません。

- [Malinskiy/action-android: Collection of Android-related GitHub Actions](https://github.com/Malinskiy/action-android)

なお、エミュレータテストのために、OSとしてMacを選択する必要があることに注意してください（Ubuntuを使用する場合に比べて10倍の速度でお金が溶けます）。

※ 2024年3月29日追記：2024年1月22日にUbuntu対応が入ったので、macOSのインスタンスを使わなくても動くようになりました🎉 => [Release 0.1.6 · Malinskiy/action-android](https://github.com/Malinskiy/action-android/releases/tag/0.1.6)

以下は簡単なGitHub Actionsのワークフローのサンプルです。参考のため、Build Variantsを `DevDebug` に設定しています。

```yaml
jobs:
  test:
    runs-on: macos-latest
    steps:
      - uses: actions/checkout@v2
      - name: Set Up JDK 11
        uses: actions/setup-java@v2
        with:
          distribution: 'zulu'
          java-version: '11'
      - uses: actions/cache@v2
        with:
          path: |
            ~/.gradle/caches
            ~/.gradle/wrapper
          key: ${{ runner.os }}-gradle-${{ hashFiles('**/*.gradle*', '**/gradle-wrapper.properties') }}
          restore-keys: |
            ${{ runner.os }}-gradle-
      - name: Install Android SDK
        uses: malinskiy/action-android/install-sdk@release/0.1.2
      - name: Instrumented Test
        uses: malinskiy/action-android/emulator-run-cmd@release/0.1.2
        with:
          cmd: ./gradlew connectedDevDebugAndroidTest
          api: 28
          tag: default
          abi: x86
```

以上で、Room / Realmの自動テストをGitHub Actions上で実行することができました。

## まとめ

2019年、2020年に続き、2021年もフラー Advent Calendarの最終日を務めさせていただきました。来年も最終日を狙って参ります。よろしくお願いいたします。

- [フラー Advent Calendar 2019 - Adventar](https://adventar.org/calendars/4155)
- [フラー Advent Calendar 2020 - Adventar](https://adventar.org/calendars/5034)
