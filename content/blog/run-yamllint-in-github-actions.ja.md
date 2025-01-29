---
title: "GitHub Actionsでyamlにlintをかける"
date: 2024-06-25T05:07:00+09:00
description: "GitHub Actionsでyamlにlintをかける方法です。"
categories: ["GitHub Actions"]
draft: false
---

yamlのlinterとして[adrienverge/yamllint](https://github.com/adrienverge/yamllint)（以下、yamllintと呼ぶ）が広く使われています。また、yamllintをGitHub Actions上で実行し、GitHubのPull Requesetに指摘事項をコメントするためのactionとして、[reviewdog/action-yamllint](https://github.com/reviewdog/action-yamllint)（以下、action-yamllintと呼ぶ）が存在します。

話としては「action-yamllintを使いましょう」で終わってしまうんですが、備忘のために使用例を示します。

まずはyamllintの設定用ファイルである、`.yamllint` を作成し、ルールを記述します。ルールの詳細については、yamllintのドキュメントを参照してください。また、このルールについては、[GitHubActionsのCI/CDにyamllintを組み込んで作業の安全性を高める](https://qiita.com/EnKUMA/items/6974e310221fc9765c2f)を参考にして作成しました。

```yaml
yaml-files:
  - '*.yaml'
  - '*.yml'

rules:
  braces:
    level: warning
    min-spaces-inside: 1
    max-spaces-inside: 1
  brackets:
    level: warning
    min-spaces-inside: 1
    max-spaces-inside: 1
  colons:
    level: warning
  commas:
    level: warning
  comments:
    level: warning
    min-spaces-from-content: 1
  comments-indentation: disable
  document-end: disable
  document-start: disable
  empty-lines: disable
  empty-values: disable
  hyphens: disable
  indentation:
    level: warning
    indent-sequences: consistent
  key-duplicates: enable
  key-ordering: disable
  line-length: disable
  new-line-at-end-of-file:
    level: warning
  new-lines: enable
  octal-values: disable
  quoted-strings:
    quote-type: double
  trailing-spaces: disable
  truthy: disable
```

次に、GitHub Actionsのワークフローを書いていきます。典型的な設定例だと思いますので、説明は割愛します。「TODO」としている箇所に、action-yamllintのステップを追加していきます。

```yaml
name: "yamllint"

on:
  pull_request:
    branches:
      - "main"
    types: [ "opened", "synchronize" ]

jobs:
  yamllint:
    runs-on: "ubuntu-latest"
    steps:
      - name: "checkout"
        uses: "actions/checkout@v4"
      # TODO
```

action-yamllintのステップを追加します。TODOとしている箇所を以下のように置き換えます。

```yaml
      - name: "review yaml format"
        uses: "reviewdog/action-yamllint@v1"
        with:
          github_token: "${{ secrets.GITHUB_TOKEN }}"
          reporter: "github-pr-review"
          level: "warning"
          yamllint_flags: "-c .yamllint ./.github/workflows/"
          fail_on_error: true
```

`yamllint_flags` に上述の `.yamllint` と対象となるyamlの置かれたパス（今回はGitHub Actionsのワークフローファイルの置かれるパス）を渡しています。

また、`fail_on_error` を `true` に設定することで、yamlのlintにwarningがあった場合（ `level` オプションで設定）にCIを失敗させています。[GitHubActionsのCI/CDにyamllintを組み込んで作業の安全性を高める](https://qiita.com/EnKUMA/items/6974e310221fc9765c2f)では、CIを失敗させるために他のactionと併用する方法が紹介されていますが、この `fail_on_error` オプションを利用することで、action-yamllintのみで完結させることができます。

お気付きでしょうか。上記のGitHub Actionsのワークフローのyamlについても、`.yamllint` に設定したルールでlintがかけられています。

元は、手癖で以下のようにyamlを書いていました。

```yaml
name: yamllint

on:
  pull_request:
    branches:
      - "main"
    types: [opened, synchronize]
```

今回作成したGitHub Actionsのワークフローにより、上記のyamlについて以下のようなコメントがPull Requestに付けられ、yamlのlintエラーとなっている箇所が指摘されます。

{{< figure src="/images/yamllint.webp">}}

GitHub Actionsにおいてyamllintとaction-yamllintを活用することで、yamlを健全な状態に保つことができるようになります。

## References

1. [adrienverge/yamllint: A linter for YAML files.](https://github.com/adrienverge/yamllint)
2. [reviewdog/action-yamllint: Run yamllint with reviewdog](https://github.com/reviewdog/action-yamllint)
3. [GitHubActionsのCI/CDにyamllintを組み込んで作業の安全性を高める #YAML - Qiita](https://qiita.com/EnKUMA/items/6974e310221fc9765c2f)
