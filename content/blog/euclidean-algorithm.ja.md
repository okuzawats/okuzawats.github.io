---
title: "[Kotlin] ユークリッドの互助法を用いて最小公約数・最小公倍数の計算を実装する"
date: 2022-03-18T22:20:00+09:00
description: ユークリッドの互助法を用いて最小公約数・最小公倍数の計算を実装します。"
categories: ["Kotlin", "Algorithm"]
draft: false
---

> 2 つの自然数 a, b (a ≧ b) について、a の b による剰余を r とすると、 a と b との最大公約数は b と r との最大公約数に等しいという性質が成り立つ。この性質を利用して、 b を r で割った剰余、 除数 r をその剰余で割った剰余、と剰余を求める計算を逐次繰り返すと、剰余が 0 になった時の除数が a と b との最大公約数となる。（「ユークリッドの互除法 - Wikipedia」より引用）

上記のユークリッドの互助法をKotlinのコードに当てはめると、最小公約数（Greatest Common Divisor）を求める処理は以下のように再帰を用いて実装できます。末尾再帰最適化を適用できますので、 `tailrec` キーワードを付けています。

```kotlin
// 最小公約数
tailrec fun gcd(a: Long, b: Long): Long =  
  if (b == 0L) a  
  else gcd(b, a % b)  
```

最小公倍数（Least Common Multiple）を求める処理については、2つの数の積を最小公約数で除算すればよいです。上記の `gcd` 関数を用いて以下のように実装できます。 `(a * b) / gcd(a, b)` としていないのは、 `()` を減らすための細かいテクニックです（コードゴルフ）。

```kotlin
// 最小公倍数
fun lcm(a: Long, b: Long): Long = a / gcd(a, b) * b
```

ここまで頑張って実装してきたのですが、JavaのBigIntegerに `gcd` が存在しますので、こちらを使うと実務上は楽ができるかもしれません。以下のコードは、AtCoder Beginner Contest 148のC問題に対する、BigIntegerを用いた自分の解答です。

```kotlin
import java.math.*

fun main() {
  val (a, b) = readLine()!!.split(' ').map(::BigInteger)
  println(a / a.gcd(b) * b)
}
```

## Reference
1. [ユークリッドの互除法 - Wikipedia](https://ja.wikipedia.org/wiki/%E3%83%A6%E3%83%BC%E3%82%AF%E3%83%AA%E3%83%83%E3%83%89%E3%81%AE%E4%BA%92%E9%99%A4%E6%B3%95)（最終アクセス日：2022年3月18日）
2. [C - Snack](https://atcoder.jp/contests/abc148/tasks/abc148_c)（最終アクセス日：2022年3月18日）
3. [提出 #29608584 - AtCoder Beginner Contest 148](https://atcoder.jp/contests/abc148/submissions/29608584)
