---
title: "[Android] スクリーンショットを撮影するShell Script"
date: 2021-06-15T20:46:50+09:00
description: "adbを利用してAndroidのスクリーンショットを撮影するShell Scriptです。"
categories: ["Android"]
aliases:
    - /posts/shellscript-to-take-a-screenshot-of-android-device/
draft: false
---

Androidのスクリーンショットを撮影するShell Scriptです。`adb` を使える状態となっていることが前提です。ローカルの `/sdcard/` にスクリーンショットを一時的に保存し、その後、カレントディレクトリに移動します。画像のフォーマットはPNGで、 `{timestamp}.png` というファイル名で保存します。

```shell
filename=`date "+%s"`
adb shell screencap -p /sdcard/${filename}.png
adb pull /sdcard/${filename}.png
adb shell rm /sdcard/${filename}.png
```

`adb` の `-p` というオプションは、スクリーンショットをPNGで撮影するためのものです。ドキュメントではこのオプションが省略されていたので、なくてもいいのかもしれません🤔

上記のスクリプトを `screenshot.sh` として保存し、実行します。

```shell
% sh screenshot.sh
```

以下の環境で動作を確認しました。

- macOS Big Sur 11.4
- zsh 5.8

## References

1. [Android Debug Bridge \(adb\)  \|  Android Developers](https://developer.android.com/studio/command-line/adb#screencap) (最終アクセス日：2021年6月15日)
