---
title: "DartでFizzBuzz"
date: 2025-06-11T13:31:00+09:00
description: "DartでFizzBuzzしました。"
categories: ["Dart"]
draft: false
---

FizzBuzzを書いてDartを味わっていきます。

まずは引数として整数 `n` を受け取り、整数 `n` に対するFizzBuzzを返す関数を実装します。結論から言うとこんな感じで実装しました。

```dart
/// 整数[n]に対するFizzBuzzを返す。
///
/// [n]が3の倍数の場合は"Fizz"、
/// 5の倍数の場合は"Buzz"、
/// 3の倍数かつ5の倍数の場合は"FizzBuzz"、
/// それ以外の場合は[n]の文字列表現を返す。
/// @param n FizzBuzzの元となる整数値
/// @returns [n]のFizzBuzz
String fizzBuzz(int n) {
  var s = '';
  if (n % 3 == 0) s += 'Fizz';
  if (n % 5 == 0) s += 'Buzz';
  return s.isNotEmpty ? s : '$n';
}
```

見どころがいくつかありますが、まずドキュメンテーションコメントです。Dartにおけるドキュメンテーションコメントに関するガイダンスが[Effective Dart: Documentation](https://dart.dev/effective-dart/documentation)に示されており、上記のコードではそのガイダンスの一部に従っています。

- ドキュメンテーションコメントは `///` で始める
- 最初の1行にサマリーを書き、1行開けて詳細を書く
- `[]` を用いて説明内で引数を参照できる
- `@param` `@returns` で引数や返り値の説明を追加することができる

次に、ローカル変数 `s` を `var s = '';` で宣言しています。Effective Dartでは、 `var` を使って型注釈なしでローカル変数を宣言することが推奨されています。ローカル変数であれば `final` も使わなくてOKです（今回のコードだと再代入を行なっているのでそもそも `final` にはできませんが）。

> Most local variables shouldn't have type annotations and should be declared using just var or final. There are two rules in wide use for when to use one or the other:
> - Use final for local variables that are not reassigned and var for those that are.
> - Use var for all local variables, even ones that aren't reassigned. Never use final for locals. (Using final for fields and top-level variables is still encouraged, of course.)
> 
> [DO follow a consistent rule for var and final on local variables](https://dart.dev/effective-dart/usage#do-follow-a-consistent-rule-for-var-and-final-on-local-variables)

次の行では、 `if (n % 3 == 0) s += 'Fizz';` で `s` に文字列を再代入しています。Dartの[if](https://dart.dev/language/branches#if)には特に説明を見つけられませんでしたが、 `if (Boolean) Statement` というシグネチャであれば良く、真偽値がtrueの場合に実行されるStatementはブロック `{}` で囲っても囲わなくても良さそうです。個人的には、今回のコードで行なっているようなブロックの省略をせず、ブロックで囲んだ方がベターだとは思います（[というような話を3年ほど前にしました](https://speakerdeck.com/okuzawats/kotlinnoifwoai-deru)）。

最後の `return s.isNotEmpty ? s : '$n';` も味わい深いです。まず `?` 演算子ですが、これはいわゆる三項演算子です。 `Boolean ? Left Expression : Right Expression` というシグネチャで、真偽値がtrueであればLeft Expressionの評価値を、falseであればRight Expressionの評価値を返します。また、 `'$n'` の記法は文字列テンプレートで、ここでは `n` を文字列に展開しています。

ここまででFizzBuzzを返す関数を実装し、味わってきました。なかなか味わい深かったことと思います。さらに、この関数をループ内で呼び出して1から100のFizzBuzzを標準出力します。

こちらについても結論から言うとこんな感じで実装しました。こちらも味わっていきます。

```dart
import 'src/fizzbuzz.dart';

void main() {
  Stream.fromIterable(
    List.generate(100, (i) => ++i)
  )
  .map(fizzBuzz)
  .listen(print);
}
```

まずは `import 'src/fizzbuzz.dart';` で、先ほど書いた `fizzBuzz` 関数を参照できるようにしています。この関数を `src/fizzbuzz.dart` に置いているので、このようなimportになっています。

次に `main` 関数ですが、[DartでHello World](https://okuzawats.com/blog/dart-hello-world/)で書いたようにエントリーポイントとなります。

以下の部分はDartの特徴が表れる部分かと思います。これは1から100までのストリームを作っている処理です。まずList#generateを使ってイテラブルであるListを作り、さらにString#fromIterableでストリームを作っています。 `1, 2, ..., 100` までの整数値のストリームを返します。

```dart
  Stream.fromIterable(
    List.generate(100, (i) => ++i)
  )
```

List#generateについてはDartのリスト内包表記を使って `[for (var i = 1; i <= 100; i++) i]` というように書いても良いですが、List#generateを使った方が美しい気がしてます。

次に以下の部分ですが、これはStreamをマップする `map` 関数の引数として `fizzBuzz` 関数を渡しています。 `map` 関数にはラムダを渡すこともできます（例： `.map((n) => n * 2)` ）。

```dart
  .map(fizzBuzz)
```

ストリームの末端で標準出力します。末端処理に `listen` 関数を使います。

```dart
  .listen(print);
```

早速実行してみます。

```bash
% dart main.dart 
1
2
Fizz
4
Buzz
...以下略
```

🎉

## Reference

1. [Fizz Buzz - Wikipedia](https://ja.wikipedia.org/wiki/Fizz_Buzz)（最終アクセス日：2025年6月11日）
2. [Effective Dart: Documentation](https://dart.dev/effective-dart/documentation)（最終アクセス日：2025年6月11日）
3. [DO follow a consistent rule for var and final on local variables](https://dart.dev/effective-dart/usage#do-follow-a-consistent-rule-for-var-and-final-on-local-variables)（最終アクセス日：2025年6月11日）
4. [If](https://dart.dev/language/branches#if)（最終アクセス日：2025年6月11日）
5. [Kotlinのifを愛でる](https://speakerdeck.com/okuzawats/kotlinnoifwoai-deru)（最終アクセス日：2025年6月11日）
6. [DartでHello World](https://okuzawats.com/blog/dart-hello-world/)（最終アクセス日：2025年6月11日）
