---
title: "Fizz Buzz"
date: 2026-09-26T10:00:00+09:00
description: "Fizz Buzzについて。"
categories: ["Other"]
draft: false
---

Fizz Buzzとは、以下のようなゲームです。

> プレイヤーは円状に座る。最初のプレイヤーは「1」と数字を発言する。次のプレイヤーは直前のプレイヤーの発言した数字に1を足した数字を発言していく。ただし、3の倍数の場合は「Fizz」（Bizz Buzzの場合は「Bizz」）、5の倍数の場合は「Buzz」、3の倍数かつ5の倍数の場合（すなわち15の倍数の場合）は「Fizz Buzz」（Bizz Buzzの場合は「Bizz Buzz」）を数の代わりに発言しなければならない。発言を間違えた者や、ためらった者は脱落となる。（Wikipediaより引用）

このゲームから派生してプログラミングのテーマとなっていることもWikipediaのとおり。新しいプログラミング言語を学ぶ時、基本的な文法、ユニットテスト、CIサーバーでの実行などを手を動かして学ぶためにちょうど良いので、自分はプログラミング言語を学ぶ時にしばしばFizz Buzzを書いてます。

## Swift

例えばSwiftの場合は以下のようなコードを書きました。この時はfor文で書いてます。実務ではこのようなコードを書くことはないでしょうが、Fizz Buzzとしては心地の良いコードです。

```swift
for i in 1...100 {
  var s = ""
  if i % 3 < 1 { s += "Fizz" }
  if i % 5 < 1 { s += "Buzz" }
  print(s != "" ? s : "\(i)")
}
```

以下のように実行できます。

```console
% swift -v 
Apple Swift version 5.7.2 (swiftlang-5.7.2.135.5 clang-1400.0.29.51)
% swift fizzbuzz.swift
```

※ このコードは2023年2月に書かれた

## Lua

次はLuaです。この時はFizz Buzzの判定処理を `function` に切り出してます。Swiftの時と同様の発想で書いてますが、Luaに独特の文法が楽しいです。

```lua
--- FizzBuzzを実行し、文字列を返す。
-- @param n 対象となる整数
-- @return nが3の倍数の場合はFizz、5の倍数の場合はBuzz、3と5の倍数の場合はFizzBuzz、それ以外の場合はnを文字列に変換したものを返す。
function fizzbuzz(n)
  s = ""
  if n % 3 < 1 then s = s.."Fizz" end
  if n % 5 < 1 then s = s.."Buzz" end
  return s ~= "" and s or tostring(n)
end
```

この `function` を `main.lua` に書いたループから呼び出します。この頃は、ドキュメントコメントを頑張って書くことにしていたらしいです。

```lua
require "src/fizzbuzz"

--- 1から100までの整数に対してFizzBuzzを実行し、その結果を標準出力する。
for i = 1, 100 do
  print(fizzbuzz(i))
end
```

※ このコードは2024年1月に書かれた

## Dart

Dartも同様の発想です。この発想で書くのが好きらしいです。

```dart
/// 整数nに対するFizzBuzzを返す。
///
/// nが3の倍数の場合は"Fizz"、
/// 5の倍数の場合は"Buzz"、
/// 3の倍数かつ5の倍数の場合は"FizzBuzz"、
/// それ以外の場合はnの文字列表現を返す。
/// @param n FizzBuzzの元となる整数値
/// @returns nのFizzBuzz
String fizzBuzz(int n) {
   var s = '';
  if (n % 3 == 0) s += 'Fizz';
  if (n % 5 == 0) s += 'Buzz';
  return s.isNotEmpty ? s : '$n';
}
```

DartはStream処理を書きにくいところがあり、どちらかというと素朴なfor文がDartらしい書き方になる気がします。ここでは頑張ってStreamを作り、`fizzBuzz` を呼び出してます。`map` と `listen` は素直ではないが、健気にStream処理をしてます。

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

※ このコードは2025年6月に書かれた

## TypeScript

