---
title: "[Android] EventBusの思い出🚌"
date: 2022-12-23T00:00:00+09:00
description: "フラー株式会社 Advent Calendar 2022の23日目の記事「EventBusの思い出🚌」です。"
categories: ["Android", "Design"]
draft: false
---

この記事は[フラー株式会社 Advent Calendar 2022](https://qiita.com/advent-calendar/2022/fuller-inc)の記事です。22日目は `@Daiji256` による「[onClick VS onClicked](https://daiji256.github.io/posts/diary/onclick-onclicked/)」でした。

さて、6日目の記事「[Androidエンジニアになってから4年経った振り返り](https://nanaten.github.io/blog/for-four-years/)」を読みました。自分は、上記の記事を書いた `@m-coder` より少し前にAndroidアプリ開発を始めたのですが、記事を読むと、この4年間やその前にあったいろいろな出来事が思い出されてきます。自分としたことがちょっとセンチメンタルな気分になってしまいました...。人生いろいろありますね。

ともかく、上記記事ではRxJavaについて触れられていました。これを読んでいて思い出したのが、RxJavaが広く使われるようになった、そのもう少し前。燎原の火のごとく燃え広がり、RxJavaの流行と共にAndroidアプリ開発の歴史に消えていったものがありました。本記事はソレについての思い出を書き記し、Androidアプリ開発史における歴史の一里塚とすることを意図したものです。

自分の理解不足や記憶違いがあればご指摘ください。予防線を張ったところで、始めていきます。

## EventBus

RxJavaが流行する前、**EventBus**なるものが流行していました。

EventBusとは、特定のライブラリというよりは、イベント通知の設計を指します。その実装を提供する代表的なライブラリとしては、Square製のOttoや、greenrobot製のEventBusが存在します。5年以上前の歴史については明るくないのですが、おそらくGuavaの流れを汲んでいたものと思います。

- [square/otto: An enhanced Guava-based event bus with emphasis on Android support.](https://github.com/square/otto)
- [greenrobot/EventBus: Event bus for Android and Java that simplifies communication between Activities, Fragments, Threads, Services, etc. Less code, better quality.](https://github.com/greenrobot/EventBus)

前者のOttoについては、[今から6年前の2016年にDeprecatedとなり](https://github.com/square/otto/commit/008bd6e3bf979522fee36a5c73036dcf0a7fa486)、本日までにプロジェクトがアーカイブされています。後者のEventBusについてはDeprecatedになったりアーカイブになったりこそしていませんが、それほど活発にメンテナンスがされている状況ではありません。こういった状況を踏まえると、EventBusは既にAndroidアプリ開発のメインストリームからは外れていると言えそうです。

それでは、EventBusは何故流行し、そして何故消えていったのでしょうか？

### Callback Hell

さて、EventBusとはイベント通知の設計である、と書きました。以下の図に、EventBusによる基本的なイベント通知のフローを示します。Publisher、すなわちイベントの発行者は、EventBusに向けてイベントを通知します。Subscriber、すなわちイベントの購読者は、EventBusからイベントを購読します。この図に示されるEventBusは、大域的に存在するインスタンスです。

{{< figure src="/images/eventbus-1.webp" title="Fig-1 EventBus" class="center" >}}

※ 画像出典：[greenrobot/EventBus: Event bus for Android and Java that simplifies communication between Activities, Fragments, Threads, Services, etc. Less code, better quality.](https://github.com/greenrobot/EventBus)

すなわち、EventBusがハブとなり、イベントのPublish/Subscribeが行われます。これの何が嬉しいのかというと、EventBusを経由することで複雑な制御フローをバイパスし、イベントの発行・購読ができるということです。

典型的には、以下の記事に記載されているようなユースケースです。

> コールバックメソッドを呼ぶタイミングでイベントを発火し、コールバックインタフェースの実装ではなく、イベントを常時監視するメソッドを用意しておいて、発火したタイミングで呼び出されるようにする仕組みとして、EventBus を使うことで、インタフェースによる依存関係を整理したり、コールバック地獄を解消したりすることができる。
> 
> [Android でイベントバスを使う - Qiita](https://qiita.com/KeithYokoma/items/793aaac6994c9242808f)より引用

この背景にあるのは、当時のAndroidアプリ開発では、非同期処理の結果を通知するための手段が乏しく、コールバックのために定義されるインターフェースを介して結果を受け取っていたという事情があります。この手法には、非同期処理の結果を受け取ってさらに非同期処理を呼び出す場合に、コールバックのネストが深くなっていくいわゆる「**Callback Hell（コールバック地獄）**」という問題がありました。

そこで上記の記事のように、Callback Hellを解決するためにEventBusが導入されるケースがあったようです。当時の時代背景を考えれば、この技術選択は理にかなっていたと言えるのでしょう。

### Dark Side of the Force

EventBusの力はそれに留まりません。例えば以下の記事では、画面（ActivityやFragment）間での通信にEventBusを使えることが（記事執筆時のベストプラクティスとして）示唆されています。

> AndroidはActivity間の通信として複雑なデータ(たとえばJava Object)を送るAPIを提供していない。Fragmentであれば、その子Fragment間の通信経路としてActivityのインスタンスを使う事ができる。とはいえ、Ottoやgreenrobot-EventBusを使ってEventBusを使いたいと思うだろう。他のライブラリの追加を避けたい場合はRxJavaを用いてEventBusを実装する事も可能である。
> 
> [Androidの開発におけるベストプラクティス](https://github.com/futurice/android-best-practices/blob/master/translations/Japanese/README.ja.md)より引用

つまり、画面Aであるイベントがトリガーされた時に画面Bの表示を更新する、というようなユースケースがその典型です。Androidフレームワークの制限を考えると、当時はこういったユースケースでEventBusを使用することも理にかなっていた、と言えるのではないでしょうか？（自分もやりました）

EventBusのパワーは事程左様に強力ですが、強力すぎる故に、濫用された時の副作用もまた大きいものでした。EventBusがひとたび濫用されてしまえば、どこからどのようにイベントが飛んでくるのかわからない、何がトリガーとなって処理が行われるのかわからない、大変な状況に陥ります。

イメージとしてはこんな感じでしょうか。

{{< figure src="/images/eventbus-2.webp" title="Fig-2 今ので気付いた奴もいるのか" class="center" width="300" >}}

※ 画像出典：冨樫義博、（2003）、HUNTER X HUNTER 16巻、集英社

間合いを無効化して一方的に殴られるのではボクシングになりません（アプリ開発はボクシングです）。もちろん用法容量を守って使用すれば強力な武器になりますが、強力な武器は得てして濫用されるものです。

緻密に設計した制御フローを無視してイベントの通知・購読ができるEventBusの力は、人類には過ぎたものでした。人類はまだ、EventBusを使いこなすほどに成熟してはいないようです。マジック・ザ・ギャザリングにおける禁止カードと同様に、その強すぎる力から、次第にEventBusの使用は禁忌となっていきました。

### Everyday at the Bus Stop

EventBusが猛威を振るっている頃、リアクティブプログラミングのためのライブラリ群、ReactiveXのJava実装であるRxJava（およびRxAndroid）もまた、Androidアプリ開発に広がり始めていました。RxJavaはリアクティブプログラミングのためのライブラリですが、RxJavaを活用することで非同期処理によるCallback Hellを回避することもできました。RxJavaではすべてを観測可能なオブジェクトとして扱いますが、非同期処理もその例外ではありません。このCallback Hellを回避するというユースケースにおいて、RxJavaはEventBusの代替手段となり得ました。

またこの頃、俗に「RxBus」と呼ばれる、RxJavaによるEventBusの実装も一部で用いられることがありました。このRxBusについて一言で説明すると、イベントのPublish/Subscribeのハブとなる（RxJavaの）Observableを大域的な領域に置くことで、通常の制御フローをバイパスしてイベント通知ができるというものです。これは用いるツールがRxJavaになったというだけで、EventBusが潜在的に抱える問題を解決したものではありません。EventBusからRxJavaへの移行過渡期を象徴する特徴的な実装だったと言えるでしょう。

EventBusの強力すぎる力の源泉は、緻密に設計された制御フローを無視してイベント通知ができてしまうことにありました。RxJavaのObservableによるリアクティブストリームは制御フローに乗せてイベント通知などを行うもので、その一面を取ればEventBusの強力すぎる力を封じたものという見方もできます（RxJavaはそれだけにとどまるものではないですが）。

### そしてRxJavaへ

このようにしてEventBusの流行は過ぎ去り、RxJavaの時代が来ました。この背景には、Androidアプリにおけるアーキテクチャの議論の進展もあったものと思います。一言で言えば、ワープして殴るのではなく、真っすぐいって右ストレートでぶっとばすスタイルに落ち着いたと言えるのではないでしょうか。

{{< figure src="/images/eventbus-3.webp" title="Fig-3 右ストレートでぶっとばす" class="center" width="300" >}}

※ 画像出典：冨樫義博、（1993）、幽遊白書 14巻、集英社

## まとめ

本記事では、Androidアプリ開発におけるEventBusの思い出を振り返ってみました。明日は `@SaturnR7` で「[縦書きの日本語をウィジェットで表現したい](https://qiita.com/SaturnR7/items/48b40782609d3faf8f5d)」です。

## 参考文献

1. [AIが語る「与謝野晶子はなぜ力道山を殺さなかったのか」 - 趣味人の宿部屋](https://yadobeya.blog.fc2.com/blog-entry-4452.html)
2. [アヤブサPさんはTwitterを使っています: 「与謝野晶子はなぜ力道山を殺さなかったのかはマジで名コラ https://t.co/2BOIMDg0HP」 / Twitter](https://twitter.com/phoenix_ayabusa/status/1429042856358862853)
3. [Androidエンジニアになってから4年経った振り返り | m.coder's Blog | m.coder's Blog](https://nanaten.github.io/blog/for-four-years/)
4. [square/otto: An enhanced Guava-based event bus with emphasis on Android support.](https://github.com/square/otto)
5. [greenrobot/EventBus: Event bus for Android and Java that simplifies communication between Activities, Fragments, Threads, Services, etc. Less code, better quality.](https://github.com/greenrobot/EventBus)
6. 冨樫義博、（2003）、HUNTER X HUNTER 16巻、集英社
7. 冨樫義博、（1993）、幽遊白書 14巻、集英社
