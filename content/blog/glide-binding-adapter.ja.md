---
title: "[Android] DataBindingのBindingAdapterでGlideを使う"
description: "AndroidのDataBindingのBindingAdapterでGlideを使う方法です。"
date: 2019-11-02T21:54:58+09:00
categories: ["Android"]
aliases:
    - /posts/glide-binding-adapter/
draft: false
---

bindした時にGlideで画像を読み込んで欲しいのでBindingAdapterを実装します。

こういうデータクラスを作ります。もちろん他のプロパティがあっても大丈夫です。

```kotlin
data class Image(
    val uri: Uri
)
```

レイアウトファイルを作ります。この例はRecyclerViewのアイテム用のレイアウトファイルです。先ほど作ったImageをvariableとしてバインドします。ImageViewに対して `app:imageURI="@{image.uri}"` としてUriを渡します。

```xml
<?xml version="1.0" encoding="utf-8"?>
<layout
        xmlns:android="http://schemas.android.com/apk/res/android"
        xmlns:app="http://schemas.android.com/apk/res-auto">

    <data>

        <variable
                name="image"
                type="com.okuzawats.sampleapp.model.Image"/>

    </data>

    <androidx.constraintlayout.widget.ConstraintLayout
            android:layout_width="match_parent"
            android:layout_height="wrap_content">

        <ImageView
                android:id="@+id/thumbnail"
                android:layout_width="0dp"
                android:layout_height="wrap_content"
                android:adjustViewBounds="true"
                app:imageURI="@{image.uri}"
                app:layout_constraintTop_toTopOf="parent"
                app:layout_constraintStart_toStartOf="parent"
                app:layout_constraintEnd_toStartOf="@id/information"/>

        <LinearLayout
                android:id="@+id/information"
                android:orientation="vertical"
                android:layout_width="0dp"
                android:layout_height="wrap_content"
                app:layout_constraintTop_toTopOf="parent"
                app:layout_constraintStart_toEndOf="@id/thumbnail"
                app:layout_constraintEnd_toEndOf="parent">

            <!-- 略 -->

        </LinearLayout>

    </androidx.constraintlayout.widget.ConstraintLayout>

</layout>
```

あとは適当なKotlinのファイルを作ってBindingAdapterを実装すればOKです。

```kotlin
@BindingAdapter("imageURL")
fun ImageView.imageURI(uri: URI) {
    Glide.with(this)
        .load(uri)
        .into(this)
}
```
