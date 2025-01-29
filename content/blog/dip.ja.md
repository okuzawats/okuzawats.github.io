---
title: "依存関係逆転の原則"
date: 2021-12-04T00:00:00+09:00
description: "フラー株式会社 Advent Calendar 2021の4日目の記事「依存関係逆転の原則」です。"
categories: ["Design"]
aliases:
    - /posts/dip/
draft: false
---

この記事はフラー株式会社 Advent Calendar 2021の4日目の記事です。

- [フラー株式会社 Advent Calendar 2021のカレンダー \| Advent Calendar 2021 \- Qiita](https://qiita.com/advent-calendar/2021/fuller-inc)

3日目の記事は `@masaya82` さんによる「[スクリーンリーダー対応の備忘録 \- Qiita](https://qiita.com/masaya82/items/b55a86f9fc7175231d0b)」でした。

## 依存関係逆転の原則

書籍「Clean Architecture 達人に学ぶソフトウェアの構造と設計」（以下、Clean Architectureと呼ぶ。本記事でのClean Architectureとは、本書のことを指す）を読んでいると、中盤に差し掛かったところでSOLID原則の話が出てきます。

SOLID原則の話の中で**依存関係逆転の原則**に触れられていますが、自分は長らくこの原則の意味がよく理解できませんでした。最近になって「もしかしてこういう意味なのかもしらん」というくらいの理解が得られてきましたのでテキストにしてみました、というのがこの記事です。

### よく出てくる図

Clean Architectureには、こんな感じの図が出てきます。

{{< figure src="/images/dip-1.webp" title="Fig-1 よく出てくる図（Robert、p.105より筆者作成）" class="center" >}}

まずはこの図をよく見て...忘れます。

### 依存関係とは

依存関係逆転の原則とは、依存関係が逆転するという原則です。では、依存関係とはなんでしょうか。

難しく考える必要はありません。 `A` というモジュールから `B` というモジュールを使う、というだけのことを「 `A` が `B` に依存する」とお洒落に言い換えているだけです。

具体的なコードにするとこういう状況です。 `A` から `B` を使っているので、 `A` は `B` に依存している、という状況です。この「 `A` は `B` に依存している」という関係を、 `A` と `B` の依存関係と呼びます。依存関係とは、つまるところこれだけのことです。

```java
package a;

import b.B;

public class A {
  public B b = new B();

  public A() {
    b.hello();
  }
}
```

```java
package b;

public class B {
  public void hello() {
    System.out.println("Hello!");
  }
}
```

### 依存関係逆転前

依存関係逆転の原則...と言うからには、逆転前の状態があるはずです。これがその図です。

{{< figure src="/images/dip-2.webp" title="Fig-2 依存関係が逆転する前の図" class="center" >}}

この図は、

1. ApplicationがConcreteImpl（具象）に依存していること（実線の矢印）
2. 制御の流れがApplicationからConcreteImplに向いていること（波線の矢印）

ということを表しています。また、ApplicationとConcreteImplの間に引かれた波線は、「抽象と具象の区切り」（Robert、p.105）です。

ソースコードで表すと、こんな感じになります。

```java
package application;

import service.ConcreteImpl;

public class Application {
  public ConcreteImpl service;
}
```

```java
package service;

public class ConcreteImpl {
}
```

### 依存関係逆転

この図の依存関係を逆転させます。依存関係を逆転させるとは、ここでは実線の矢印を具象から抽象に向ける、ということを意味しています。

{{< figure src="/images/dip-3.webp" title="Fig-3 依存関係が逆転した後の図" class="center" >}}

抽象層にServiceというinterfaceを設け、ConcreteImplはServiceを実装します。それだけで、抽象から具象に向かっていた下向きの矢印が、具象から抽象への上向の矢印に変わりました。

ソースコードで確認します。

```java
package application;

public class Application {
  public Service service;
}
```

```java
package application;

public interface Service {
}
```

```java
package service;

import application.Service;

public class ConcreteImpl implements Service {
}
```

import文に着目すると、beforeではApplicationがserviceパッケージに依存していたのに対し、afterではConcreteImplがapplicationパッケージに依存する、というように、依存の向きが入れ替わっていることがわかります。

すなわち、

1. 具象が抽象に依存している（実線の矢印）
2. 制御の流れが抽象から具象に向いている（波線の矢印）

というようになっています。

この、実線（依存の方向）と波線（制御の流れ）の向きが反対方向に向いている状態が、依存関係が逆転した状態です。

依存関係逆転の原則とは、畢竟、「**ソースコードにあるすべての依存関係は、インターフェイスを挿入することで逆転可能**」（Robert、p.69）ということを言っているに過ぎないのです（ちゃんとClean Architectureに書いてありました）。

### まとめ

駆け足で依存関係逆転の原則について説明してきましたが、じゃあ具体的にどう実装するのかとか、抽象と具象の区切りって何だとか、たくさんの疑問符が頭の中を駆け巡っているかもしれません。その答えは...皆さんの目で確かめてください。

ということで、5日目の記事は `@m-coder` さんによる「[DroidKaigi2021に登壇した話｜m\-coder｜note](https://note.com/m_coder/n/n74901d195d83)」です。

### 参考文献

1. [Clean Architecture 達人に学ぶソフトウェアの構造と設計 \| Robert C\.Martin, 角 征典, 高木 正弘 \|本 \| 通販 \| Amazon](https://www.amazon.co.jp/dp/4048930656)（最終アクセス日：2021年12月1日）
2. [依存性逆転の原則 \- Wikipedia](https://ja.wikipedia.org/wiki/%E4%BE%9D%E5%AD%98%E6%80%A7%E9%80%86%E8%BB%A2%E3%81%AE%E5%8E%9F%E5%89%87)（最終アクセス日：2021年12月1日）
