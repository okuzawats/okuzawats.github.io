---
title: "自己カプセル化"
date: 2022-06-05T06:28:00+09:00
description: "自己カプセル化についてです。"
categories: ["Design"]
draft: false
---

**自己カプセル化（Self-Encupsulation）** は、クラスの持つフィールドに対してクラス内部からアクセスする場合もアクセサメソッドを経由して行うパターンです。

以下のコード（Javaっぽい疑似コードです）は、自己カプセル化のパターンを適用する前のコードです。コンストラクタおよび `getUserName` で、アクセサメソッドを経由せずに `firstName` 、 `lastName` にアクセスしています。

```java
class UserName {
  private String firstName;
  private String lastName;

  public UserName(String firstName, String lastName) {
    this.firstName = firstName;
    this.lastName = lastName;
  }

  public String getUserName() {
    return firstName + " " + lastName;
  }
}
```

`firstName` と `lastName` に対するアクセサメソッドを追加して、 `firstName` 、 `lastName` へのアクセスをアクセサメソッド経由に変更します。

```java
class UserName {
  private String firstName;
  private String lastName;

  public UserName(String firstName, String lastName) {
    setFirstName(firstName);
    setLastName(lastName);
  }

  public String getUserName() {
    return getFirstName() + " " + getLastName();
  }

  private void setFirstName(String firstName) {
    this.firstName = firstName;
  }

  private String getFirstName() {
    return firstName;
  }

  private void setLastName(String lastName) {
    this.lastName = lastName;
  }

  private String getLastName() {
    return lastName;
  }
}
```

これだけだとコード量が増えただけで旨味がないですが、例えばsetterにassertionを追加した場合はどうでしょうか。この場合、 `firstName` への代入を必ずsetter経由で行うのであれば、 `firstName` に `null` を代入できないことが保証されます。

```java
  private void setFirstName(String firstName) {
    assert firstName != null;
    this.firstName = firstName;
  }
```

また、例えば仮に「 `firstName` は10文字以上を設定しなければならない」という仕様が追加された場合であっても、容易にそれを保証することができます[^1]。

```java
  private void setFirstName(String firstName) {
    assert firstName != null;
    assert firstName.length >= 10;
    this.firstName = firstName;
  }
```

クラスの内部からアクセサメソッド経由でフィールドにアクセスするメリットは他にもあって、例えば、クラスの継承を行う場合にサブクラスでのフックを提供できるということが挙げられます。

> But there are common circumstances where self-encapsulation is worth the effort. If there is logic in the setter, then it's wise to consider it for any internal updates too. Another circumstance is when the class is part of an inheritance structure, in which case the accessors provide valuable hook points for subclasses to override behavior. (Martin Fowler(2017))

## Reference

1. Martin Fowler、（2017）、SelfEncapsulation、Retrieved from https://martinfowler.com/bliki/SelfEncapsulation.html（最終アクセス日：2022/06/05）

[^1]: 「仮に」です。
