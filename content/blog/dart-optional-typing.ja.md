---
title: "Dartのオプショナルタイピング（任意の型付け）"
date: 2025-09-25T21:09:00+09:00
description: "Dartのオプショナルタイピングについてです。"
categories: ["Dart"]
draft: false
---

Dartでは型付けは任意であり、型付けしても良いし、しなくても良いです。この型付けをしてもしなくても良いということを、 **オプショナルタイピング（Optional Typing）** と呼びます。Dartの初期からDartはオプショナルタイピングです。

プショナルタイピングは、Dartが当時altJSを目指して開発されていた中で、戦略的に導入された言語機能なのかもしれません。またそれもあってか、元々の言語設計者の意図としては、「型付けありがデフォルト」というより、「型付けなしがデフォルト」であるという思想を感じました。なお、この1パラグラフは完全に個人の感想で根拠は全くありません。

さて、具体的なコードの例としては以下の関数定義です。この関数定義は完全に正しいDartのコードです。それぞれ `void` 、 `String` に型付けをしていませんが、Dartのコードとして実行可能です。

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

変数に型付けをする場合は、以下のように宣言します。

```dart
int n = 42;
```

```dart
final int n = 42;
```

[Effective Dart](https://dart.dev/effective-dart/usage#do-follow-a-consistent-rule-for-var-and-final-on-local-variables)では、ほとんどのローカル変数については型付けすべきでないとしています。

> Most local variables shouldn't have type annotations and should be declared using just `var` or `final`. 

逆に言えば、それ以外の場合については型付けした方が良いでしょう。

一般論として、スコープが広くなるほどコードのコンテキストを把握するための認知負荷は高まります。型付けは、人間がコードを読む時の大きなヒントとなります。ローカル変数であればコンテキストの把握に大きな認知負荷はかからないので型付けをしなくても読める、または型付けをしない方が素早く読める場合もあるのでしょうが、スコープが広い場合は人間がコードを読むためには型付けがあった方が好ましいと考えています。

また、関数や変数のスコープが広い場合はその型が暗黙のうちに変わってしまうことの影響が大きく、型付けによって型を明示的に宣言することは有益です。

## Reference

1. [The Dart type system](https://dart.dev/language/type-system)（最終アクセス日：2025年9月25日）
2. [Google Dartのエッセンス:アプリケーションの構築、スナップショット、Isolate](https://www.infoq.com/jp/articles/google-dart/)（最終アクセス日：2025年9月25日）
3. [DO follow a consistent rule for var and final on local variables](https://dart.dev/effective-dart/usage#do-follow-a-consistent-rule-for-var-and-final-on-local-variables)（最終アクセス日：2025年9月25日）
4. Chris Buckett, (2014), 「プログラミング言語 Dart」, 角川
