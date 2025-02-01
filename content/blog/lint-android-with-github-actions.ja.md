---
title: "GitHub ActionsでktlintとAndroid Lintを並列実行して、DangerでPRにまとめてコメントする🐝"
date: 2023-06-17T23:13:00+09:00
description: "GitHub ActionsでktlintとAndroid Lintを並列実行して、DangerでPRにまとめてコメントする方法のメモです。"
categories: ["Android", "GitHub Actions"]
draft: false
---

Androidアプリ開発での静的解析として広く使われているktlintとAndroid LintをGitHub Actions上で実行して、Dangerを使ってPRにコメントして欲しい、ということは良くあると思います。ktlintとAndroid Lintを直列に実行するのは簡単ですが、並列に実行できた方がCI待ちの時間が減って嬉しいですよね。そこまで難しくないのでやっていきます。

- [pinterest/ktlint: An anti-bikeshedding Kotlin linter with built-in formatter](https://github.com/pinterest/ktlint)
- [Improve your code with lint checks | Android Studio | Android Developers](https://developer.android.com/studio/write/lint)
- [Danger - Stop Saying "You Forgot To…" in Code Review](https://danger.systems/ruby/)

## `jobs.<job_id>.needs` 構文

GitHub Actionsでは、ひとつのワークフロー（YAMLファイル単位で作成される）に複数のジョブを定義できます。ひとつのワークフローにktlint、Android Lint、Dangerのジョブをそれぞれ定義すれば良さそうですが、Dangerの実行はktlintとAndroid Lintの両方の実行完了を待たないといけません。

ここで使えるのが、 `jobs.<job_id>.needs` 構文です。

- [GitHub Actions のワークフロー構文 - GitHub Docs](https://docs.github.com/ja/actions/using-workflows/workflow-syntax-for-github-actions#jobsjob_idneeds)

この構文を使って、

```yaml
jobs:
  ktlint:
    # do ktlint
  androidLint:
    # do android lint
  danger:
    needs:
      - ktlint
      - androidLint
    # do danger
```

というように、ktlintのジョブ、Android Lintのジョブ、Dangerのジョブをそれぞれ定義しておき、Dangerのジョブの依存ジョブとしてktlintおよびAndroid Lintのジョブを指定すれば良いです。

## Gemfile

Gemfileに必要なgemを追加しておきます。今回は、PRにktlintの指摘をコメントするために `danger-checkstyle_format` 、Android Lintの指摘をコメントするために `danger-android_lint` を使用します。

```
gem 'danger'
gem 'danger-checkstyle_format'
gem 'danger-android_lint'
```

- [noboru-i/danger-checkstyle_format: Danger plugin for checkstyle formatted xml file.](https://github.com/noboru-i/danger-checkstyle_format)
- [loadsmart/danger-android_lint: A Danger plugin for Android Lint](https://github.com/loadsmart/danger-android_lint)

## Dangerfile

Dangerfileでは前述の `danger-checkstyle_format` と `danger-android_lint` の設定を書いておきます。Android LintのGradleタスクは先にGitHub Actionsのジョブとして実行しておくので、 `skip_gradle_task` にtrueを設定しておきます。また、ファイルのパスはプロジェクトによって異なるパスとなっている可能性があるので、適宜修正します。

```
checkstyle_format.base_path = Dir.pwd
checkstyle_format.report 'app/build/ktlint.xml'

android_lint.report_file = 'app/build/reports/lint-results-debug.xml'
android_lint.skip_gradle_task = true
android_lint.lint
```

## upload-artifact、download-artifact

GitHub Actionsのジョブ間では、それぞれのジョブ内で生成されたファイルは共有されません。そのため、ktlintとAndroid Lintの出力したレポートファイルを何らかの方法でDangerのジョブに渡す必要があります。

この問題を解決するための方法はいくつかあると思いますが、今回はupload-artifactアクション、およびdownload-artifactアクションを使用します。

- [actions/upload-artifact](https://github.com/actions/upload-artifact)
- [actions/download-artifact](https://github.com/actions/download-artifact)

ktlintのジョブとAndroid Lintのジョブの最後に、upload-artifactアクションを用いて、それぞれのレポートファイルをアップロードしておきます。こうしておくと、あとからレポートファイルを参照することもできるというメリットもあります。

```yaml
- uses: actions/upload-artifact@v3
  with:
    name: ktlint-output
    path: 'app/build/ktlint.xml'
```

```yaml
- uses: actions/upload-artifact@v3
  with:
    name: androidlint-output
    path: 'app/build/reports/lint-results-debug.xml'
```

Dangerのジョブでは、Dangerの実行前に、download-artifactアクションを用いてこれらのファイルをダウンロードしておきます。これで、Dangerのジョブの中でそれぞれのファイルを使用可能な状態となります。

```yaml
- uses: actions/download-artifact@v3
  with:
    name: ktlint-output
    path: 'app/build/'
- uses: actions/download-artifact@v3
  with:
    name: androidlint-output
    path: 'app/build/reports/'
```

## GitHub Actionsのワークフローを実行

ここまで来たら、GitHub Actionsのワークフローを実行できます。試しに実行してみると、以下のようにktlintとAndroid Lintの警告・エラーがGitHubのPRにコメントされました。

{{< figure src="/images/lint-action.webp" title="Fig-1 GitHubのPRへのコメント" class="center" >}}

PRへのコメントについてはdanger-actionを使用しました。

- [MeilCli/danger-action: Execute danger action for GitHub Actions.](https://github.com/MeilCli/danger-action)

実際のYAMLファイルはGitHub上で参照可能ですので、見てみたい方は以下のリンクからどうぞ。

- [yumemi-android-engineer-codecheck/.github/workflows/danger.yml at main · okuzawats/yumemi-android-engineer-codecheck · GitHub](https://github.com/okuzawats/yumemi-android-engineer-codecheck/blob/main/.github/workflows/danger.yml)
