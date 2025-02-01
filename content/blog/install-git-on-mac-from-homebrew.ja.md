---
title: "[MacOS] HomebrewからGitをインストールする"
date: 2023-02-21T22:28:08+09:00
description: "MacOS（Venture 13.0、Apple M2、MacBook Pro 2022）でHomebrewを用いたGitをインストールする方法の忘備録です。"
categories: ["Git", "MacOS"]
draft: false
---

まったく目新しい情報ではないですが、MacOS（Venture 13.0、Apple M2、MacBook Pro 2022）でHomebrewを用いてGitをインストールする方法をメモしておきます。せっかく新しいMacを買ったので。

- MacOS Venture 13.0
- Apple M2
- MacBook Pro 2022

## Homebrewのインストール

Homebrewはプリインストールされていないので、インストールします。

```shell
% brew -v
zsh: command not found: brew
```

[Homebrew](https://brew.sh/index_ja)のインストールスクリプトを実行します。今回は以下のコマンドでした。

```shell
 % /bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/HEAD/install.sh)"
 // 略
 ==> Next steps:
 - Run these two commands in your terminal to add Homebrew to your PATH:
     (echo; echo 'eval "$(/opt/homebrew/bin/brew shellenv)"') >> /Users/okuzawats/.zprofile
     eval "$(/opt/homebrew/bin/brew shellenv)"
 - Run brew help to get started
 - Further documentation:
     https://docs.brew.sh
 % (echo; echo 'eval "$(/opt/homebrew/bin/brew shellenv)"') >> /Users/okuzawats/.zprofile
 % eval "$(/opt/homebrew/bin/brew shellenv)"
```

## Gitのインストール

この時点でGitがインストールされていますが、このGitではなく、Homebrewからインストールしたものを使いたいのです。このGitは、MacOSにプリインストールされていたものだと思います。バージョンが古かったり、手軽に更新できなかったりするため、HomebrewからインストールしたGitを使うようにしておきたいわけです。

```shell
% git -v
git version 2.37.1 (Apple Git-137.1)
```

Homebrew経由でGitをインストールします。この時点ではまだApple Gitの方が使われています。

```shell
% brew install git
% git -v          
git version 2.37.1 (Apple Git-137.1)
```

MacOSのデフォルトシェルは、Catalinaからzshになっています。 `~/.zshrc` に以下を書いて、パスを通します。

```shell
export PATH="/opt/homebrew/bin:$PATH"
```

```shell
% source ~/.zshrc
% git -v         
git version 2.39.2
% which git
/opt/homebrew/bin/git
```

適当に `user.name` と `user.email` を設定して終わります。

```shell
% git config --global user.name "okuzawats"
% git config --global user.email okuzawats@gmail.com
```
