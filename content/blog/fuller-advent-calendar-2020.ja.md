---
title: "Androidアプリ開発入門2020"
date: 2020-12-25T00:00:00+09:00
description: "フラー Advent Calendar 2020、25日目の記事「Androidアプリ開発入門2020」です。"
categories: ["Android"]
aliases:
    - /posts/fuller-advent-calendar-2020/
draft: false
---

この記事は[フラー Advent Calendar 2020 \- Adventar](https://adventar.org/calendars/5034)の25日目の記事です。24日目の記事は、shmokmtによる「multipart をGoで扱ってみる」でした。

さて、元気にAndroidアプリ開発してますか。僕はしてます。

突然ですが、Androidアプリ開発をする上でつらいことの一つは、Androidアプリ開発の初心者にAndroidアプリ開発のことを教えることだと思ってます。まずはこの図を見てください。

{{< figure src="/images/android-in-2020-1.webp" title="Fig-1 Android in 2020" class="center" >}}

出典：[来年に備えるために Android の知識を網羅する / Looking back on this Android year in preparation for next year\. \- Speaker Deck](https://speakerdeck.com/wasabeef/looking-back-on-this-android-year-in-preparation-for-next-year)

こちらのスライドは、GDG DevFest Tokyo 2019にてwasabeefさんが発表されていたスライドです。Androidアプリ開発でよく使われるライブラリやらツールやらを示しています。多いですね。多いだけならいいですが、RxJavaやDagger2など、単体で多くの学習コストを払わないといけない人がカジュアルに混ざっていて困ります。

こちらのスライドは前述のとおり2019年のものですので、日進月歩のAndroidアプリ開発の世界ではちょっと内容が古いと言えます。このような混乱した状況がそう長く続くわけはありません。2020年の今はきっともっと楽になっているはずです。そうに違いない。

{{< figure src="/images/android-in-2020-2.webp" title="Fig-2 Android Developers right now" class="center" >}}

出典：[\(20\) KeithYokomaさんはTwitterを使っています 「Android Developers right now 😂 https://t\.co/vhQgjPJYqo」 / Twitter](https://twitter.com/KeithYokoma/status/1276404081070768130)

こちらの画像（ツイート）が投稿されたのが2020年6月26日です。見て分かるとおり...新たなラプトルが襲いかかってきており、状況は悪化しました。その他にも、Kotlin Android ExtensionsがDeprecatedになったり、Androidアプリエンジニアが退屈することはなさそうです。

- [Say Good\-Bye to Kotlin Android Extensions \- Speaker Deck](https://speakerdeck.com/okuzawats/say-good-bye-to-kotlin-android-extensions)

## Androidアプリ開発入門2020

学ばなくてはならない概念が多く、今からAndroidアプリ開発に入門するのはしんどそうです。が、これからAndroidアプリ開発に入門しなければならない人もいるでしょう。そういった方のために、これから学ぶならこの辺から...という指針を自分なりに書いておきます。キーワードを散りばめておくので、検索しながら頑張ってください。大丈夫、Androidアプリ開発は楽しいから...。

### Kotlin

Androidアプリ開発で使うプログラミング言語といえば、Kotlinですね。Java（Android Java）も使えますが、Kotlinを使った方が生産性が高いでしょう。公式でサポートされていることもあり、Kotlinを選ばない理由はあまり見当たりません。

この辺をやったり（自分はやったことないけど）、適当な入門書を読めばとりあえず書くことはできるでしょう。

- [Kotlin Bootcamp for Programmers  \|  Training Courses](https://developer.android.com/courses/kotlin-bootcamp/overview)

もっとKotlinのことを詳しく学びたい人は、英語の電子書籍しかないと思いますが、Effective Kotlinを読むのがいいと思います。

- [Effective Kotlin by Marcin Moskała \[Leanpub PDF/iPad/Kindle\]](https://leanpub.com/effectivekotlin/)

CoroutinesとかFlowとかもついでにどこかで...。

### LiveData

Lifecycle Awareなデータホルダーで、変更通知ができるやつです。後述するViewModelと一緒に使うと最も力を発揮します。MVVMでViewModelからViewに変更通知を流したりするのに使うことが多いでしょうか。Lifecycle Awareなので、正しく使えば比較的簡単にクラッシュしないアプリを作ることができるでしょう...。ちょっと癖があるといえばあるので、使いこなしましょう。

- [LiveData Overview  \|  Android Developers](https://developer.android.com/topic/libraries/architecture/livedata)

### ViewModel

前述のLiveDataと一緒に使うといいやつです。Androidは、画面が回転する時など、割とカジュアルに画面が一度破棄されて再生成されるんですが、ViewModelを使うとその辺をいい感じにやってくれます。その名の通り、MVVMにおけるViewModelとして使うことが多いでしょうか。

- [ViewModel Overview  \|  Android Developers](https://developer.android.com/topic/libraries/architecture/viewmodel)

### DataBinding

Android公式のDataBinding機構です。レイアウトに式を書けるようになるので、楽になったり（ならなかったり）します。また、自分でBindingAdapterを書いたりすると楽になったり（ならなかったり）します。ActivityとかFragmentとかで`binding.textView`みたいに書くとViewの取得をしてくれるのは偉いです。

- [Data Binding Library  \|  Android Developers](https://developer.android.com/topic/libraries/data-binding)

Viewを取得したいだけなら、ViewBindingが使えます。

- [View Binding  \|  Android Developers](https://developer.android.com/topic/libraries/view-binding)

Kotlinを使っているなら、wada811氏の作ったライブラリを一緒に使うと良いでしょう。

- [wada811/DataBinding\-ktx: DataBinding\-ktx make easy to use DataBinding\.](https://github.com/wada811/DataBinding-ktx)
- [wada811/ViewBinding\-ktx: ViewBinding\-ktx make easy to use ViewBinding\.](https://github.com/wada811/ViewBinding-ktx)

### MVVM

 アプリのアーキテクチャとしては、こんな感じのMVVMっぽくしておくと無難な気がします。先述のLiveDataとViewModelも、MVVMで使うことで力を発揮してくれるものと思います。

{{< figure src="/images/android-in-2020-3.webp" title="Fig-3 Android Architecture" class="center" >}}

- [Guide to app architecture  \|  Android Developers](https://developer.android.com/jetpack/guide)

### Retrofit

上の図にあるRetrofitですが、HTTPクライアントです。OkHttpや、JsonパーサのMoshi（最近は、kotlinx.serializationという選択肢もありますが...）と一緒に使うことが多いでしょう。

- [square/retrofit: A type\-safe HTTP client for Android and the JVM](https://github.com/square/retrofit)
- [square/okhttp: Square’s meticulous HTTP client for the JVM, Android, and GraalVM\.](https://github.com/square/okhttp)
- [square/moshi: A modern JSON library for Kotlin and Java\.](https://github.com/square/moshi)

### Room

こちらも上の図にあるRoomですが、ローカルDBです。SQLiteのラッパーです。前述のLiveDataを一緒に使うと変更通知の実装がとても楽にできます。

- [Save data in a local database using Room  \|  Android Developers](https://developer.android.com/training/data-storage/room)

このCodelabsが勉強になります。

- [Android fundamentals 10\.1 Part A: Room, LiveData, and ViewModel](https://developer.android.com/codelabs/android-training-livedata-viewmodel#0)

### Dagger or Dagger Hilt (or Koin)

DIは、Daggerを使うか、Koinを使うか、Dagger Hiltを使うかという感じだと思います。どれにすべきかは本日現在微妙だと思います。来年はDagger Hiltになるかもしれません（わかりません）。無難なのはDaggerという気はしますが（GoogleがForkして開発していて、Googleのドキュメントにも登場する）、学習コストが高いです。Dagger HiltはDaggerをAndroidで使いやすくしたやつです。yanzm氏のMaster of Daggerを読みましょう...。

- [Master of Dagger あんざいゆき第2版【技術書典9 新刊】 \- TechBooster \- BOOTH](https://booth.pm/ja/items/1577764)

### Epoxy

リスト表示を楽にしてくれるやつです。GroupieというライブラリもありEpoxyと人気を二分していた気もしますが、今見たらLast Updateからしばらく時間が経っており、あまりメンテナンスされてないような...。EpoxyはAirbnbがちゃんとメンテナンスしている気がするので、Epoxyの方が無難だと思います。

- [airbnb/epoxy: Epoxy is an Android library for building complex screens in a RecyclerView](https://github.com/airbnb/epoxy)

### Glide or Coil

画像を読み込むやつはGlideかCoilでしょうか。PicasoとかFrescoとかもありますが、今はGlideかCoilかな...。Coilの方が読み込みが速いと聞いた気がする（未確認）。

- [bumptech/glide: An image loading and caching library for Android focused on smooth scrolling](https://github.com/bumptech/glide)
- [coil\-kt/coil: Image loading for Android backed by Kotlin Coroutines\.](https://github.com/coil-kt/coil)

番外編でLottieを使うとアニメーションが楽です（Lottie形式で書き出せる人がいれば）。

- [airbnb/lottie\-android: Render After Effects animations natively on Android and iOS, Web, and React Native](https://github.com/airbnb/lottie-android)

## まとめ

あっ、最後の方はAndroid入門というよりライブラリの紹介になってしまった。まあいいや...。

ということで、フラー Advent Calendar 2020も完走ですね。おつかれさまでした。また来年！
