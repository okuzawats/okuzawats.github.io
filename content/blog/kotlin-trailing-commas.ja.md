---
title: "[Kotlin] 末尾カンマ"
date: 2022-05-31T22:22:00+09:00
description: "Kotlinの末尾カンマについての記事です。"
categories: ["Kotlin"]
draft: false
---

Kotlinの末尾カンマは、Kotlin 1.4において導入されました。引数リストの最後の要素の後にカンマを付けられる文法です。以下の `val c: Int,` の `,` の部分を指しています。Kotlin 1.4から、この `,` の部分を追加することが可能になりました（なくても良い）。

```kotlin
data class AwesomeDataClass(
  val a: Int,
  val b: Int,
  val c: Int,
)
```

この末尾カンマが導入されて一番嬉しいのは、ここに新たな引数を追加した場合に、追加した行にのみ差分が生じることです。以下のような差分です。

```
   val c: Int,
+  val d: Int,
 )
```

末尾カンマを使わない場合の差分は以下のようになります。

```
-  val c: Int
+  val c: Int,
+  val d: Int
 )
```

末尾カンマを使った場合は、追加された引数の行にだけ差分が生じています。追加された箇所にだけ着目して差分を見ることが可能です。

引数を横に並べる場合は末尾カンマがあってもあまり嬉しくはありませんが、引数を縦に並べる場合は末尾カンマがあると嬉しいです。自分は引数を縦に並べたいので、（Kotlinのバージョンが1.4以前の場合を除いて）可能な限り、末尾カンマを使うようにしています。また、コードレビューなどでも、IMO[^1]、IMHO[^2]として、末尾カンマの使用を広める活動を地道に行っています。

なお、Kotlin公式のCoding conventionsには、以下の記述があります。変更の差分が見やすくなるというメリットの他に、引数の順序を入れ替えることとコードの生成が容易となることが、末尾カンマのメリットとしてあげられています。

> - It makes version-control diffs cleaner – as all the focus is on the changed value.
> - It makes it easy to add and reorder elements – there is no need to add or delete the comma if you manipulate elements.
> - It simplifies code generation, for example, for object initializers. The last element can also have a comma.

## References

1. What's new in Kotlin 1.4 | Kotlin、Retrieved from https://kotlinlang.org/docs/whatsnew14.html)（最終アクセス日：2022/05/31）
2. Coding conventions | Kotlin、Retrieved from https://kotlinlang.org/docs/coding-conventions.html#trailing-commas)（最終アクセス日：2022/05/31）

[^1]: in my opinion。私の意見では、の意味。
[^2]: in my humble opinion。IMOより弱い表現。
