---
title: "ADR-Toolsを使ってADR（Architecture Decision Record）を作る"
date: 2023-08-11T15:53:06+09:00
description: "ADR（Architecture Decision Record）を管理するためのCLIであるADR-Toolsを使ってみるテスト。"
categories: ["Documentation"]
draft: false
---

[npryce/adr-tools](https://github.com/npryce/adr-tools)は、ADR（Architecture Decision Record）を管理するためのCLIです。ADRの作成や置き換えなど、いくつかの機能を提供しています。

macOSであればHomebrewでインストールできます。

```shell
% brew install adr-tools
```

以下のコマンドでADRのディレクトリが作成されます。デフォルトでは、`/doc/adr/` となります。

```shell
% adr init
```

## ADRとADR-Tools

ADRは、[Documenting Architecture Decisions](https://cognitect.com/blog/2011/11/15/documenting-architecture-decisions)が初出であるとのこと (Mark, Meal (2022), p.289)。

ADR-Toolsの使い方については、[hasCode.com » Blog Archive » Managing Architecture Decision Records with ADR-Tools](https://www.hascode.com/2018/05/managing-architecture-decision-records-with-adr-tools/)が詳しいです。

日本語で読める書籍では、「ソフトウェアアーキテクチャの基礎 エンジニアリングに基づく体系的アプローチ」 (Mark Richards, Neal Ford (2022))、「エンジニアリングマネージャーのしごと チームが必要とするマネージャーになる方法」 (James Stanier (2022)) に記述が見られます。

## ADR-Tools

以下のようなコマンドで、ADRの雛形を生成します。"absolutely important architecture decision" の部分は自由に入力します。

```shell
% adr new absolutely important architecture decision
```

`0003-absolutely-important-architecture-decision.md` というような名前でMarkdownファイルが作成されます。このファイル名の `0003` の部分は連番で作成されます。ファイルの中身は以下のようなものになります。「ADRの基本構造は、タイトル、ステータス、コンテキスト、決定、影響という5つの主要セクションから構成される (Mark, Meal (2022), p.289)」 という説明のとおりです。

```
# 3. absolutely important architecture decision

Date: 2023-08-11

## Status

Accepted

## Context

The issue motivating this decision, and any context that influences or constrains the decision.

## Decision

The change that we're proposing or have agreed to implement.

## Consequences

What becomes easier or more difficult to do and any risks introduced by the change that will need to be mitigated.
```

## ADR-Toolsを使って嬉しい時

ADR-Toolsを用いる主なメリットとして、ソースコードと同じ（Gitなどの）リポジトリでADRを管理できることが挙げられます。また、例えばGitHubを使っている場合は、ADR作成のPull Requestを作成し、Pull Requestが承認されるまでADRの変更が可能となるでしょう（実際に試したことはないので実効性は不明）。

反対に、この方針が組織やチームのドキュメント管理方針と異なる場合は、ADR-Toolsを用いてソースコードと同じリポジトリでADRを管理するのはやめた方が良さそうです。

## Reference

1. [npryce/adr-tools: Command-line tools for working with Architecture Decision Records](https://github.com/npryce/adr-tools)（最終アクセス日：2023年8月11日）
2. [Documenting Architecture Decisions](https://cognitect.com/blog/2011/11/15/documenting-architecture-decisions)（最終アクセス日：2023年8月11日）
3. [hasCode.com » Blog Archive » Managing Architecture Decision Records with ADR-Tools](https://www.hascode.com/2018/05/managing-architecture-decision-records-with-adr-tools/)（最終アクセス日：2023年8月11日）
4. Mark Richards, Neal Ford, 島田浩二 (訳), (2022), ソフトウェアアーキテクチャの基礎 エンジニアリングに基づく体系的アプローチ, オライリー・ジャパン
5. James Stanier, 吉羽龍太郎 (訳), 永瀬美穂 (訳), 原田騎郎 (訳), 竹葉美沙 (訳), (2022), エンジニアリングマネージャーのしごと チームが必要とするマネージャーになる方法, オライリー・ジャパン
