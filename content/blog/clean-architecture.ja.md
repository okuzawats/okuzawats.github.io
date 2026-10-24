---
title: "[Android] Clean Architectureの理論と実装"
date: 2022-11-06T22:38:00+09:00
description: "Androidアプリ開発でのClean Architectureの理論と実装に関するアイデアを示します。"
categories: ["Android", "設計"]
draft: false
---

追記：以下の書籍に、本記事の内容に加筆・修正した内容が収録されています。是非、お手にとっていただけますと幸いです。

- [［入門］ドメイン駆動設計 基礎と実践・クリーンアーキテクチャ](https://gihyo.jp/book/2024/978-4-297-14317-6)

---

Androidアプリにおいてクリーンアーキテクチャを採用する場合の基本的な実装アイデアをこの記事にまとめます。あくまで自分の個人的なアイデアですので、他にもいろいろな異なる考え方が存在する可能性があります。その辺を割り引いて、ひとつのアイデアとして受け取っていただけますと幸いです。予防線を張ったところで始めていきます。

サンプルコードはGitHubの以下のリポジトリに置いてあります。ので、プロジェクト全体をローカル環境で参照したい場合はリポジトリをForkしてください。また、本記事執筆時点でのコードにはタグが切ってありますので、記事に対するソースコードを参照したい場合は、タグのコミットをチェックアウトしてください。

- [okuzawats/android-clean-architecture: Android Clean Architecture Sample App (WIP🤪)](https://github.com/okuzawats/android-clean-architecture)
- [Release verion.1 · okuzawats/android-clean-architecture](https://github.com/okuzawats/android-clean-architecture/releases/tag/v1)

本記事では、Androidアプリ開発に関する基本的な知識と、依存性注入（DI）、依存関係逆転の原則を理解していることを前提条件とします。DI及び依存関係逆転の原則については本ブログ内にも記事がありますので、必要に応じて参考にしていただければと思います。

- [何故、依存性注入（DI）するのか](https://okuzawats.com/blog/dependency-injection/)
- [依存関係逆転の原則](https://okuzawats.com/blog/dip/)

## クリーンアーキテクチャの図

まずは、書籍クリーンアーキテクチャ (Robert (2018), p.200) の有名な図と、Androidのアプリアーキテクチャガイド (https://developer.android.com/topic/architecture) を参考に、目指すアプリのアーキテクチャの図を描きます。それが以下の図です。

{{< figure src="/images/clean-architecture.webp" class="center" >}}

### Domain

この図の中心には、Domainが存在します。本記事におけるDomainでは、ビジネスルールをUseCaseとして定義します。書籍クリーンアーキテクチャでは、UseCaseの中心にさらにEntityが存在しますが、本記事ではEntityは登場しません。

UseCaseの実装においての注意点としては、ひとつのUseCaseは単一の機能のみを持つ、ということがあります。すなわち、ひとつのUseCaseに対して、ひとつのメソッドのみが定義されます。この方針での実装はいろいろなやり方が考えられると思いますが、本記事では、以下の記事で紹介した、`operator fun invoke` を実装する方針で実装します。

- [[Android] UseCaseの実装](https://okuzawats.com/blog/implement-usecase-in-android/)

この方針で実装した場合、UseCaseのインスタンスに対して `awesomeUseCase()` というようにinvokeを呼び出し、UseCaseの処理を実行することができます。

### Presentation

図のDomainの上に目を向けると、Presentationが存在します。PresentationはUIに対するInterface Adapterの役割を持ち、ViewModelとして実装しています。PresentationはAndroidフレームワークへの依存を持たないように実装すべきですが、実装都合で、Android Architecture ComponentsのViewModelを用います。

Android Architecture ComponentsのViewModelはAndroidの複雑なライフサイクルをいい感じに扱うために非常に便利ですが、すなわちフレームワーク非依存と言いにくくなってしまいます。まあ便利だから使うんですが...。

以下、本記事で特に断りなくViewModelと書く時は、Android Architecture ComponentsのViewModelではなく、Presentationにおける（MVVM的な意味での）ViewModelを指します。

### UI

図のPresentationのさらに上に目を向けると、UIが存在します。AndroidにおけるUIは、Activity、Fragment、Composeといったものです。UIは、Androidのフレームワークに依存します。

### Data

今度は、図のDomainの下に目を向けます。ここには、Dataが存在します。Dataは主にRepositoryとして実装します。Repositoryは、RESTful API、ローカルデータベース、SharedPreferenceなどのデータソースに対するInterface Adapterの役割を持ち、データアクセスを抽象化します。

### Data Source

さらにDataの下に目を向けると、DataSourceが存在します。DataSourceには、RESTful API、ローカルデータベース、SharedPreference等、データに直接アクセスするコードが書かれます。

## 制御の方向と依存の方向

図中の青い色の矢印と赤い色の矢印に注目します。これは制御の方向と依存の方向を表しています。

例えば、UIとPresentationについて考えます。具体的な状況としては、MVVMにおけるViewとしてのActivityとViewModelです。

{{< figure src="/images/view-viewmodel.webp" class="center" >}}

典型的なMVVMの実装では、Activityがユーザーアクションを受け、ViewModelにアクションを伝えます。ViewModelではアクションを受け取り、何らかの処理を行ったのち、ViewであるActivityに状態更新通知などを送ることになるでしょう。この更新通知は主としてObserverパターンを用いて実装されるため、ViewModelはそのサブスクライバであるViewへの依存を持たない、ということになります。すなわち、依存の方向は、UI (View) → Presentation (ViewModel) となります。ViewがViewModel側に定義されたメソッドを呼び出しますので、制御の流れも同じくUI (View) → Presentation (ViewModel)です。

一方、DomainとDataでは、制御の方向と依存の方向が逆を向いています。つまり、DomainにおけるUseCaseはDataにおけるRepositoryのメソッドを呼び出しますが、RepositoryがUseCaseに依存する、という依存関係の方向になっているということです。どういうことでしょうか？

これはつまり、UseCaseとRepositoryに依存関係逆転の原則を適用し、RepositoryからUseCaseに依存の方向が向くようにしている、ということです（「[依存関係逆転の原則](https://okuzawats.com/blog/dip/)」を参照）。

具体的には、Domain側にRepositoryのinterfaceを定義し、Data側ではそのinterfaceを実装します。こうすることで、依存の向きはDataからDomainに向き、制御の方向と依存の方向を逆にすることができます。

{{< figure src="/images/domain-data.webp" class="center" >}}

DataとDataSouceについても、DomainとDataと同様に依存性を逆転させて、DataSourceからDataへ依存の方向が向くようにします。

## 実装

ここからは具体的なコードを見ていきます。サンプルアプリとして、犬の画像をランダムで表示するアプリを考えます。犬の画像をランダムで表示するアプリのドメインとは？と疑問に思うかもしれませんが、あくまで実装のサンプルですのでご容赦ください。

犬の画像を取得するため、Dog APIをお借りします。

- [Dog API](https://dog.ceo/dog-api/)

### Domainの実装

最初にDomainの実装をしていきます。Domainは、（冒頭の図における）円の中心に位置し、他のどこにも依存しませんでした。ということは、Domainは独立して実装することが可能ということを意味します。

まずはDomainのモジュールを作成します。Android Studioから、Android Libraryとして `:domain` を作成します。また、アプリケーションのモジュールは `:domain` モジュールに依存する必要があるため、app/build.gradle.ktsに `:domain` モジュールを追加します。以降、特に断らずに、app/build.gradle.ktsに全てのモジュールへの依存を追加していきます。

```kotlin
dependencies {
  implementation(project(":domain"))
  // 略
}
```

さて、UseCaseのinterfaceを定義します。前述のとおり、UseCaseでは `operater fun invoke` を実装します。UseCase#invokeがどのような型を返すべきかは一考の余地がありますが...ここではEither型のFlowを返すことにしました。

```kotlin
package com.okuzawats.cleanarchitecture.domain.getrandomdogimage

[imports]

/**
 * A use case of getting a random dog image
 */
interface GetRandomDogImageUseCase {
  suspend operator fun invoke(): Flow<Either<DogImageFetchingException, DogImage>>
}
```

前述のとおり、DomainではRepositoryのinterfaceを定義します。ここでは、Either型を返す `suspend fun` を定義します。

```kotlin
package com.okuzawats.cleanarchitecture.domain.getrandomdogimage.repository

[imports]

/**
 * Repository for dog images
 */
interface DogImageRepository {
  suspend fun getRandom(): Either<DogImageFetchingException, DogImage>
}
```

UseCaseを実装します。UseCaseの実装クラスでは、Dagger HiltでのDIを前提として（DIについては後述します）、RepositoryをDIします。ここでは動作確認のために `delay` を入れていますが、あまり気にしないでください :)

```kotlin
package com.okuzawats.cleanarchitecture.domain.getrandomdogimage.impl

[imports]

/**
 * Implementation of [GetRandomDogImageUseCase]
 */
class GetRandomDogImageUseCaseImpl @Inject constructor(
  private val dogImageRepository: DogImageRepository,
) : GetRandomDogImageUseCase {
  override suspend fun invoke(): Flow<Either<DogImageFetchingException, DogImage>> =
    flow {
      // TODO remove this delay
      delay(1500L)
      emit(dogImageRepository.getRandom())
    }
}
```

### Dataの実装

dataモジュールを作成します。dataモジュールはdomainモジュールに依存するので、`:data` のbuild.gradle.ktsに `:domain` への依存を追加します。

```kotlin
dependencies {
  implementation(project(":domain"))
  // 略
}
```

domainモジュールに定義したRepositoryのinterfaceを実装します。このクラスでは、RemoteDataSource、及びDataToDomainMapperをDI（コンストラクタインジェクション）しています。RemoteDataSourceはinterfaceで、dataモジュール内で定義します（後述）。RemoteDataSourceをdataモジュールで定義することで、依存の向きをdataSourceモジュールからdataモジュールに向けています。DataToDomainMapperは、dataモジュールで定義するModelをdomainモジュールで定義するModelに変換します（説明は省略）。

```kotlin
package com.okuzawats.cleanarchitecture.data.repository.dogimage

[imports]

/**
 * Implementation of [DogImageRepository]
 */
class DogImageRepositoryImpl @Inject constructor(
  private val remoteDataSource: RemoteDataSource,
  private val dataToDomainMapper: DataToDomainMapper,
) : DogImageRepository {
  override suspend fun getRandom(): Either<DogImageFetchingException, DogImage> =
    dataToDomainMapper.toDomain(
      dogImage = remoteDataSource.getRandomDogImage(),
    )
}
```

上述のRemoteDataSource interfaceを定義します。

```kotlin
package com.okuzawats.cleanarchitecture.data.datasource

import arrow.core.Either

/**
 * Data source over network
 */
interface RemoteDataSource {
  /**
   * Get random dog image
   */
  suspend fun getRandomDogImage(): Either<Throwable, String>
}
```

### DataSourceの実装

DataSourceについては、dataモジュールの中に `:remote` モジュールを追加します。今回はRESTful APIであるDog APIをデータソースとして用いるため、モジュールの名前をremoteとしています。ローカルデータベースやSharedPreferenceに対するデータソースは、例えば `:local` などの名前になるでしょう。

remoteモジュールは、dataモジュールに依存します。

```kotlin
dependencies {
  implementation(project(":data"))
  // 略
}
```

dataモジュールで定義したRemoteDataSource interfaceを実装します。コンストラクタ引数として受け取っているApiClientは、Retrofitを用いてRESTful APIアクセスを行うためのinterfaceです。RemoteDataSourceImplでは、RetrofitのResponseクラスの型としてRESTful APIからデータを受け取ります。こうしたAPI用のModelへの依存を外に漏らさないように、依存先のdataモジュールに定義されているModelの型に変換します。

```kotlin
package com.okuzawats.cleanarchitecture.data.remote

[imports]

class RemoteDataSourceImpl @Inject constructor(
  private val apiClient: ApiClient,
  private val dataSourceToDataMapper: DataSourceToDataMapper,
) : RemoteDataSource {
  // TODO: define error types
  override suspend fun getRandomDogImage(): Either<Throwable, String> =
    try {
      dataSourceToDataMapper.toData(
        apiClient.getRandomImage()
      )
    } catch (e: IOException) {
      Throwable("response not received").left()
    }
}
```

### Presentationの実装

次に、Presentationを実装します。`:presentation` モジュールを作成し、domainモジュールへの依存を追加します。

```kotlin
dependencies {
  implementation(project(":domain"))
  // 略
}
```

そして謝らなくてはならないことがあります。ごめんなさい。「クリーンアーキテクチャの図」の節で、「実装都合で、Android Architecture ComponentsのViewModelを用います」と書いていましたが、サンプルコードではUniFlowというライブラリの提供するAndroidDataFlowを各ViewModelの親クラスとして用いていました。まあ、AndroidDataFlowはAndroid Architecture ComponentsのViewModelを継承しているので、嘘はついていないかもしれません。

- [uniflow-kt/uniflow-kt: Uniflow 🦄 - Simple Unidirectional Data Flow for Android & Kotlin, using Kotlin coroutines and open to functional programming](https://github.com/uniflow-kt/uniflow-kt)

とにかく、ViewModelは以下のように実装しました。

```kotlin
package com.okuzawats.cleanarchitecture.presentation.main

[imports]

@HiltViewModel
class MainViewModel @Inject constructor(
  private val getRandomDogImageUseCase: GetRandomDogImageUseCase,
  private val domainToPresentationMapper: DomainToPresentationMapper,
) : AndroidDataFlow(defaultState = MainViewModelState.Initial) {
  fun onEntered() {
    action {
      setState(MainViewModelState.Loading)
      getRandomDogImageUseCase()
        .map(domainToPresentationMapper::toPresentation)
        .map(MainViewModelState::from)
        .onEach(::setState)
        .launchIn(viewModelScope)
    }
  }
}
```

UniFlowの機能を使っているためわかりにくいと思いますが、

- `onEntered` が呼び出された時、
  1. ViewModelStateをLoadingに設定する
  2. UseCaseをinvokeし、データを受け取ったらそのデータに応じたViewModelStateを設定する

というだけのコードです。

ポイントとしては、ViewModelがUseCaseをコンストラクタ引数として受け取り（コンストラクタインジェクション）、アクションを受け取った時にUseCaseの処理を実行している点です。これは、依存と制御の方向がPresentationからDomainに向いているということに他なりません。

### UIの実装

最後に、`:ui` モジュールを作成し、`:presentation` モジュールへの依存を追加します。

```kotlin
dependencies {
  implementation(project(":presentation"))
  // 略
}
```

View (Activity) の実装は以下のように行いました。ViewModelの公開するViewModelStateを購読した上でViewModelStateをUiStateに変換し、変更通知を受け取った時に画面の更新を行っています（以下のコードだけを読んでも良くわからないと思いますので、ソースコードを読んでいただければと思います）。

```kotlin
package com.okuzawats.cleanarchitecture.ui.main

[imports]

@AndroidEntryPoint
class MainActivity : ComponentActivity() {

  @Inject
  internal lateinit var presentationToUiMapper: PresentationToUiMapper

  @Inject
  internal lateinit var uiStateRenderer: UiStateRenderer

  private val viewModel: MainViewModel by viewModels()

  override fun onCreate(savedInstanceState: Bundle?) {
    super.onCreate(savedInstanceState)
    setContent {
      CleanArchitectureTheme {
        Surface(
          modifier = Modifier.fillMaxSize(),
          color = MaterialTheme.colors.background
        ) {
          viewModel.states
            .map(presentationToUiMapper::toUi)
            .observeAsState()
            .let { uiStateRenderer.RenderAsComposable(it) }
        }
      }
    }
  }

  override fun onResume() {
    super.onResume()
    viewModel.onEntered()
  }
}
```

### Darty Main

冒頭の図だけ見れば、以上で実装が終わりです。しかしながら、実際にはもうひとつだけ必要なモジュールが存在します。Robert C. Martinの言うところの、「Dirty Main」です。Dirty Mainの役割を担うモジュールは、Androidのプロジェクト新規作成時に自動的に作成される `:app` モジュールがピッタリでしょう。

このappモジュールの役割は、これまで登場した個々のモジュールを統合し、ひとつのアプリケーションとして成立させることです。サンプルプロジェクトでは、DIのためのコードをこのモジュール内に置いています。サンプルプロジェクトには存在しませんが、もし画面遷移を実装する場合はこのモジュールが適当であると思います。

既に述べたように、appモジュールは自身以外の全てのモジュールへの依存を持っている必要があります。個々のモジュールを統合してDIの依存解決を行う、等の役割を持つためです。

```kotlin
dependencies {
  implementation(project(":ui"))
  implementation(project(":presentation"))
  implementation(project(":domain"))
  implementation(project(":data"))
  implementation(project(":data:remote"))
  // 略
}
```

appモジュールでは、ApplicationクラスやDIのためのコードが存在します。

```kotlin
package com.okuzawats.cleanarchitecture

[imports]

@HiltAndroidApp
class App : Application()
```

```kotlin
package com.okuzawats.cleanarchitecture.di

[imports]

@Module
@InstallIn(ViewModelComponent::class)
class UseCaseModule {
  @Provides
  fun provideGetRandomDogImageUseCase(
    impl: GetRandomDogImageUseCaseImpl,
  ): GetRandomDogImageUseCase = impl
}
```

ここまでで、クリーンアーキテクチャを用いたサンプルプロジェクトが完成しました。

## 参考文献

1. Robert C. Martin, 角征典(訳), 高木正弘(訳), (2018), Clean Architecture 達人に学ぶソフトウェアの構造と設計, アスキードワンゴ
2. Eran Boudjnah, (2022), Clean Architecture for Android, BPB Online
3. Alexandru Dumbravan, (2022), Clean Android Architecture, Packt Publishing Ltd.
4. Android Developers, Guide to app architecture, retrieved form https://developer.android.com/topic/architecture (Last Access: 2022/11/5)
