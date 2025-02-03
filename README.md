# okuzawatsの日記

[![Deploy Hugo site to Pages](https://github.com/okuzawats/okuzawats.github.io/actions/workflows/hugo.yml/badge.svg)](https://github.com/okuzawats/okuzawats.github.io/actions/workflows/hugo.yml)

[okuzawatsの日記](https://okuzawats.com/)のソースコードです。ソースコードには、静的コンテンツをビルドするためのテーマ、コンテンツ、CI/CDの設定ファイル等を含みます。

## ビルド環境

[Hugo](https://gohugo.io/)を用いて静的コンテンツをビルドしています。各OSにHugoをインストールする方法は、以下のドキュメントを参照してください。本ディレクトリ構成は、Hugoのディレクトリ構成に従います。

- [Installation | Hugo](https://gohugo.io/installation/)

例えば、macOSに[Homebrew](https://brew.sh/)を用いてHugoをインストールするには、以下のコマンドを実行します。

```
brew install hugo
```

## CI/CD環境

GitHub Actionsを用いて、CI/CDを行っています。GitHub ActionsでHugoのビルドを実行し、GitHub Pagesに配信しています。

詳細は設定ファイルを参照してください。

## ホスティング環境

このリポジトリにおいて管理される、GitHub Pagesでホスティングされています。

ドメインの設定については以下のドキュメントを参照。

- [ドメインの設定 - okuzawatsのScrapbox](https://scrapbox.io/okuzawats/%E3%83%89%E3%83%A1%E3%82%A4%E3%83%B3%E3%81%AE%E8%A8%AD%E5%AE%9A)（要ログイン）

## ADR（Architecture Decision Record）

adr-toolsを用いてADRの管理を行います。

- [npryce/adr-tools: Command-line tools for working with Architecture Decision Records](https://github.com/npryce/adr-tools)

## CSSフレームワーク

new.cssを使用しています。

- [new.css](https://newcss.net/)

## ビールをおごる🍻

- [Pure CSS responsive "Fork me on GitHub" ribbon](https://codepo8.github.io/css-fork-on-github-ribbon/)

ビールの色はDeep Goldです。以下のサイトを参照しています。

- [SRM - BrewNote](https://brewnote.tokyo/kb/srm/)
