---
title: "定率成長配当割引モデル"
date: 2024-05-29T06:13:00+09:00
description: "定率成長配当割引モデルの概要について述べる。"
categories: ["Economics"]
draft: false
katex: true
---

**定率成長配当割引モデル**は、将来の配当を元に、企業の現在価値を評価するためのモデルです。ここで、企業は成長し続けるという期待があるため、それに伴い配当も毎年成長率 `g` で増えていくという前提を置きます。また、1年後の配当を現在価値に割り引くため、割引率を `ρ`　とします。

この時、1年後の配当の現在価値（Present Value）は以下のように表すことができます。 `D` は配当（Diviend）です。割引率を用いて配当を現在価値に割り引いています。

$$
\begin{equation}
\begin{split}
PV_1 = { \frac {D_1} {1 + \rho} }
\end{split}
\end{equation}
$$

2年後の配当の現在価値は以下のように表すことができます。配当は成長率 `g` で成長します。

$$
\begin{equation}
\begin{split}
PV_2 = { \frac {{D_1} \cdot {(1 + g)}} {(1 + \rho)^2} }
\end{split}
\end{equation}
$$

(1)式と(2)式から、以下のように書くことができます。

$$
\begin{equation}
\begin{split}
PV_2 = PV_1 \cdot { \frac {1 + g} {1 + \rho} }
\end{split}
\end{equation}
$$

同様に、3年目の配当の現在価値も以下のように書くことができます。

$$
\begin{equation}
\begin{split}
PV_3 = PV_2 \cdot { \frac {1 + g} {1 + \rho} }
\end{split}
\end{equation}
$$

4年目以降も(4)式のような関係式が成立するので、配当の現在価値の列は等比数列となることがわかります。等比数列の一般項、および等比数列の和は、それぞれ以下のように表すことができるのでした（[数列、等比数列、等比級数、無限等比級数](https://okuzawats.com/blog/geometric-sequence/)を参照）。ここで `a` は初項、 `r` は公比です。

$$
\begin{equation}
\begin{split}
a_n = {a}{\cdot}{r^n}
\end{split}
\end{equation}
$$

$$
\begin{equation}
\begin{split}
S_n = { \frac {a {\cdot} (1 - r^{n+1})} {1 - r} }
\end{split}
\end{equation}
$$

企業の現在価値は、将来の配当を現在価値に割り引いた合計です。1年目から `n` 年目までの配当の合計を現在価値とします。配当の現在価値の和は以下のように表すことができます。

$$
\begin{equation}
\begin{split}
\sum_{k = 1}^n{PV_k} &= { \frac {{ \bigl( \frac {D_1} {1 + \rho} \bigr)} {\cdot} \Bigl(1 - { \bigl( \frac {1 + g} {1 + \rho} \bigr) }^{n+1} \Bigr)} {1 - { \bigl( \frac {1 + g} {1 + \rho} \bigr) }} }\\\
   &= { \frac {{D_1} \cdot \Bigl( 1 - \bigl( { \frac {1 + g} {1 + \rho}} \bigr)^{n + 1}  \Bigr)} {\rho - g} }
\end{split}
\end{equation}
$$

一般的に企業はゴーイング・コンサーンを前提に置いているので、 `n → ∞` とします。ここで `g > ρ` である場合を考えると、企業の現在価値は無限大に発散します。無限大の価値を持つ資産は存在しないため、 `g < ρ` という条件を置くことができます。

(7)式、および上記の仮定から、企業の現在価値を `PV` と置くと `PV` は以下のように表すことができます。

$$
\begin{equation}
\begin{split}
PV &= \lim_{n \to \infty}{\sum_{k = 1}^{n}{PV_k}}\\\
   &= \lim_{n \to \infty}{\frac {{D_1} \cdot \Bigl( 1 - \bigl( { \frac {1 + g} {1 + \rho}} \bigr)^{n + 1}  \Bigr)} {\rho - g}}\\\
   &= \frac {D_1} {\rho - g}
\end{split}
\end{equation}
$$

以上から、企業の現在価値は、1年後の配当、割引率、および配当の成長率という3つの数字を用いて表すことができるということがわかりました。

### References

1. 砂川伸幸, (2017), コーポレートファイナンス入門 第2版, 日本経済新聞出版
2. 手嶋宣之, (2011), 基本から本格的に学ぶ人のためのファイナンス入門, ダイヤモンド社
3. [数列、等比数列、等比級数、無限等比級数](https://okuzawats.com/blog/geometric-sequence/)（最終アクセス日：2024年5月29日）
