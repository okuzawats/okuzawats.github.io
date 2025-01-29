---
title: "GitHub ActionsでSonarCloudにカバレッジをアップロードする"
date: 2023-09-18T22:48:21+09:00
description: "GitHub ActionsからSonarCloudにカバレッジをアップロードする方法です。JaCoCoを用いたAndroidのカバレッジレポートの例を書いています。"
categories: ["Android", "GitHub Actions"]
draft: false
---

表記のことを行う方法をメモしておきます。

サンプルとしてAndroidアプリのカバレッジレポートをJaCoCoで出力する場合を取り上げています。JaCoCoについては[build.gradle.ktsでJaCoCoを動かす](https://okuzawats.com/blog/configure-jacoco/)などを参考にして設定済みであることを前提とします。

## GitHub Actionsのワークフロー

結論から先に書くと、こんな感じのステップを作ります。このステップを動かすために、いろいろな準備が必要です。

```yaml
jobs:
  test:
    # 省略
    steps:
      # 省略
      - name: sonarcloud
        uses: SonarSource/sonarcloud-github-action@master
        env:
          GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}
          SONAR_TOKEN: ${{ secrets.SONAR_TOKEN }}
        with:
          args: >
            -Dsonar.organization=your-organization-here
            -Dsonar.projectKey=your-project-key-here
            -Dsonar.coverage.jacoco.xmlReportPaths=./app/build/reports/jacoco/debugCoverage/debugCoverage.xml
```

## 準備

### CI baseの解析を有効にする

SonarCloudは、GitHubのリポジトリと連携した後、デフォルトブランチとPull Requestを自動で解析する設定になっています。SonarCloudのカバレッジ機能を有効にするためには、この設定を無効化して、Pull Requestベースでの解析を行うように変更する必要があります。

SonarCloudのプロジェクトのダッシュボードから、Administration > Analysis Methodを開き、Automatic Analysisを無効化します。もし"Administration"が表示されない場合は権限的な問題かもしれません。

### SONAR_TOKEN

GitHubのリポジトリシークレットに保存する、SonarCloudのトークンです。

このトークンは、SonarCloudのダッシュボードから、My Account > Securityを開き、"Generate Tokens"で作成できます。この辺りは、組織のポリシーがあればそれに従います。今回は自分個人のアカウントで試しているので、自分のアカウントでトークンを作成しました。

### オーガニゼーションとプロジェクトキー

yamlの以下の部分です。これら2つは、[SonarSource/sonarcloud-github-action](https://github.com/SonarSource/sonarcloud-github-action)で `with` を用いた構文を使う場合は必須のパラメータです。

```yaml
            -Dsonar.organization=your-organization-here
            -Dsonar.projectKey=your-project-key-here
```

SonarCloudのプロジェクトのダッシュボードから、Informationを開き、"About This Project"の下に表示されている"Project Key"と"Organization Key"をそれぞれ入力します。

### カバレッジレポートのパス

yamlの以下の部分です。カバレッジレポートのパスを設定します。言語によってパラメータが異なるので、[Test Coverage Parameters](https://docs.sonarcloud.io/enriching/test-coverage/test-coverage-parameters/)を参考に設定します。

```yaml
            -Dsonar.coverage.jacoco.xmlReportPaths=./app/build/reports/jacoco/debugCoverage/debugCoverage.xml
```