Bunは、JavaScriptにトランスパイルを経ず、TypeScriptを直接実行できることを思い出し、TypeScriptに興味を持った時のことです。途中でClaude Codeを使ってFizz Buzzを1行で書くことに興味が移った記憶があり、変わった書き方をしてます。

```typescript
for (let i = 1; i <= 100; i++) console.log(((i % 3) ? "" : "Fizz") + ((i % 5) ? "" : "Buzz") || i)
```

GitHub ActionsでもBunを使って直接TypeScriptを実行してます。

```yaml
name: FizzBuzz
on:
  pull_request:

jobs:
  fizzbuzz:
    runs-on: ubuntu-24.04
    steps:
      - uses: actions/checkout@de0fac2e4500dabe0009e67214ff5f5447ce83dd
      - uses: oven-sh/setup-bun@0c5077e51419868618aeaa5fe8019c62421857d6
      - run: bun fizzbuzz.ts
```

※ このコードは2026年5月に書かれた

## Elixir

[Elixirへの誘い](https://okuzawats.com/blog/elixir/)でも触れているように、Elixirに興味を持ってました（ます）。`defguardp` などの技巧を凝らしてパターンマッチで書いてます。

```elixir
defmodule FizzBuzz do
  @moduledoc "run FizzBuzz."

  # fizz/1 true if n is divisible by 3
  defguardp fizz(n) when rem(n, 3) == 0
  # buzz/1 true if n is divisible by 5
  defguardp buzz(n) when rem(n, 5) == 0

  @doc "returns FizzBuzz, Fizz, Buzz or the number in string."
  def run(n) when fizz(n) and buzz(n), do: "FizzBuzz"
  def run(n) when fizz(n), do: "Fizz"
  def run(n) when buzz(n), do: "Buzz"
  def run(n), do: "#{n}"
end
```

ユニットテストも書いてます。

```elixir
Code.require_file("fizzbuzz.exs", __DIR__)

ExUnit.start()

defmodule FizzBuzzTest do
   use ExUnit.Case

   test "returns FizzBuzz when multiples of 15" do
     assert FizzBuzz.run(15) == "FizzBuzz"
     assert FizzBuzz.run(30) == "FizzBuzz"
   end

   test "returns Fizz when multiples of 3" do
    assert FizzBuzz.run(3) == "Fizz"
    assert FizzBuzz.run(6) == "Fizz"
  end

  test "returns Buzz when multiples of 5" do
    assert FizzBuzz.run(5) == "Buzz"
    assert FizzBuzz.run(10) == "Buzz"
  end

  test "returns the number in string when not multiples of 3 nor 5" do
    assert FizzBuzz.run(1) == "1"
    assert FizzBuzz.run(2) == "2"
  end
end
```

Elixirのパイプ演算子はやはり面白くて、繰り返し処理はここまで書いたどの言語よりもbeautifulです。

```elixir
Code.require_file("fizzbuzz.exs", __DIR__)

1..100 |> Enum.each(fn n -> n |> FizzBuzz.run() |> IO.puts() end)
```

なお、[mise](https://mise.jdx.dev/)使ってバージョン管理できます。以下は `mise.toml` の例。

```toml
[tools]
elixir = "1.20.0"
```

※ このコードは2026年6月に書かれた

こうしてふりかえると、毎年少なくとも1つはプログラミング言語を学んでおり、「達人プログラマー」の「1年に1つの言語」を愚直に守っているようです。まだまだ学ぶべきプログラミング言語は存在するので、学習を継続したいと思います。

> 毎年、少なくとも1つの言語を学習する。言語が異なれば、同じ問題に対して異なる解決方法が採用される。いくつかの異なったアプローチを学習することで思考に幅が生まれる。(Thomas & Hunt, 2020)

## Reference

- [Wikipedia:Fizz_Buzz](https://ja.wikipedia.org/wiki/Fizz_Buzz)（最終アクセス日：2026/09/26）
- David Thomas, Andrew Hunt, 村上雅章(訳), (2020), 「達人プログラマー（第2版）」, オーム社
