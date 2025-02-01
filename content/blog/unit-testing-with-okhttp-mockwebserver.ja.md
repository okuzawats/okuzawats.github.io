---
title: "[Android] OkHttpのMockWebServerを用いた、OkHttp + Retrofitのユニットテスト"
date: 2022-01-10T19:43:00+09:00
description: "OkHttpのMockWebServerを用いて、OkHttp + RetrofitでRESTful APIのクライアントを実装する部分のユニットテストを行うサンプルコードです。"
categories: ["Android", "Test"]
draft: false
---

OkHttpのMockWebServerを用いて、OkHttpとRetrofitを用いたRESTful APIアクセス部分のユニットテストを書いていきます。Android Developersに掲載されているアプリアーキテクチャガイドにおける、Data layerのRemoteDataSourceにあたる部分です。

- [Data layer | Android Developers](https://developer.android.com/jetpack/guide/data-layer)

以下のサンプルコードのOkHttpとMockWebServerのバージョンは4.9.1、Retrofitのバージョンは2.9.0です（バージョンが少し古いのは、このサンプルコードを書いたのがしばらく前だったからです）。MockWebServerは `testImplementation` としています。

```kotlin
dependencies {
  implementation("com.squareup.okhttp3:okhttp:4.9.1")
  implementation("com.squareup.retrofit2:retrofit:2.9.0")
  implementation("org.jetbrains.kotlinx:kotlinx-serialization-json:1.2.0")
  implementation("com.jakewharton.retrofit:retrofit2-kotlinx-serialization-converter:0.8.0")
  testImplementation("com.squareup.okhttp3:mockwebserver:4.9.1")
}
```

### MockWebServerを用いた、OkHttpとRetrofitを用いたRESTful APIアクセス部分のユニットテスト
はじめにテストクラスの最終型を示します。サンプルコードの元になったのが自分の個人プロジェクトなので、趣味に走っているところがあります。それについては後段で説明を加えていきます。ポイントは、 `@Before` と `@After` でMockWebServerの起動と停止をおこなっていること、各テストケースでMockWebServerのレスポンスコードとレスポンスボディを `enqueue` していることになるでしょうか。

```kotlin
class RemoteDataSourceImplTest {

  private val mockWebServer = MockWebServer()

  private lateinit var target: RemoteDataSource

  @ExperimentalSerializationApi
  @Before
  fun setUp() {
    mockWebServer.start()

    target = RemoteDataSourceImpl(
      apiClient = TestApiClientProvider().provideWith(mockWebServer),
    )
  }

  @After
  fun tearDown() {
    mockWebServer.shutdown()
  }

  @Test
  fun testGetAwesomeData_returnRightIfSuccess() = runBlocking {
    mockWebServer.enqueue(MockResponse().setResponseCode(200).setBody(dummyJson()))

    val actual = target.getAwesomeData()

    assert(actual is Either.Right && actual.value.items.isNotEmpty())
  }

  @Test
  fun testGetAwesomeData_returnLeftIfFailure() = runBlocking {
    mockWebServer.enqueue(MockResponse().setResponseCode(500))

    val actual = target.getAwesomeData()

    assert(actual is Either.Left)
  }
}

private fun dummyJson(): String =
"""
{
    "items": [
        {
            "id": 1,
            "name": "Starbucks",
            "created_at": "2022-01-02T05:48:15.209Z",
            "updated_at": "2022-01-02T05:48:15.209Z"
        },
        {
            "id": 2,
            "name": "Komeda Coffee",
            "created_at": "2022-01-02T05:48:46.648Z",
            "updated_at": "2022-01-02T05:48:46.648Z"
        },
        {
            "id": 3,
            "name": "Mr.Donuts",
            "created_at": "2022-01-02T05:49:16.757Z",
            "updated_at": "2022-01-02T05:49:16.757Z"
        }
    ]
}""".trimIndent()
```

コードの説明を以下に示します

### MockWebServerの起動と停止
MockWebServerの起動と停止は以下の箇所で行っています。テストケースごとにMockWebServerの起動と停止を行うため、 `@Before` で `MockWebServer#start` を、 `@After` で `MockWebServer#shutdown` を呼んでいます。

```kotlin
  private val mockWebServer = MockWebServer()

  @ExperimentalSerializationApi
  @Before
  fun setUp() {
    mockWebServer.start()

    // 省略
  }

  @After
  fun tearDown() {
    mockWebServer.shutdown()
  }
```

### MockWebServerへのレスポンスのenqueue
以下の箇所では、MockWebServerにモックレスポンスをenqueueしています。MockWebServerにリクエストを行った時に、queueからdequeueされたモックレスポンスを返してくれます。テストケースに合わせてモックレスポンスを作ってあげることで、RESTful APIアクセス部分のユニットテストを書くことができます。

```kotlin
  @Test
  fun testGetAwesomeData_returnRightIfSuccess() = runBlocking {
    mockWebServer.enqueue(MockResponse().setResponseCode(200).setBody(dummyJson()))

    // 略
  }

  @Test
  fun testGetAwesomeData_returnLeftIfFailure() = runBlocking {
    mockWebServer.enqueue(MockResponse().setResponseCode(500))

    // 略
  }
```

上記の1個目のテストケースでは、リクエストに成功した場合（レスポンスコードが200番台）の時のテストを行っています。レスポンスボディーには、後述するJsonをrawStringで定義した値を渡しています。

上記の2個目のテストケースでは、リクエストに失敗した場合（レスポンスコードが200番台でない）のテストを行っています。レスポンスコードを適宜設定することで、レスポンスコードに応じた挙動のテストを記述できます。また、1個目の例と同様に `setBody` を呼び出すことで、エラーレスポンスを定義することもできます。

### Json
レスポンスのJsonはKotlinのrawStringで定義しました。Jsonのファイルをプロジェクト内に置き、ファイルからJsonの文字列を読み込む方法もあると思います。

```kotlin
private fun dummyJson(): String =
"""
{
    "items": [
        {
            "id": 1,
            "name": "Starbucks",
            "created_at": "2022-01-02T05:48:15.209Z",
            "updated_at": "2022-01-02T05:48:15.209Z"
        },
        {
            "id": 2,
            "name": "Komeda Coffee",
            "created_at": "2022-01-02T05:48:46.648Z",
            "updated_at": "2022-01-02T05:48:46.648Z"
        },
        {
            "id": 3,
            "name": "Mr.Donuts",
            "created_at": "2022-01-02T05:49:16.757Z",
            "updated_at": "2022-01-02T05:49:16.757Z"
        }
    ]
}""".trimIndent()
```

### Retrofit
Retrofitを用いた、RESTful APIのクライアントは以下のように作っています。クライアントのinterfaceはApiClientという名前にしました。APIアクセスを行う関数については、データをRetrofitのResponseに包んで返すsuspend関数として定義しています。Kotlin Coroutinesではなく、例えばRxJavaのSingleを使うような場合でも、全体的な実装の流れは変わらないと思います。

```kotlin
interface ApiClient {
  @GET("/")
  suspend fun getAwesomeData(): Response<AwesomeData>
}
```

### ApiClientの注入
テスト対象にApiClientを注入していますが、そのインスタンスは以下のような処理で生成するようにしています。このサンプルコードではJsonのパーサとしてKotlin Serializationを使っていますが、例えばMoshiを使う時なども同様のインスタンス生成のコードを書けると思います。

```kotlin
@ExperimentalSerializationApi
internal class TestApiClientProvider {
  fun provideWith(mockWebServer: MockWebServer): ApiClient =
    Retrofit.Builder()
      .baseUrl(mockWebServer.url(""))
      .client(OkHttpClient())
      .addConverterFactory(converterFactory())
      .build()
      .create(ApiClient::class.java)

  private val json = Json { ignoreUnknownKeys = true }

  private fun converterFactory(): Converter.Factory =
    json.asConverterFactory(contentType = "application/json".toMediaType())
}
```

### テスト対象のクラス
参考のために、テスト対象のクラスの定義も記載しておきます。Either型を返すsuspend funのみを定義したinterfaceです。Either型については、 [Either / Option in Kotlin](https://okuzawats.com/blog/either-and-option-in-kotlin/) に書きました。簡単に言うと、成功をRight型、失敗をLeft型で表す[sealed class](https://okuzawats.com/blog/kotlin-sealed-class/)です。

```kotlin
interface RemoteDataSource {
  suspend fun getAwesomeData(): Either<AwesomeException, Awesomedata>
}
```

その実装クラスは以下です。前述のApiClientをコンストラクタで注入しています。RetrofitのResponse型をEither型に変換して返しています（Retrofitへの依存をRemoteDataSourceImplの外に持ち出さないようにするためです）。

```kotlin
class RemoteDataSourceImpl @Inject constructor(
  private val apiClient: ApiClient,
) : RemoteDataSource {
  override suspend fun getAwesomeData(): Either<AwesomeException, Awesomedata> =
    apiClient.getAwesomeData().toEither()
}
```

RetrofitのResponse型は、レスポンスコードが200番台の時に `isSuccessful` がtrueを返します。このサンプルコードでは、単純に、 `isSuccessful` がtrueの時はEither.Right、falseの時はEither.Leftを返すようにしています。

```kotlin
private fun <T : Any> Response<T>.toEither(): Either<AwesomeException, T> =
  if (isSuccessful) {
    Either.Right(requireNotNull(body())
  } else {
    Either.Left(AwesomeException())
  }
```

## Reference
1. [Data layer | Android Developers](https://developer.android.com/jetpack/guide/data-layer)（最終アクセス日：2022年1月7日）
2. [Either / Option in Kotlin](https://okuzawats.com/blog/either-and-option-in-kotlin/)（最終アクセス日：2022年1月10日）
3. [[Kotlin] sealed classに親しむ](https://okuzawats.com/blog/kotlin-sealed-class/)（最終アクセス日：2022年1月20日）
