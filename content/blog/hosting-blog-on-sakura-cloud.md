---
title: "さくらのクラウドのウェブアクセラレータとオブジェクトストレージで静的サイトを配信する（HugoとGitHub Actionsを添えて）"
date: 2023-11-15T05:35:00+09:00
description: "本ブログをさくらのクラウドのウェブアクセラレータとオブジェクトストレージを使って配信するようにしました。また、静的サイトの生成にHugo、そのデプロイにGitHub Actionsを使っています。"
categories: ["GitHub Actions", "Hugo"]
draft: false
---

本ブログを「さくらのクラウド」のCDN、[ウェブアクセラレータ](https://www.sakura.ad.jp/services/cdn/)とストレージサービスの[オブジェクトストレージ](https://cloud.sakura.ad.jp/specification/object-storage/)を使って配信するようにしました。

元々はNetlifyを使っていて機能的には何の不満もなかったんですが、Netlifyが簡単すぎて物足りなくなってきたので、新たな環境にお引越ししました。お金はかかりますが。ついでにブログ（Hugo）の多言語化も対応しました（英語のみ。英語の記事はこれから頑張って書きます！）。また、オブジェクトストレージにファイルをアップロードするためにGitHub Actionsを使っています。

備忘を兼ねて、設定手順をメモしておきます。

## オブジェクトストレージ

まずは元気にオブジェクトストレージのバケットを作成します。そんなに難しいことはないと思います。作成したら、GitHub Actionsを使ってファイルをアップロードするためのシークレットと、後述するウェブアクセラレータからオブジェクトストレージに繋ぐためのシークレットを追加しておきます。

GitHub Actionsで使う方のシークレットは、GitHubのリポジトリシークレットに入れておきます。GitHub Actionsのワークフローは、以下の記事を参考に書きました。

- [GitHub Actionsからさくらのクラウド オブジェクトストレージへアクセスする - Hateburo: kazeburo hatenablog](https://kazeburo.hatenablog.com/entry/2021/02/15/095632)

実際に使っているymlを一部記載するとこんな感じです。`./path/to/hugo/public/` と `//your-bucket-here/` 、およびS3エンドポイントは各自の環境にあわせて修正してください。また、Hugoのセットアップには、[peaceiris/actions-hugo](https://github.com/peaceiris/actions-hugo)を使用しています（いつもありがとうございます）。

```yml
jobs:
  upload:
    runs-on: ubuntu-latest
    steps:
      # 省略
      - name: setup Hugo
        uses: peaceiris/actions-hugo@v2
        with:
          hugo-version: '0.119.0'
      - name: build Hugo
        run: |
          cd ./path/to/hugo
          hugo --minify
      - name: sync to object storage
        env:
          AWS_ACCESS_KEY_ID: ${{ secrets.SAKURA_ACCESS_KEY_ID }}
          AWS_SECRET_ACCESS_KEY: ${{ secrets.SAKURA_SECRET_ACCESS_KEY }}
          AWS_REGION: eu-west-1
        run: |
          aws --endpoint-url=https://s3.isk01.sakurastorage.jp s3 --only-show-errors --delete sync ./path/to/hugo/public/ s3://your-bucket-here/
```

自分の理解が足らずにハマっていたんですが、`AWS_REGION` のところはオブジェクトストレージのコンソールに表示されている `jp-north-1` のようなオブジェクトストレージのリージョン**ではない**ことに注意してください。

## ウェブアクセラレータ

オブジェクトストレージへのアップロードができたら、次はウェブアクセラレータのサイトを作ります。ウェブアクセラレータでは、「独自ドメイン」形式と「サブドメイン」形式が選択できます。今回使いたいのは独自ドメイン形式です。本ブログはwwwなどのサブドメインのつかない、いわゆるネイキッドドメイン（APEXドメイン）で運用しているため、少し苦労しました。

ドメインの管理は他社から「さくらのドメイン」に移管しておいたのですが、「さくらのドメイン」のネームサーバではこの設定ができません（[初期設定（独自ドメイン利用）](https://manual.sakura.ad.jp/cloud/webaccel/manual/settings-domain.html)を参照）。そのため、今回は「さくらのクラウドDNS」を使いました（後述）。

ウェブアクセラレータでサイトを作成後、独自ドメインの有効化をするために、移行前のNetlifyのダッシュボードからDNSにtxtレコードを追加しました。txtレコードの中身についてはウェブアクセラレータのコンソールに表示されるので、その指示に従います。これでウェブアクセラレータで独自ドメインのサイトを有効化できるようになります。

次に、ドメインの管理画面（自分の場合は「さくらのドメイン」）から、Whoisのネームサーバを「さくらのクラウド」のDNSに設定します。「さくらのクラウド」のコンソールから、グローバル > DNSと開くと設定値が書いてあります。あわせて、DNSゾーンを作成した上でリソースレコードを追加します。ここにALIASやらCNAMEやらを追加して、忘れずに「反映」します（「反映」ボタンが微妙にわかりにくいです）。

ウェブアクセラレータではLet's Encryptを用いてSSL証明書を作ってくれるんですが、しばらく待たないと有効化できません。自分の場合は半日（12時間）ほど待つと有効化できるようになりました。「Let's Encrypt 自動更新証明書を使用する（無料）」を選択して有効化します。

## 終わり

以上で完了です。体感にすると2週間くらいかけて移行作業をしていた気がするんですが、記事にすると短いですね。「さくらのクラウド」はコンソールが使いやすくて良かったです。サポートに問い合わせた時も丁寧に対応してもらえて、無事に移行が完了しました🎉

## Reference

1. [ウェブアクセラレータ](https://www.sakura.ad.jp/services/cdn/)（最終アクセス日：2023年11月15日）
1. [オブジェクトストレージ](https://cloud.sakura.ad.jp/specification/object-storage/)（最終アクセス日：2023年11月15日）
1. [GitHub Actionsからさくらのクラウド オブジェクトストレージへアクセスする - Hateburo: kazeburo hatenablog](https://kazeburo.hatenablog.com/entry/2021/02/15/095632)（最終アクセス日：2023年11月15日）
1. [peaceiris/actions-hugo: GitHub Actions for Hugo ⚡️ Setup Hugo quickly and build your site fast. Hugo extended, Hugo Modules, Linux (Ubuntu), macOS, and Windows are supported.](https://github.com/peaceiris/actions-hugo)（最終アクセス日：2023年11月15日）
1. [初期設定（独自ドメイン利用） | ウェブアクセラレータ | さくらのクラウド ドキュメント](https://manual.sakura.ad.jp/cloud/webaccel/manual/settings-domain.html)（最終アクセス日：2023年11月15日）
1. [DNS(アプライアンス) | さくらのクラウド ドキュメント](https://manual.sakura.ad.jp/cloud/appliance/dns/index.html)（最終アクセス日：2023年11月15日）
