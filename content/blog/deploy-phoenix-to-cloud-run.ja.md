---
title: "PhoenixプロジェクトをCloud Runにデプロイする"
date: 2026-10-12T21:10:00+09:00
description: "PhoenixプロジェクトをCloud Runにデプロイしてみた話です。"
categories: ["Google Cloud", "Cloud Run"]
draft: false
---

Phoenixで作ったAPIサーバーをCloud Runにデプロイした手順のメモ。勉強を兼ねた備忘のための個人的なメモであり、実務上の参考とすることは意図してません。また、ところどころ省略します。

まず、miseでElixirとErlangを入れます。

```toml
[tools]
elixir = "1.20.4"
erlang = "29.1"
```

## Phoenixの準備

早速Phoenixプロジェクトを作成します。今回はEctoも含めて種々の設定を無効化しておきます。

```bash
mix phx.new . --app my_awesome_app --no-ecto --no-html --no-assets --no-gettext --no-mailer
mix setup
```

Phoenixプロジェクトを作成した後、適当にヘルスチェック用のAPIを生やしておきます。

```elixir
# lib/my_awesome_app_web/router.ex
scope "/", MyAwesomeAppWeb do
  get "/health", HealthController, :show
end
```

```elixir
# lib/my_awesome_app_web/controllers/health_controller.ex
defmodule MyAwesomeAppWeb.HealthController do
  use MyAwesomeAppWeb, :controller

  def show(conn, _params) do
    json(conn, %{status: "ok"})
  end
end
```

Phoenixは、本番環境においては `force_ssl` によってHTTPでのリクエストをHTTPSにリダイレクトします。ところが、Cloud Runではコンテナの前段にあるGoogle Front EndでHTTPSの通信を復号してHTTPに戻します。そのため、クライアントからHTTPSでアクセスしても、PhoenixはHTTPでリクエストを受け取ってHTTPSへリダイレクトしようとするので、リダイレクトが繰り返されてしまいます。

Google Front Endはリクエストをコンテナに転送する時に `X-Forwarded-Proto: https` ヘッダーを付け、HTTPSでリクエストを受けたことを示します。そこで、`rewrite_on: [:x_forwarded_proto]` を設定して、Google Front Endが付けた `X-Forwarded-Proto` を見てHTTPSのリクエストとして扱うようにします。

ただし、先ほど作成したヘルスチェック用のAPIについては例外にしておきます。ヘルスチェック用のAPIは、Cloud Runへのデプロイ時にプローブのパスとして指定します。プローブはGoogle Front Endを通らずにCloud Runの内部からリクエストされるので、`X-Forwarded-Proto: https` ヘッダーが付きません。`force_ssl` の対象のままだとプローブが失敗してコンテナが起動できないと判断されてしまいます。そのため、ヘルスチェック用のAPIは `force_ssl` の対象から除外しておきます。

```elixir
# config/prod.exs
config :my_awesome_app, MyAwesomeAppWeb.Endpoint,
  force_ssl: [
    rewrite_on: [:x_forwarded_proto],
    exclude: [
      paths: ["/health"],
      # 略
    ]
  ]
```

ローカルでDockerイメージをビルドして起動し、ローカル環境でのAPIの応答を確認します。

```bash
mix phx.gen.release --docker
docker build -t my_awesome_app .
mix compile
docker run --rm -p 4000:8080 -e PORT=8080 -e SECRET_KEY_BASE=$(mix phx.gen.secret) -e PHX_HOST=localhost my_awesome_app
curl -i http://localhost:4000/health
```

なお、Apple SiliconのMacで `linux/amd64` のイメージをビルドするとBEAMが起動できずに失敗するので、Dockerfileのビルド用ステージに以下の1行を足しておきます。

```dockerfile
ENV ERL_FLAGS="+JMsingle true"
```

## Cloud Runの準備

Dockerのイメージを置くためのリポジトリとしてArtifact Registryを作成します。

```bash
gcloud services enable artifactregistry.googleapis.com
gcloud artifacts repositories create my-awesome-app --repository-format=docker --location=asia-northeast1 --description="My Awesome App API server images"
gcloud auth configure-docker asia-northeast1-docker.pkg.dev
```

本番用のシークレットをSecret Managerに登録します。先に `mix compile` しておかないと、`mix phx.gen.secret` の実行時に標準出力されるコンパイル時のメッセージがシークレットに混入します。

```bash
gcloud services enable secretmanager.googleapis.com
mix compile
mix phx.gen.secret | tr -d '\n' | gcloud secrets create my-awesome-app-secret-key-base --data-file=- --replication-policy=automatic
```

Cloud RunはデフォルトではCompute Engineのデフォルトサービスアカウントで動きますが、このサービスアカウントは権限が広いです。そのため、権限の狭い専用のサービスアカウントを作って使うようにします。このサービスアカウントにはシークレットの読み取り権限だけ与えればOKです。

```bash
gcloud iam service-accounts create my-awesome-app-runner --display-name="My Awesome App Cloud Run"
gcloud secrets add-iam-policy-binding my-awesome-app-secret-key-base --member="serviceAccount:my-awesome-app-runner@<PROJECT_ID>.iam.gserviceaccount.com" --role="roles/secretmanager.secretAccessor"
```

Cloud Runで動くのは `linux/amd64` なので、`--platform linux/amd64` を指定してビルドします。イメージにはGitのコミットハッシュをタグ付けしておきます。

```bash
export IMAGE=asia-northeast1-docker.pkg.dev/<PROJECT_ID>/my-awesome-app/my-awesome-app:$(git rev-parse --short HEAD)
docker buildx build --platform linux/amd64 -t "$IMAGE" --push .
```

ビルドできたらデプロイします。`--service-account` オプションで先ほど作成したサービスアカウントを指定してます。`--set-secrets` で、先ほど作成したシークレットを環境変数 `SECRET_KEY_BASE` として渡しています。`--set-env-vars` でホストを指定していますが、Cloud RunのURLは `https://<サービス名>-<プロジェクト番号>.<リージョン>.run.app` となるのでデプロイを待たずとも決定できます。`--startup-probe=httpGet.path` で `/health` を渡してますが、これが前述のプローブの設定です。

```bash
gcloud run deploy my-awesome-app \
  --image="$IMAGE" \
  --region=asia-northeast1 \
  --service-account=my-awesome-app-runner@<PROJECT_ID>.iam.gserviceaccount.com \
  --set-secrets=SECRET_KEY_BASE=my-awesome-app-secret-key-base:latest \
  --set-env-vars=PHX_HOST=my-awesome-app-<PROJECT_NUMBER>.asia-northeast1.run.app \
  --allow-unauthenticated \
  --cpu=1 \
  --memory=512Mi \
  --min-instances=0 \
  --max-instances=2 \
  --startup-probe=httpGet.path=/health \
  --liveness-probe=httpGet.path=/health
```

curlでヘルスチェックを叩いて、`{"status":"ok"}` が返ってくればOKです。

```bash
curl -i https://my-awesome-app-<PROJECT_NUMBER>.asia-northeast1.run.app/health
```
