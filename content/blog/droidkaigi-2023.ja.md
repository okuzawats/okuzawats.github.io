---
title: "DroidKaigi 2023に行ってきた"
date: 2023-09-15T22:20:25+09:00
description: "DroidKaigi 2023に行ってきました。"
categories: ["Other"]
draft: false
---

昨日と今日の2日間、[DroidKaigi 2023](https://2023.droidkaigi.jp/)に行ってきました（明日の3日目も開催されますが、自分は不参加です）。数年振りのオフライン参加で、いろいろな人と会って話せたので楽しかったです。忘れないうちにセッションの感想を残しておきます。

## Android Open-source repositories 101: Everything you always wanted to know

OSSの管理方法について扱ったセッションでした。自動化が大事だという話をされてましたが、自動化のスキルは実務に役立つことも多そうです。OSS活動、なかなかできてないですが、やりたい気持ちだけはもう何年もあります。

[Android Open-source repositories 101: Everything you always wanted to know | DroidKaigi 2023](https://2023.droidkaigi.jp/timetable/483318/)

## Supercharging Android Testing: Scalable Environments for Large Teams on Diverse Devices & Features

AndroidはOSバージョン、スクリーンサイズ、メーカーをたくさんサポートしなくてはならない、というようなことを言っていた記憶があります。事業成長のためにはテストがスケールしないといけないと言っていた点については耳が痛いのと同時に頑張っていこうと思えました。長期のコミットメントとイテレーティブなマインドセットが必要である、とも言っていたので心の支えにします。

あとはテスト結果のトレンドをモニタリングできるダッシュボードを整備して、アラートを出すようにすると良い、とのこと。確かにダッシュボードがあれば良さそうですが、準備が大変そうだとも思いました。

[Supercharging Android Testing: Scalable Environments for Large Teams on Diverse Devices & Features | DroidKaigi 2023](https://2023.droidkaigi.jp/timetable/494775/)

## (Unofficial) Guide to App Architecture Guide Vol. 2

Hiltについての話が多かったです。HiltはComponent Treeから外れたことをしようとすると難しいという話をしていて、例えば `@AccountScope` みたいなやつを定義できればマルチアカウントが楽になるけどできないという話をしていました。Daggerの拡張である[square/anvil](https://github.com/square/anvil)や、さらにその拡張である[deliveryhero/whetstone](https://github.com/deliveryhero/whetstone)の話もありました。

マルチモジュールまわりの話は耳が痛かったです（すみません）。

[(Unofficial) Guide to App Architecture Guide Vol. 2 | DroidKaigi 2023](https://2023.droidkaigi.jp/timetable/494987/)

## モニタリングでパフォーマンス改善入門

立ち見になってしまってメモが取れませんでした。最初の10分と最後の5分は見れましたが、あとの時間はブースを回ってました。アーカイブでまた視聴します。

[モニタリングでパフォーマンス改善入門 | DroidKaigi 2023](https://2023.droidkaigi.jp/timetable/495100/)

## Maintaining E2E Test Automation as We Transition from View to Compose

E2EテストをAndroid ViewからComposeに移行した時の話でした。リリースサイクルを2週間から1週間にした、というような話をされており、それを支えるのがE2Eテストとのこと。リリースサイクルを早めることに興味があったので、E2Eテストも重要だなという感想を持ちました。

[Maintaining E2E Test Automation as We Transition from View to Compose | DroidKaigi 2023](https://2023.droidkaigi.jp/timetable/495024/)

## MVIに基づくStateMachineアーキテクチャ：KMMとJetpack ComposeとSwiftUIを組み合わせる

Processor、Reducerを用いてMVIアーキテクチャを構築する話。UMLからKotlinのDSLを作る話は面白かったです。自分では取ることのないアプローチだと思ったので、関心しました。

また、オフィスアワーで質問をさせてもらい、ProcessorでのIntentのprocessとReducerでのActionのreduceが組合せ爆発しそうだけど、その組み合わせのテストどうするのって聞きました。スライドの最後の方にあったロードマップに入っているということと、重要なところだけテストを書いているとのことでした。

[MVIに基づくStateMachineアーキテクチャ：KMMとJetpack ComposeとSwiftUIを組み合わせる | DroidKaigi 2023](https://2023.droidkaigi.jp/timetable/495051/)

## Androidの『音』を制御する

Androidのサウンド制御の話。端末差異がすごいなあと思ったのと、各端末の挙動についてよく調べたなあと感動しました。

[Androidの『音』を制御する | DroidKaigi 2023](https://2023.droidkaigi.jp/timetable/493899/)

## Androidアプリの良いユニットテストを考える

DeNAのSWETのNozomi Takumaさんの発表。去年もCoroutinesのテストの話をされていましたね。去年の発表も今年の発表もとても勉強になりました。

内容としては単体テストの保守性の話。参考文献として「単体テストの考え方 / 使い方」「xUnit Test Patterns」を挙げられていました。前者は自分も読みましたが、後者はかなり読み応えのある本（英語だし、900ページくらいある）なので自分は斜めにしか読んでおらず、通読したのかな、すごいな、と思いました。

Takumaさんのインタビュー（[Androidアプリ開発者の福利厚生？スペシャリストの挑戦](https://engineering.dena.com/blog/2022/10/cto_interview_07/)）も読みましたが（DroidKaigiへの登壇は5年連続だったんですね）、Androidのテストにキャリアを振って専門性を深められているのもすごいなと思っていて、ちょっと自分のキャリアについても考えてしまいました。ファンです。本とか書いて欲しいです。

[Androidアプリの良いユニットテストを考える | DroidKaigi 2023](https://2023.droidkaigi.jp/timetable/495066/)

## Jetpack Compose で Android/iOSアプリを作る

元同僚の[m.coderさん](https://twitter.com/_m_coder)の発表でした。内容はComposeを用いてAndroidアプリとiOSアプリを作るというもの。自分がこのアプローチを取ることはない（少なくとも仕事では）と思いますが、実験としては面白いです。KMP周りのエコシステムも頑張っているなと思いましたが、まだまだ大変そうです。Windows Phoneもあったらもっと大変だったと思います。

[Jetpack Compose で Android/iOSアプリを作る | DroidKaigi 2023](https://2023.droidkaigi.jp/timetable/483040/)

## 中規模アプリにおけるレイヤードアーキテクチャでの例外との向き合い方

アプリ全体で例外をどう体系的にハンドリングするかという話でした。例外のハンドリング設計をアーキテクチャとしてできているといいよねということで、自分もそう思います。例外のハンドリングが場当たり的だと、かなりカオスなコードベースになってしまうと思うので。示されていたサンプルコードは目指す目標としては綺麗だなと思いました。

[中規模アプリにおけるレイヤードアーキテクチャでの例外との向き合い方 | DroidKaigi 2023](https://2023.droidkaigi.jp/timetable/488101/)

## Our ongoing journey from REST to GraphQL on Android

GraphQLの話。2020年にGraph QLが導入されて、今も ongoing とのこと...。何かを思い出して遠い目になってしまいました（多くは語らず）。

[Our ongoing journey from REST to GraphQL on Android | DroidKaigi 2023](https://2023.droidkaigi.jp/timetable/482639/)

## まとめ

数年振りにオフラインでDroidKaigiに参加しました。久し振りに会う方やオフラインで会うのが初めてという方も多くて、楽しかったです。セッションについてはアーキテクチャやテストの話を中心に聞いて、なんというか「頑張ろう」と思いました。

関係者の皆さま、おつかれさまでした＆ありがとうございました。
