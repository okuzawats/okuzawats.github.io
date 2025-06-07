---
title: "DartでHello World"
date: 2025-06-07T23:14:00+09:00
description: "DartでHello Worldしました。"
categories: ["Dart"]
draft: false
---

ひさしぶりにDartを書きたい気持ちがむくむくと湧いてきたので、まずはHello Worldしてみます。

動作環境：
- MacBook Pro 2022
- Apple M2
- macOS 15.5

## Dart SDKのインストール

[Build a web app with Dart](https://dart.dev/web/get-started)に従うと、Homebrewを用いて以下のようにDart SDKをインストールできます（できました）。

```
% brew tap dart-lang/dart
% brew install dart
```

が、この方法でインストールした場合、Dartのバージョンをプロジェクトごとに管理したい...という時に困ることになります。なので、今回はこの方法は採用しません。上記のコマンドでインストールしたDart SDKは消しておきます。

```bash
% brew uninstall dart
% brew untap dart-lang/dart
```

代わりに `asdf` を使います。 `asdf` をまだ使ってない場合は、[Getting Started](https://asdf-vm.com/guide/getting-started.html)に従ってセットアップします。セットアップしたら、以下のコマンドでDartをインストールします。

```bash
% asdf plugin add dart
% asdf install dart 3.8.1
% asdf set dart 3.8.1
```

`.tool-versions` が作成されて、その中身はこんな感じになってます。 `.tool-versions` の挙動については `asdf` のドキュメントを参照してください。

```
dart 3.8.1
```

## DartでHello World

Dart SDKをインストールできたので、Hello Worldします。こんな感じのコードを `helloworld.dart` として保存します。適当に書きましたが、[Introduction to Dart](https://dart.dev/language)と同一のコードでした。

```dart
void main() {
  print('Hello, World!');
}
```

Dartは、エントリーポイントとして `main()` 関数を要求します。また、その返り血の型は `void` でなければなりません。インデントが半角スペース2つなのは自分好みです。

上記のコードを実行します。以下のように実行できます。

```bash
% dart helloworld.dart
Hello, World!
```

🎉

## Reference

1. [Build a web app with Dart](https://dart.dev/web/get-started)（最終アクセス日：2025年6月7日）
2. [Getting Started | asdf](https://asdf-vm.com/guide/getting-started.html)（最終アクセス日：2025年6月7日）
3. [Introduction to Dart](https://dart.dev/language)（最終アクセス日：2025年6月7日）
