---
title: "Elixirへの誘い"
date: 2026-06-07T21:27:00+09:00
description: "Elixirへの誘い（いざない）。"
categories: ["Elixir"]
draft: false
---

**Elixir**は、動的型付け・関数型の特徴を持つプログラミング言語です。Erlangの仮想環境であるBEAM上で動作します。BEAM上で動作することにより、並列処理や分散処理に強みがあります。つい先日（2026年6月3日）発表されたv1.20では「段階的型付け」と呼ばれる型システムが導入され、型付けが強化されました。

Elixirはメインストリームにあるとは言えない言語ですが、上記のとおり非常に魅力的な特徴を持っています。自分は何年か前に軽く学んだきりでしたが、再度入門していきます。

早速、Elixirのインストールからしていきましょう（macOS Tahoe 26.3.1）。今回はElixirのバージョン管理に[mise](https://mise.jdx.dev/)を使います。`mise.toml` に以下を書きます。

```toml
[tools]
elixir = "1.20.0"
```

```bash
mise install
```

[FizzBuzz](https://ja.wikipedia.org/wiki/Fizz_Buzz)を書きます。先に全体を俯瞰し、その後で詳細を語ります。いろいろな書き方をできると思いますが、調べながら自分にとって最も学習的な書き方を目指しました。

```elixir
defmodule FizzBuzz do
  @moduledoc "run FizzBuzz."

  # `fizz/1` true if `n` is divisible by 3
  defguardp fizz(n) when rem(n, 3) == 0
  # `buzz/1` true if `n` is divisible by 5
  defguardp buzz(n) when rem(n, 5) == 0

  @doc "returns FizzBuzz, Fizz, Buzz or the number in string."
  def run(n) when fizz(n) and buzz(n), do: "FizzBuzz"
  def run(n) when fizz(n), do: "Fizz"
  def run(n) when buzz(n), do: "Buzz"
  def run(n), do: "#{n}"
end
```

まず `defmodule` ですが、モジュールを定義するマクロです。Elixirにおけるモジュールとは、関数に名前空間を与える役割を持っているようです。Javaなどの言語におけるclassはインスタンスを作ることができますが、Elixirにおけるモジュールはインスタンスを作ることはできません。

次に `defguardp` についてですが、ガード句を定義するマクロです。ガード句についてはすぐに後述しますが、関数のパターンマッチに使う文法です。

ガード句は、 `when` キーワードと真偽値を返す式の組み合わせにより、関数の引数に対する条件を追加する仕組みです。例えば、 `def f(n) when n > 0, do: "something"` という関数定義においては `when n > 0` がガード句、 `n > 0` が真偽値を返す式です。このガード句により、引数 `n` に対して `n > 0` が真である場合に `"something"` が評価されて返り値となります。

Elixirでは関数を多重定義し、ガード句と組み合わせてパターンマッチすることができます。この部分です。美しいですね。

```elixir
   def run(n) when fizz(n) and buzz(n), do: "FizzBuzz"
   def run(n) when fizz(n), do: "Fizz"
   def run(n) when buzz(n), do: "Buzz"
   def run(n), do: "#{n}"
```

`defguardp` マクロを使わない場合は、以下のような感じで書けます。

```elixir
   def run(n) when rem(n, 3) == 0 and rem(n, 5) == 0, do: "FizzBuzz"
   def run(n) when rem(n, 3) == 0, do: "Fizz"
   def run(n) when rem(n, 5) == 0, do: "Buzz"
   def run(n), do: "#{n}"
```

定義した `FizzBuzz.run/1` 関数を呼び出す側のコードも書きます（ `1` の部分はElixir特有の概念「アリティ（Arity）」と呼ばれるもので、引数が1つの関数であることを表しています）。こちらも自分にとって学習的な書き方としています。

```elixir
Code.require_file("fizzbuzz.exs", __DIR__)

1..100 |> Enum.each(fn n -> n |> FizzBuzz.run() |> IO.puts() end)
```

まず、 `1..100` はRange型を返します（他の言語にも良くあると思うので説明は不要ですね）。

次の `|>` はパイプ演算子（またはパイプライン演算子）と呼ばれるもので、Elixirを特徴付ける構文のひとつであると思います。一言で言えば、左辺を右辺の第1引数として渡す演算子です。ここでは、左辺はRange型の `1..100` 、右辺はEnum.each/2です。Enum.eachは一見すると関数のみを受け取っているように見えるんですが、パイプ演算子により第1引数として `1..100` を受け取っています。

つまり、パイプ演算子を用いずに以下のように書いても処理は同じです。しかし、パイプ演算子を用いることで左から右に処理の流れを読んでいくことができ、非常に美しいです。

```elixir
Enum.each(1..100, fn n -> n |> FizzBuzz.run() |> IO.puts() end)
```

Enum.each/2の第2引数は以下の関数オブジェクトです。これは無名関数であり、関数本体（ `n |> FizzBuzz.run() |> IO.puts()` の部分）ではパイプ演算子を用いて返り値をチェーンしています。

```elixir
fn n -> n |> FizzBuzz.run() |> IO.puts() end
```

パイプ演算子を用いずに書くと `fn n -> IO.puts(FizzBuzz.run(n)) end` と書けますが、ここでもパイプ演算子を用いることで左から右に処理の流れを読んでいくことができ、美しいです。

また、Elixirの無名関数には省略記法があり、例えば `fn() -> end` は `&()` と書くことができます。短く書くことができるのは良いんですが、初学者の自分には少し読みにくかったので、使いませんでした。慣れたら読みやすい / 美しいのかもしれません。

以上、動的型付け・関数型の特徴を持ち、Erlangの仮想環境（BEAM）上で動作するため並列処理や分散処理に強みがあり、文法も非常に美しいプログラミング言語**Elixir**に入門しました。しばらくはElixirで遊んでいこうと思います。

## Reference

1. [Elixir (プログラミング言語)](https://ja.wikipedia.org/wiki/Elixir_(%E3%83%97%E3%83%AD%E3%82%B0%E3%83%A9%E3%83%9F%E3%83%B3%E3%82%B0%E8%A8%80%E8%AA%9E)) （最終アクセス日：2026/06/07）
2. [Elixir v1.20 released: now a gradually typed language](https://elixir-lang.org/blog/2026/06/03/elixir-v1-20-0-released/) （最終アクセス日：2026/06/07）
