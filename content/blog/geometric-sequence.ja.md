---
title: "数列、等比数列、等比級数、無限等比級数"
date: 2024-05-26T05:45:00+09:00
description: "数列、等比数列、等比級数、無限等比級数の概要について述べる。"
categories: ["Mathematics"]
draft: false
katex: true
---

本記事では、数列、等比数列、等比級数、無限等比級数の概要について記述します。

### 数列

数を列に並べたものを一般に**数列**と呼びます。例えば、[フィボナッチ数](https://ja.wikipedia.org/wiki/%E3%83%95%E3%82%A3%E3%83%9C%E3%83%8A%E3%83%83%E3%83%81%E6%95%B0)として知られる数の数列は以下のようになります。

$$
0, 1, 1, 2, 3, 5, 8, 13, {\cdots}
$$

数列の最初の数を**初項**と呼びます。上記フィボナッチ数列の例では `0` が初項です。

### 等比数列

初項を `a` として、次の項が前の項の定数倍となっている数列、すなわち `n` 番目の項が `n-1` 番目の項の `r` 倍となっている数列を一般に**等比数列**と呼びます。ここで `r` を**公比**と呼びます。 例えば、初項を `1` 、公比を `2` とした数列は以下のようになります。

$$
1, 2, 4, 8, 16, 32, 64, 128, {\cdots}
$$

等比数列の一般項、すなわち第 `n` 番目の項は、以下のように表すことができます。ここで `a` は初項、 `r` は公比です。

$$
\begin{equation}
\begin{split}
a_n = {a}{\cdot}{r^n}
\end{split}
\end{equation}
$$

### 等比級数

第 `n` 項までの等比数列の和（**等比級数**）は以下のように表すことができます。

$$
\begin{equation}
\begin{split}
S_n &= a_0 + a_1 + {\cdots} + a_n
\end{split}
\end{equation}
$$

ここで、両辺に `1 - r` を乗じます。 `r = 1` の場合の等比級数は自明なので、ここでは `r ≠ 1` と仮定します。

$$
\begin{equation}
\begin{split}
S_n{\cdot}(1 - r) = a_0{\cdot}(1 - r) + a_1{\cdot}(1 - r) + {\cdots} + a_n{\cdot}(1 - r)
\end{split}
\end{equation}
$$

ここで、等比数列の各項には以下の関係が成り立ちます。

$$
\begin{equation}
\begin{split}
a_{k+1} = a_k {\cdot} r
\end{split}
\end{equation}
$$

(4)式を用いると、(3)式の右辺は以下のようになります。

$$
\begin{equation}
\begin{split}
S_n{\cdot}(1 - r) &= (a_0 - a_1) + (a_1 - a_2) + {\cdots} + (a_n - a_{n+1})\\\
  &= a_0 - a_{n+1}
\end{split}
\end{equation}
$$

最終的に、等比級数は式(1)及び式(5)を用いて以下のように書くことができます。

$$
\begin{equation}
\begin{split}
S_n = { \frac {a {\cdot} (1 - r^{n+1})} {1 - r} }
\end{split}
\end{equation}
$$

### 無限等比級数

式(6)から、 `n` が無限大の時の等比級数（**無限級数**）は式(7)となる。特に `-1 < r < 1` の時、無限級数は収束します。

$$
\begin{equation}
\begin{split}
\lim_{n \to \infty} S_n = \begin{cases} \infty(r \ge 1)\\\ {\frac a {1-r}}(|r| \lt 1) \end{cases}
\end{split}
\end{equation}
$$

### References

1. [フィボナッチ数 - Wikipedia](https://ja.wikipedia.org/wiki/%E3%83%95%E3%82%A3%E3%83%9C%E3%83%8A%E3%83%83%E3%83%81%E6%95%B0)（最終アクセス日：2024年5月26日）
2. [等比数列 - Wikipedia](https://ja.wikipedia.org/wiki/%E7%AD%89%E6%AF%94%E6%95%B0%E5%88%97)（最終アクセス日：2024年5月26日）
