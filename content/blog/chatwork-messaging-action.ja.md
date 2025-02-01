---
title: "Chatworkにメッセージを送信するGitHub Actionsを公開しました🎉"
date: 2022-08-31T13:32:00+09:00
description: "Chatworkにメッセージを送信するGitHub Actionsを公開しました。"
categories: ["GitHub Actions"]
draft: false
aliases:
    - /blog/send-message-to-chatwork-with-github-actions/
---

GitHub ActionsからChatworkに何かを通知したいことがありませんか？僕はあります。例えば、CIが成功した時は成功したことをChatworkに通知して欲しいですし、CIが失敗した時は失敗したことをChatworkに通知して欲しいです。GitHub Actionsが元気に動いているのを眺めているのは楽しい趣味ですが（そして時に必要なことでもありますが）、大抵の場合は結果だけをチャット（ここではChatwork）に通知してくれれば足ります。

結論としては、GitHub ActionsからChatworkにメッセージを送信するためのActionをGitHubのマーケットプレースに公開しました🎉こちらのActionを使っていただければ、GitHub ActionsからChatworkへ、簡単にメッセージを送信できるようになるかと思います。

- [Chatwork Messaging Action · Actions · GitHub Marketplace](https://github.com/marketplace/actions/chatwork-messaging-action)

ちなみに、GitHubのマーケットプレースを探すと、既に以下のアクションが存在しました。どのActionもPull Requestに関する通知を送ってくれるActionのようです。自分はCIの結果をChatworkに通知する、というようなユースケースを満たすActionが欲しいと考えていたので、これらのActionはちょっと目的が違うかなと思いました。

- [chatwork-actions · Actions · GitHub Marketplace](https://github.com/marketplace/actions/chatwork-actions)
- [Pull Request Chatwork Notifier · Actions · GitHub Marketplace](https://github.com/marketplace/actions/pull-request-chatwork-notifier)
- [Open Pull Requests Chatwork Notifier · Actions · GitHub Marketplace](https://github.com/marketplace/actions/open-pull-requests-chatwork-notifier)

## 使い方

以下にサンプルとなるGitHub Actionsのワークフローを示します。 `okuzawats/chatwork-messaging-action` を使用している箇所に着目してください。

```yml
name: send message
on:
  push:
    branches: [main]
jobs:
  message:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      - name: set current datetime to env
        env:
          TZ: 'Asia/Tokyo'
        run: echo "CURRENT_DATETIME=$(date +'%Y-%m-%d %H:%M:%S')" >> $GITHUB_ENV
      - name: set arguments
        id: arguments
        run: echo '::set-output name=message::"[info][title](beer)what time is it now?[/title]it is ${{ env.CURRENT_DATETIME }} now.[/info]"'
      - uses: okuzawats/chatwork-messaging-action@v1.0
        with:
          apiToken: ${{ secrets.API_TOKEN }}
          roomId: ${{ secrets.ROOM_ID }}
          message: ${{ steps.arguments.outputs.message }}
```

このActionを使う時に必要となるのは、APIのトークン（ `apiToken` ）、チャットルームのID（ `roomID` ）、そして送信するメッセージ（ `message` ）です。いずれもこのActionを使用する上で必須のパラメータとしています。

APIトークンは以下から利用申請を行うと取得できるものです。こちらはGitHubのシークレットに入れて使うことを想定しています。

- [Chatwork APIへようこそ！](https://developer.chatwork.com/docs)

チャットルームのIDは、「チャットの説明 > グループチャットの設定 > チャット情報 > 画面下部のルーム ID」というように進んで確認できる数字です。ブラウザ版のChatworkでチャットルームを開いた時のURLに含まれるrid以下の数字も同じものです。ブラウザ版を使っているのであれば、URLを確認するのが一番早いかもしれません。こちらも一応シークレットに入れて使うことを想定しています。

最後のパラメータは、Chatworkに送信したい文字列です。Chatworkのメッセージは、「メッセージ記法」というものを使ってリッチにできます。メッセージ記法については、以下のリンクを参照してください。また、もしActionで使用できないメッセージ記法があれば、Issueを立てていただけるとありがたいです。

- [メッセージ記法について](https://developer.chatwork.com/docs/message-notation)

## Reference

1. Chatwork, Chatwork APIへようこそ！, retrieved from https://developer.chatwork.com/docs （最終アクセス日：2022年8月31日）
2. Chatwork, メッセージ記法について, retrieved from https://developer.chatwork.com/docs/message-notation （最終アクセス日：2022年8月31日）

[^1]: 環境構築的に楽そうな気がしました。
[^2]: Rubyはこういう処理を書くのが得意そう、という先入観があります。
