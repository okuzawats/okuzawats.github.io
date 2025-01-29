---
title: "goenvでGoのバージョンを管理する"
date: 2024-09-23T09:26:00+09:00
description: "goenvを用いてGo言語のバージョンを管理する方法です。"
categories: ["Go"]
draft: false
---

趣味でGoを書く機会が増えてきたので、そろそろ `anyenv` 的な方法でGoのバージョンを管理するか...と重い腰を上げました。Go言語のバージョンマネージャで広く使われているものは、 `pyenv` や `rbenv` などの流れをくむ `goenv` と `nvm` などの流れをくむ `gvm` があります。なお、自分の使っているOSはmacOS Sonomaです。

- [go-nv/goenv: :blue_car: Like pyenv and rbenv, but for Go.](https://github.com/go-nv/goenv?tab=readme-ov-file)
- [moovweb/gvm: Go Version Manager](https://github.com/moovweb/gvm)

どちらもGitHubにリポジトリが存在しているので最近のメンテナンスの状況を見つつ、今回は `goenv` を用います。

## Goのアンインストール

まずはグローバルにインストールされているGoはもう不要になるので、アンインストールしておきます。

```console
% go version
go version go1.23.1 darwin/arm64
% brew uninstall go
Uninstalling /opt/homebrew/Cellar/go/1.23.1... (13,231 files, 268.2MB)
% go version
zsh: command not found: go
```

## goenvのインストール

Homebrewで `goenv` をインストールします。

```console
% brew install goenv
```

[INSTALL.md](https://github.com/go-nv/goenv/blob/master/INSTALL.md)を参考にしつつ、自分はzshを使っているので `~/.zshrc` に以下を書き込み、最後に `source ~/.zshrc` を実行します。

```shell
export GOENV_ROOT=$HOME/.goenv
export PATH=$GOENV_ROOT/bin:$PATH
eval "$(goenv init -)"
```

## Goの特定バージョンをインストールする

元々使っていた `Go 1.23.1` を使いたいので、このバージョンをインストールします。

```console
% goenv install 1.23.1
```

`goenv local` で該当バージョンを指定します。

```console
% goenv local 1.23.1
% go version
go version go1.23.1 darwin/arm64
```

OK。

## References

1. [go-nv/goenv: :blue_car: Like pyenv and rbenv, but for Go.](https://github.com/go-nv/goenv?tab=readme-ov-file)(https://developer.apple.com/documentation/swiftui/view)（最終アクセス日：2024/09/23）
2. [moovweb/gvm: Go Version Manager](https://github.com/moovweb/gvm)（最終アクセス日：2024/09/23）
3. [goenv: ローカルのGoバージョンを柔軟に切り替えるツールの導入](https://zenn.dev/erueru_tech/articles/13e8512d9312de)（最終アクセス日：2024/09/23）
