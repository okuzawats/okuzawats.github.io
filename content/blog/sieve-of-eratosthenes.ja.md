---
title: "Luaでエラトステネスの篩を実装してみた"
date: 2024-02-04T05:23:00+09:00
description: "プログラミング言語Luaを使ってエラトステネスの篩（ふるい）を実装してみました。"
categories: ["Lua", "Algorithm"]
draft: false
---

最近興味を持っているプログラミング言語「[Lua](https://ja.wikipedia.org/wiki/Lua)」[^1]を使って、エラトステネスの篩（ふるい）を実装してみました。「エラトステネス」というのは人の名前で、紀元前に生きていた方です。地球の大きさを始めて測定した人物だそうです。エラトステネスの篩とは、そのエラトステネスさんが考えた、ある整数以下の素数を求めるためのアルゴリズムです。

> エラトステネスの篩 (エラトステネスのふるい、英: Sieve of Eratosthenes) は、指定された整数以下の全ての素数を発見するための単純なアルゴリズムである。古代ギリシアの科学者、エラトステネスが考案したとされるため、この名がついている。（[エラトステネスの篩 - Wikipedia](https://ja.wikipedia.org/wiki/%E3%82%A8%E3%83%A9%E3%83%88%E3%82%B9%E3%83%86%E3%83%8D%E3%82%B9%E3%81%AE%E7%AF%A9)より引用）

ちなみに、現在では「アトキンの篩」と呼ばれる、より高速なアルゴリズムも考えられているそうです。

## 実装

Luaははじめて書くので、まずは関数のシグネチャを定義するところから始めます。整数値 `n` を引数として受け取り、整数値の配列を返します。Luaのコメントの書き方ってこうなんだ〜と思いながら書きました。なお、ローカル変数には `local` 修飾子を付けないとグローバル変数として扱われてしまいます。

```lua
--- n以下の整数のうち、素数のみを格納した配列を返す。
-- @param n 対象となる整数。正の値である必要がある。
-- @return n以下の整数のうち、素数のみを格納した配列。
function sieve(n)
  local prime = {} -- 素数と判定された整数値を格納する配列
  return prime
end
```

`assert` を用いて引数の検証を行います。この関数が意味を持つのは引数が自然数の時のみなので、それを検証します。Luaの `assert` の文法は簡潔でいいですね。

```lua
assert(n > 0, "n must be positive.")
```

2以上 `n` 以下の整数値に対して、それらが自然数か否かを `true` / `false` で配列に持ちます。この配列が、篩のメタファーとして働きます。Luaの配列は、テーブルの特殊型です。テーブルのキーを対象となる整数値に、そのバリューを真偽値にしています。配列をいい感じにデフォルト値で初期化する方法があると嬉しかったのですが、自分が調べた限りLuaにはそういった機能はなさそうでした。そのため、ループを回して配列を初期化しています。

```lua
local sieve = {} -- 判定対象となる整数値を格納する配列（「篩（ふるい）」と呼ぶ）

-- 篩をtrueで初期化する。1はスキップして良い。
for i = 2, n do
  sieve[i] = true
end
```

2から `n` の平方根までループを回して、それぞれの整数値が素数か否かに応じて整数値を配列から篩落としていきます。つまり、各整数値の倍数は合成数（素数でない数）であるため、配列に格納されている真偽値を `false` に設定します。

```lua
-- nの平方根（より小さい最大の整数）まで、篩操作を行う。この操作はnの平方根までで十分。
for i = 2, math.sqrt(n) do
  -- 対象の整数がtrue（素数）の場合は、その倍数をfalse（合成数）とする。
  -- iの2倍の整数値からnまで、iの倍数について篩落とす。
  if sieve[i] then
    for ii = i * 2, n, i do
    sieve[ii] = false
    end
  end
end
```

ループを `n` まで回して、篩に残った整数値は素数です。これらをテーブルに格納して返り値として返します。Luaのforループでは、 `#` を用いて配列に対するループを回せる点が面白かったです。`#` は、配列や文字列の要素数を返します。

```lua
-- 篩に残った整数は素数であるため、配列に格納する。
for i = 2, #sieve do
  if sieve[i] then
    table.insert(prime, i)
  end
end
```

## 完成

以上をまとめると、以下のようなコードとなります。

```lua
--- n以下の整数のうち、素数のみを格納した配列を返す。
-- @param n 対象となる整数。正の値である必要がある。
-- @return n以下の整数のうち、素数のみを格納した配列。
function sieve(n)
  assert(n > 0, "n must be positive.")

  local sieve = {} -- 判定対象となる整数値を格納する配列（「篩（ふるい）」と呼ぶ）
  local prime = {} -- 素数と判定された整数値を格納する配列

  -- n = 2、n = 3の時に正しく動いていないことに気付いたので、そのバグを回避するためのコード。
  if n == 2 then return { 2 } end
  if n == 3 then return { 2, 3 } end

  -- 篩をtrueで初期化する。1はスキップして良い。
  for i = 2, n do
    sieve[i] = true
  end

  -- nの平方根（より小さい最大の整数）まで、篩操作を行う。この操作はnの平方根までで十分。
  for i = 2, math.sqrt(n) do
    -- 対象の整数がtrue（素数）の場合は、その倍数をfalse（合成数）とする。
    -- iの2倍の整数値からnまで、iの倍数について篩落とす。
    if sieve[i] then
      for ii = i * 2, n, i do
        sieve[ii] = false
      end
    end
  end

  -- 篩に残った整数は素数であるため、配列に格納する。
  for i = 2, #sieve do
    if sieve[i] then
      table.insert(prime, i)
    end
  end

  return prime
end
```

ソースコードはGitHubにも置いてあります。もしコードを参考にされる方がいたらスターをお願いします⭐️

- [okuzawats/lua-sieve-of-eratosthenes: Luaでエラトステネスの篩](https://github.com/okuzawats/lua-sieve-of-eratosthenes)

## Reference

1. [Lua - Wikipedia](https://ja.wikipedia.org/wiki/Lua)（最終アクセス日：2024年2月4日）
1. [エラトステネスの篩 - Wikipedia](https://ja.wikipedia.org/wiki/%E3%82%A8%E3%83%A9%E3%83%88%E3%82%B9%E3%83%86%E3%83%8D%E3%82%B9%E3%81%AE%E7%AF%A9)（最終アクセス日：2024年2月4日）
1. [Lua 配列(Array)](https://ooo.iiyudana.net/htm/lua_chp5frame.htm)（最終アクセス日：2024年2月4日）
1. [okuzawats/lua-sieve-of-eratosthenes: Luaでエラトステネスの篩](https://github.com/okuzawats/lua-sieve-of-eratosthenes)（最終アクセス日：2024年2月4日）

[^1]: Luaを使うことで得られるメリットは特に期待してないです。自分が仕事でLuaを使う機会は無いと思います。
