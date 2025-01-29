---
title: "[Android] RecyclerViewのLayoutManagerをxmlで指定する"
description: "AndroidのRecyclerViewのLayoutManagerをxmlで指定する方法です。"
date: 2019-10-26T22:19:29+09:00
categories: ["Android"]
aliases:
    - /posts/set-layoutmanager-in-xml/
draft: false
---

RecyclerViewのLayoutManagerをKotlin / Javaのコードで指定しているケースを散見しますが、RecyclerViewのLayoutManagerはxml側でも定義できます。個人的には、LayoutManagerをxml側で定義した方がロジックとレイアウトを分離できて良いと感じています。

xml側でRecyclerViewのLayoutManagerを指定する方法は、以下のように `app:layoutManager` にLayoutManagerを指定します。

```xml
<androidx.recyclerview.widget.RecyclerView
    android:id="@+id/recycler_view"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    app:layoutManager="androidx.recyclerview.widget.LinearLayoutManager"/>
```

上記の例ではLayoutManagerにLinearLayoutManagerを指定しています。LinearLayoutManagerは、Orientationに縦か横のどちらかを指定できます。デフォルトのOrientationは縦ですので、特に指定がなければ縦に要素が並びます。LinearLayoutManagerのOrientationを明示的に指定するには、 `orientation` に `vertical` か `horizontal` を指定します。以下は `vertical` の例です。

```xml
<androidx.recyclerview.widget.RecyclerView
    android:id="@+id/recycler_view"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:orientation="vertical"
    app:layoutManager="androidx.recyclerview.widget.LinearLayoutManager"/>
```

LayoutManagerにGridLayoutManagerを指定する場合、一行あたりのセルの数は、 `spanCount` で指定できます。以下は `spanCount` として2を指定する例です。

```xml
<androidx.recyclerview.widget.RecyclerView
    android:id="@+id/recycler_view"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:orientation="vertical"
    app:spanCount="2"
    app:layoutManager="androidx.recyclerview.widget.GridLayoutManager"/>
```
