---
title: "[Android] SnackbarをBuilderパターンっぽく表示したい"
date: 2022-01-10T22:44:00+09:00
description: "AndroidのSnackbarをBuilderパターンっぽく表示する方法を模索した時のメモです。"
categories: ["Android"]
draft: false
---

2022年の今、遅きに失したというか今更感があるんですが...。AndroidのSnackbarについて、テキストの色、背景色、アクションなどよく変更する設定をBuilderパターンっぽく書きたいなと思いまして、ちょっと試してみました、というのがこの記事です。

## Builderクラスを作るパターン

最初に試したのが、SnackbarBuilderというクラスを作るパターン（ソースコードは後述します）。本当は `Snackbar.Builder()` というようにBuilderのコンストラクタを呼び出したかったのですが、Kotlinの文法ではいい感じにできなさそうです。そのため、SnackbarBuilderというクラスを作った、というのがこのパターンです。

```kotlin
SnackbarBuilder()
  .view(view)
  .text("Awesome Text")
  .textColor(R.color.text_color)
  .backgroundColor(R.color.background_color)
  .length(Snackbar.LENGTH_LONG)
  .action("Action") { /* TODO */ }
  .actionTextColor(R.color.text_color)
  .build()
  .show()
```

本来やりたかったことがだいたいできているんですが、viewとtextは必須にしたいです（lengthはデフォルト値を決めておけば必須でなくても良い）。SnackbarBuilderのコンストラクタでviewとtextを受け取るようにもできますが、それならば標準で用意されている `Snackbar#make` を使った方が筋が良さそうです。

ということで、没案となったSnackbarBuilderの（実験的な）ソースコードはこちらです。このコードブロックのあとに改善版を示しますので、改善版をすぐに見たい方は急いでスクロールしてください。

```kotlin
class SnackBarBuilder {
  private var view: View? = null
  private var string: String? = null
  @ColorRes private var textColor: Int? = null
  @ColorRes private var backgroundColor: Int? = null
  private var length: Int = Snackbar.LENGTH_SHORT
  private var actionString: String? = null
  private var action: ((View) -> Unit)? = null
  @ColorRes private var actionTextColor: Int? = null

  fun view(view: View): SnackBarBuilder =
    this.also {
      it.view = view
    }

  fun text(string: String): SnackBarBuilder =
    this.also {
      it.string = string
    }

  fun textColor(@ColorRes textColor: Int): SnackBarBuilder =
    this.also {
      it.textColor = textColor
    }

  fun backgroundColor(@ColorRes backgroundColor: Int): SnackBarBuilder =
    this.also {
      it.backgroundColor = backgroundColor
    }

  fun length(length: Int): SnackBarBuilder =
    this.also {
      it.length = length
    }

  fun action(actionText: String, action: (View) -> Unit): SnackBarBuilder =
    this.also {
      it.actionString = actionText
      it.action = action
    }

  fun actionTextColor(@ColorRes actionTextColor: Int): SnackBarBuilder =
    this.also {
      it.actionTextColor = actionTextColor
    }

  fun build(): Snackbar {
    val view = requireNotNull(view)
    val string = requireNotNull(string)

    val snackBar = Snackbar.make(view, string, length)

    if (textColor != null) {
      snackBar.view
          .findViewById<TextView>(R.id.snackbar_text)
          .setTextColor(
            ContextCompat.getColor(view.context, requireNotNull(textColor))
          )
    }

    if (backgroundColor != null) {
      snackBar.view
          .setBackgroundColor(
            ContextCompat.getColor(view.context, requireNotNull(backgroundColor))
          )
    }

    if (actionString != null && action != null) {
      snackBar.setAction(actionString, action)
    }

    if (actionTextColor != null) {
      snackBar.setActionTextColor(
        ContextCompat.getColor(view.context, requireNotNull(actionTextColor))
      )
    }

    return snackBar
  }
}
```

長い！

## 拡張関数で頑張るパターン

次に試したのが、Kotlinの拡張関数で頑張るパターンです。こちらのパターンは、標準で用意されている `Snackbar#make` を使ってSnackbarのインスタンスを生成した後、その他の属性値をBuilderパターンっぽく渡せるようにしています。とりあえずは、この実装で満足しました。

```kotlin
Snackbar.make(view, "Awesome Text", Snackbar.LENGTH_LONG)
  .textColor(R.color.text_color)
  .backgroundColor(R.color.background_color)
  .action("Action") { /* TODO */ }
  .actionTextColor(R.color.text_color)
  .show()
```

Kotlinの拡張関数で頑張るパターンのソースコードを以下に示します。Snackbarに対して自身を返す拡張関数を定義して、 `also` 内で追加の属性値を設定しています。先程の実装よりもこちらの実装の方が短く書けますね。

```kotlin
fun Snackbar.textColor(@ColorRes color: Int): Snackbar =
  this.also {
    it.view
      .findViewById<TextView>(R.id.snackbar_text)
      .setTextColor(ContextCompat.getColor(context, color))
  }

fun Snackbar.backgroundColor(@ColorRes color: Int): Snackbar =
  this.also {
    it.view
      .setBackgroundColor(ContextCompat.getColor(context, color))
  }

fun Snackbar.action(text: String, action: (View) -> Unit): Snackbar =
  this.also {
    it.setAction(text, action)
  }

fun Snackbar.actionTextColor(@ColorRes color: Int): Snackbar =
  this.also {
    it.setActionTextColor(ContextCompat.getColor(context, color))
  }
```

今更感がすごいですが、きれいに書けるようになって自分的には満足です。
