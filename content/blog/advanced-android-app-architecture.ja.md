---
title: "「Advanced Android App Architecture」を読みました📚"
date: 2022-01-31T22:10:37+09:00
description: "「Advanced Android App Architecture」を読んだ感想です。"
categories: ["Android", "Reading", "Design"]
draft: false
---

「**Advanced Android App Architecture**」 (Cheng、Olivares (2019))を読みました。Androidアプリ開発におけるアーキテクチャの基礎がまとまっていてよい本でした。もう少し早く読んでおけばよかったな、と思います。あと、2019年に出版された書籍の内容がこんなにも早く古びてしまうのか、とも思いました（後述）。

## 何故、アーキテクチャを構築するのか

アーキテクチャが何故重要なのかという点について、最初の方に議論があります。extensibleでmaintenableでtestableなアプリを作るためのベストプラクティスのパターンがアーキテクチャとしてまとまっているということ、アーキテクチャを構築することで関心事が分離されてユニットテストが書きやすくなってこれらの目的が達成されるということ、などが書かれています。

だいぶ端折りましたが、大筋としてはこんな感じのことが書かれていたと思います。

## 何故、AndroidアプリではMVCがうまくいかないのか

アーキテクチャと聞くと真っ先に思い浮かぶのはMVCだと思うんですが、Androidアプリ開発だとMVCがうまく行かない理由が書いてありました。これを読んで思い出したのが `@konifar` さんの以下の記事です。もう7年も前の記事ですか。月日が経つのは早いものですね。

* [AndroidではMVCよりMVPの方がいいかもしれない - Konifar's WIP](https://konifar.hatenablog.com/entry/2015/04/17/010606)

Androidアプリ開発（プログラミング）初心者だった私は、この記事を何度何度も読みましたが、結局のところ、何でAndroidアプリ開発でMVCがうまくいかないのかは理解できませんでした。Advanced Android App Architectureにはこのあたりの話が詳しく書かれていてわかりやすいので、興味がある方は読んでみてください。

余談ですが、MVCという概念がなかなか理解できませんでした。自分は初心者の頃にAndroidだけをやっていた時期が長かったわけですが、振り返って考えると、Androidアプリ開発ではMVCはうまくいかないので、MVCで考えようとしてもうまくいかないわけですね。何故ならうまくいかないので。うまくいっていないものを見て理解しようとしても理解できないわけです。自分はRailsに触れた時にMVCってそういうことね、と閃きました。

## MVP、MVVM

MVCがうまくいかないとする一方で、Androidアプリ開発において、MVP / MVVMがその問題をどうやって解決しているのかという話が続きます。一言で超訳すると、MVCがActivity / Fragmentから責務を引き剥がせない一方で、MVP / MVVMだとActivity / Fragmentからうまく責務を引き剥がせるということでしょう。「何故、AndroidアプリではMVCがうまくいかないのか」の答えの一つになるかと思いますが、コードがActivity / Fragmentに書かれると途端に単体テストが困難になってしまうわけなんですよね。

MVP / MVVMにすることで、Activity / FragmentをViewとして振る舞わせ、関心事をActivity / Fragmentから分離することで、単体テストが突如として書きやすくなるということです。

## VIPER、MVI

さらに応用的なアーキテクチャとして、VIPERとMVIが紹介されています。VIPERはMVPに近いアーキテクチャで、RouterやInteractorといったレイヤが設けられたものだと理解しています。MVIもMVPに近い気はしますが、リアクティブ / ファンクショナルな考え方を取り入れているもののように思います。

これらの発展的なアーキテクチャが存在するにもかかわらず、MVPやMVVMがAndroidアプリのアーキテクチャとして語られることが多い（要出典）のはどういう理由か、ということを考えてみると、VIPERやMVIというアーキテクチャがMVPやMVVMに比べて重厚なんだろうなと思います。アプリやチームの規模がさほど大きくなければMVPやMVVMといったアーキテクチャで事が足り、そこから成長していくにつれ、抽象化層を増やし、関心事の分離を進めたアーキテクチャが欲しくなってくるのであろう、ということです。

Advanced Android App Architectureにも書かれていますが、穴を掘るのにスプーンを使うのかシャベルを使うのかは、掘る穴の大きさに依るわけです。

## RxJava

冒頭に「2019年に出版された書籍の内容がこんなにも早く古びてしまうのか、とも思いました（後述）」と書いたわけですが、その理由がRxJavaです。本書では、多くのサンプルコードにRxJavaが使われています。2019年から2022年の間に、Kotlin Flowが存在感を増しました。今、同様の書籍を書くのであれば、きっとRxJavaよりもFlowを使ったサンプルコードが増えるのではないかと思います。この界隈、技術の移り変わりが激しいですね。

## Reference
1. Advanced Android App Architecture、Yun Cheng、Aldo Olivares（2019）、Yun Cheng & Aldo Olivares、raywenderlich.com
2. [AndroidではMVCよりMVPの方がいいかもしれない - Konifar's WIP](https://konifar.hatenablog.com/entry/2015/04/17/010606)（最終アクセス日：2022年1月31日）
