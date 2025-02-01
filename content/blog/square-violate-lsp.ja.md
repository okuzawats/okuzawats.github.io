---
title: "何故、Squareクラスはリスコフの置換原則（LSP）に違反するのか"
date: 2023-04-30T15:25:53+09:00
description: "Squareクラスがリスコフの置換原則（LSP）に違反している理由について考えます。"
categories: ["Design"]
draft: false
---

**リスコフの置換原則（The Liskov Substitution Principle: LSP）** に違反する例として、Rectangleクラスを継承したSquareクラスの例を複数の書籍で目にしました（Robert (2004)、Michael (2009)、Robert (2018)）。何故、Squareクラスはリスコフの置換原則に違反するのでしょうか。

リスコフの置換原則とは、以下のように要約されます。

> 派生型はその基本型と置換可能でなければならない。（Robert, (2004), p.144）

Rectangleクラスは長方形を表すクラスで、幅（width）と高さ（height）を持ち、幅と高さから面積（area）を計算することができます。

```java
public class Rectangle {
    public Rectangle(int width, int height) { /* 略 */ }

    private int width;
    private int height;

    public void setWidth(int width) { /* 略 */ }
    public void setHeight(int height) { /* 略 */ }
    public int getArea() { /* 略 */ }
}
```

Squareクラスは正方形を表すクラスで、リスコフの置換原則に違反する例では、Rectangleを継承します。正方形は長方形の一種、というわけです。正方形の幅と高さは定義により常に等しいです。

```java
public Square(int size) {
    /* 略 */
}
```

Squareを親クラスのRectangle型としてインスタンス化してみます。

```java
Rectangle rectangle = new Square(3);
```

このインスタンスに対して `setWidth` と `setHeight` を呼んだ時、面積はどうなるでしょうか？

長方形の面積は幅と高さの積で計算されます。一方、正方形は定義により幅と高さが常に等しいので、`setHieght(4)` を呼び出した時に、その幅も `3` から `4` に変更されることでしょう。この場合、面積は（Rectangle型なので）3と4の積の12が期待されるにも関わらず、実際には16となります。

```java
rectangle.setWidth(3);
rectangle.setHeight(4); // 正方形の幅と高さは常に等しいので、同時にwidthも4に変更される。
System.out.println(rectangle.getArea()); // => 16
```

そうすると、RectangleがSquareかそうでないかによって振る舞いが異なってしまい、「派生型はその基本型と置換可能でなければならない」（Robert, (2004), p.144）というリスコフの置換原則に違反しているということになります。

```java
if (rectangle instanceof Square) {
    assert rectangle.getArea() == 16;
} else {
    assert rectangle.getArea() == 12;
}
```

書籍「Clean Architecture」では、この問題を「悪名高い」正方形・長方形問題と呼んでいます（Robert, (2018), p.94）。

## 参考文献

1. Robert C.Martin, (2004), アジャイルソフトウェア開発の奥義 オブジェクト指向開発の神髄と匠の技 第2版, SBクリエイティブ
2. Michael C.Feathers, (2009), レガシーコード改善ガイド, 翔泳社
3. Robert C.Martin, (2018), Clean Architecture 達人に学ぶソフトウェアの構造と設計, アスキードワンゴ
