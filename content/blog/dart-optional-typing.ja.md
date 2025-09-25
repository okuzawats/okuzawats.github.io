---
title: "Dartのオプショナルタイピング（任意の型付け）"
date: 2025-09-25T21:09:00+09:00
description: "Dartのオプショナルタイピングについてです。"
categories: ["Dart"]
draft: false
---

Dartでは型付けは任意であり、型付けしても良いし、しなくても良いです。この型付けをしてもしなくても良いということを、 **オプショナルタイピング（Optional Typing）** と呼びます。

例えば、以下の関数定義は完全に正しいDartのコードです。それぞれ `void` 、 `String` に型付けをしていませんが、Dartのコードとして実行可能です。

```dart
a() => print('a');
```

```dart
b() => 'b';
```

型付けが任意であるということは、型付けをしても良いということです。

```dart
void a() => print('a');
```

```dart
String b() => 'b';
```

変数は `var` または `final` を用いて以下のように宣言できます。

```dart
var n = 42;
```

```dart
final n = 42;
```

[Effective Dart](https://dart.dev/effective-dart/usage#do-follow-a-consistent-rule-for-var-and-final-on-local-variables)では、ほとんどのローカル変数については型付けすべきでないとしています。

> Most local variables shouldn't have type annotations and should be declared using just `var` or `final`. 

逆に言えば、それ以外の場合については型付けした方が良いでしょう。

## Reference

1. [The Dart type system](https://dart.dev/language/type-system)（最終アクセス日：2025年9月25日）
2. [Google Dartのエッセンス:アプリケーションの構築、スナップショット、Isolate](https://www.infoq.com/jp/articles/google-dart/)（最終アクセス日：2025年9月25日）
3. [DO follow a consistent rule for var and final on local variables](https://dart.dev/effective-dart/usage#do-follow-a-consistent-rule-for-var-and-final-on-local-variables)（最終アクセス日：2025年9月25日）
4. Chris Buckett, (2014), 「プログラミング言語 Dart」, 角川
