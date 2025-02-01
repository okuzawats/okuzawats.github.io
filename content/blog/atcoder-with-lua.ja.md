---
title: "LuaでAtCoderの過去問精選10問を味わう"
date: 2024-02-17T06:20:00+09:00
description: "プログラミング言語Luaを使ってAtCoderの過去問精選10問を味わってみました。"
categories: ["Lua"]
draft: false
---

プログラミング言語Luaを使って、AtCoderの過去問精選10問を味わってみました。こういった問題を解く時、Luaは美しく書けるな〜という感想です。

[AtCoder に登録したら次にやること ～ これだけ解けば十分闘える！過去問精選 10 問 ～ #AtCoder - Qiita](https://qiita.com/drken/items/fd4e5e3630d0f5859067)

## 第1問 ABC 086 A - Product

問題の内容は、AtCoderの出題ページ（[A - Product](https://atcoder.jp/contests/abc086/tasks/abc086_a)）を参照してください。以下9問についても同様に出題ページへのリンクのみ提示します。

ABCのA問題という感じの難易度の問題ですが、Luaの文法を学ぶには丁度良いです。最終的な自分の回答は以下のコードです。

```lua
a, b = io.read("n", "n")
print(a * b % 2 < 1 and "Even" or "Odd")
```

まず、1行目のコードでは、標準入力から数値型の値を2個読み取り、変数 `a` と `b` に分割代入しています。`io.read` に文字列 `n` を渡すわけですが、可変長の引数を取ることが可能で、標準入力から複数の値を読み取る時に重宝します。自分の場合、プロダクト開発では分割代入を使うことはあまりありませんが、こういった問題を解く時は簡潔に記述することができて嬉しいです。

```lua
a, b = io.read("n", "n")
```

2行目のコードでは、`and` と `or` を用いた、Luaでの頻出イディオムを使っています。`and` の左辺値に真偽値を取り、真偽値が真である場合は `and` の右辺値が評価されます。真偽値が偽である場合は `and` の右辺値は評価されず、`or` の右辺値が評価されます。一部の言語における三項演算子のような使い方をするイディオムとなっています。

```lua
print(a * b % 2 < 1 and "Even" or "Odd")
```

## 第2問 ABC 081 A - Placing Marbles

[A - Placing Marbles](https://atcoder.jp/contests/abc081/tasks/abc081_a)

こちらも難易度は低いですが、Luaのコードとしてはなかなか味わい深いです。

`io.read` を用いて、今度は1文字ずつ標準入力から読み取り、分割代入しています。驚くべきことに、読み込んだ文字をそのまま加算することで数値型の加算として計算が実行されます。

```lua
a, b, c = io.read(1, 1, 1)
print(a + b + c)
```

自分にとっては驚くべきコードでしたが、結果として非常に簡潔にコードを書くことができ、美しいです。

## 第3問 ABC 081 B - Shift only

[B - Shift only](https://atcoder.jp/contests/abc081/tasks/abc081_b)

精選問題も3問目に入り、ABC B問題の難易度になります。1問目、2問目と比較すると本格的な感じになってきます。本問では、配列やループの処理が必要になります。最終的な自分の回答は以下のコードです。

```lua
a, n = {}, io.read("n")
for i = 1, n do
  a[i] = io.read("n")
end

c = 0
while true do
  f = false
  for i = 1, n do
    if a[i] % 2 > 0 then
      f = true
      break
    end
  end
  if f then
    break
  end
  for j = 1, n do
    a[j] = a[j] // 2
  end
  c = c + 1
end
print(c)
```

Luaでは、配列のインデックスは1からです。これは便利なことが多いですが、慣れないと戸惑うかもしれません。配列の初期化は以下のように行なっています。

```lua
for i = 1, n do
  a[i] = io.read("n")
end
```

配列の更新処理も同様です。

```lua
for j = 1, n do
  a[j] = a[j] // 2
end
```

## 第4問 ABC 087 B - Coins

[B - Coins](https://atcoder.jp/contests/abc087/tasks/abc087_b)

3重ループです。1問目から3問目までの知識で解けます。

```lua
a, b, c, x = io.read("n", "n", "n", "n")

count = 0
for i = 0, a do
  for j = 0, b do
    for k = 0, c do
      sum = 500 * i + 100 * j + 50 * k
      if sum == x then
        count = count + 1
      end
    end
  end
end
print(count)
```

## 第5問 ABC 083 B - Some Sums

[B - Some Sums](https://atcoder.jp/contests/abc083/tasks/abc083_b)

こちらの問題も基本的にはここまでの知識で解けます。整数の各位の和を求める処理は算術的に求める方が美しいと感じますが...楽をして文字列として処理しています。前述のように、Luaでは文字列が整数であれば算術演算を適用できます。

```lua
n, a, b = io.read("n", "n", "n")

count = 0
for i = 1, n do
  s, sum = tostring(i), 0
  for j = 1, #s do
    sum = sum + s:sub(j, j)
  end
  if b >= sum and sum >= a then
    count = count + i
  end
end
print(count)
```

文字列への変換は `tostring` です。

```lua
s, sum = tostring(i), 0
```

文字列から部分文字列を抽出するには、`string:sub` を使います。以下のコードは `string.sub(s, j, j)` と同じです。

```lua
s:sub(j, j)
```

## 第6問 ABC 088 B - Card Game for Two

[B - Card Game for Two](https://atcoder.jp/contests/abc088/tasks/abc088_b)

ソートです。Luaでは `table.insert` を用いたソートができます。デフォルトでは昇順でソートされるので、今回のように降順でソートしたいような場合は、第2引数にラムダを渡します。

```lua
a, n = {}, io.read("n")
for i = 1, n do
  table.insert(a, io.read("n"))
end

table.sort(a, function(a, b) return a > b end)

count = 0
for i = 1, #a do
  if i % 2 > 0 then
    count = count + a[i]
  else
    count = count - a[i]
  end
end
print(count)
```

## 第7問 ABC 085 B - Kagami Mochi

[B - Kagami Mochi](https://atcoder.jp/contests/abc085/tasks/abc085_b)

参考文献1においてバケット法として紹介されている手法を使います。本問では、配列の添え字として直径の整数値を用いて、その値が真か偽かを判定することにより問題を解きます。Luaの配列（というかテーブル）はこのような用途を扱う場合にとても便利で、スマートに解くことができました。

```lua
k, c, n = {}, 0, io.read("n")
for i = 1, n do
  d = io.read("n")
  if not k[d] then
    k[d], c = 0, c + 1
  end
end
print(c)
```

配列 `k` に対して直径 `d` を添え字にしてアクセスした場合、値が格納されていなければ `nil` が返ります。これを利用して、`k[d]` に対する真偽を判定します。構文としては、`k[d] == nil` または `not k[d]` となります。

```lua
if not k[d] then
```

`k[d]` の真偽を判定するためにLuaで用意されている真偽値（true/false）を使うこともできますが、本問ではnil/0で判定すれば十分であることに気付きました。そのため、配列にtrueを入れる代わりに0を入れて文字数の節約を試みています。こういった、ちょっとした気付きが得られた時は嬉しくなります。

```lua
k[d], c = 0, c + 1
```

## 第8問 ABC 085 C - Otoshidama

[C - Otoshidama](https://atcoder.jp/contests/abc085/tasks/abc085_c)

ついにABCにおけるC問題です。が、本問については左程難しくはありません。3重のループを2重のループに書き直すことができることに気付けば解くことができます。

```lua
n, y = io.read("n", "n")
for i = 0, n do
  for j = 0, n - i do
    k = n - i - j
    if 10000 * i + 5000 * j + 1000 * k == y then
      print(i.." "..j.." "..k)
      os.exit()
    end
  end
end
print("-1 -1 -1")
```

Luaにおける文字列の連結には、`..` という演算子を使うことができます。`"abc".."def"` は `"abcdef"` と評価されます。面白いですね。

```lua
print(i.." "..j.." "..k)
```

条件を満たす整数の組が見つかった時に、以下のコードでプログラムを強制終了しています。個人的にはあまり美しくないな〜とは思いますが、今回のような問題を解くためには `os.exit()` を使うのがスマートだと思います。

```lua
os.exit()
```

## 第9問 ABC 049 C - Daydream

[C - 白昼夢](https://atcoder.jp/contests/abc049/tasks/arc065_a)

貪欲法です。文字列を逆順にする、という解説を読むまではうまく解けませんでした。正規表現を用いて文字列を置き換えていく、という解答例もありました。自分としては、Luaを味わう、というような解答はまだできておりません...。

```lua
s = io.read()
s = s:reverse()

w = { "dream", "dreamer", "erase", "eraser" }
for i = 1, #w do w[i] = w[i]:reverse() end

f = true
while true do
  f2 = false
  for j = 1, #w do
    w2 = w[j]
    if s:sub(1, #w2) == w2 then
      f2 = true
      s = s:sub(#w2 + 1, #s)
      break
    end
  end
  if s == "" then
    break
  end
  if not f2 then
    f = false
    break
  end
end

print(f and "YES" or "NO")
```

## 第10問 ABC 086 C - Traveling

[C - Traveling](https://atcoder.jp/contests/abc086/tasks/arc089_a)

ここまでの知識を総動員してバリバリ書く感じで解答しました。

```lua
x, y, z, f, n = 0, 0, 0, true, io.read("n")
for i = 1, n do
  t, a, b = io.read("n", "n", "n")
  d = math.abs(a - x) + math.abs(b - y)
  if t - z < d or (z - t - d) % 2 > 0 then
    f = false
    break
  end
  x, y, z = a, b, t
end
print(f and "Yes" or "No")
```

## まとめ

楽しかったです。

## Reference

1. [AtCoder に登録したら次にやること ～ これだけ解けば十分闘える！過去問精選 10 問 ～ #AtCoder - Qiita](https://qiita.com/drken/items/fd4e5e3630d0f5859067#5-%E9%81%8E%E5%8E%BB%E5%95%8F%E7%B2%BE%E9%81%B8-10-%E5%95%8F)（最終アクセス日：2024年2月17日）
1. [A - Product](https://atcoder.jp/contests/abc086/tasks/abc086_a)（最終アクセス日：2024年2月17日）
1. [A - Placing Marbles](https://atcoder.jp/contests/abc081/tasks/abc081_a)（最終アクセス日：2024年2月17日）
1. [B - Shift only](https://atcoder.jp/contests/abc081/tasks/abc081_b)（最終アクセス日：2024年2月17日）
1. [B - Coins](https://atcoder.jp/contests/abc087/tasks/abc087_b)（最終アクセス日：2024年2月17日）
1. [B - Some Sums](https://atcoder.jp/contests/abc083/tasks/abc083_b)（最終アクセス日：2024年2月17日）
1. [B - Card Game for Two](https://atcoder.jp/contests/abc088/tasks/abc088_b)（最終アクセス日：2024年2月17日）
1. [B - Kagami Mochi](https://atcoder.jp/contests/abc085/tasks/abc085_b)（最終アクセス日：2024年2月17日）
1. [C - Otoshidama](https://atcoder.jp/contests/abc085/tasks/abc085_c)（最終アクセス日：2024年2月17日）
1. [C - 白昼夢](https://atcoder.jp/contests/abc049/tasks/arc065_a)（最終アクセス日：2024年2月17日）
1. [C - Traveling](https://atcoder.jp/contests/abc086/tasks/arc089_a)（最終アクセス日：2024年2月17日）
