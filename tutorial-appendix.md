## Appendix

この追加のチュートリアルでは、LogCLI という Grafana Loki のクライアントツールを用いて、CLI ベースでの Kubernetes クラスタレベルロギングを行います

## a.1 テンポラリコンテナの作成

LogCLI を使うためのコンテナを作成し、ログインします

```bash
kubectl run tmp-shell -n loki --rm -i --tty --image centos -- /bin/bash
```

## a.2 インストールの準備

必要なコマンドをインストールします

```bash
yum install unzip -y
```

## a.3 LogCLI のインストール

LogCLI のバイナリを公式リポジトリからダウンロードします

```bash
curl -O -L "https://github.com/grafana/loki/releases/download/v1.3.0/logcli-linux-amd64.zip"
```

.

zip ファイルを解凍します

```bash
unzip "logcli-linux-amd64.zip"
```

.

/usr/local/bin/ 配下に実行ファイルを移動します

```bash
mv ./logcli-linux-amd64 /usr/local/bin/logcli
```

.

実行権限を付与します

```bash
chmod a+x /usr/local/bin/logcli
```

## a.4 Loki サーバへの接続

LogCLI が接続する Loki サーバを設定します

```bash
export LOKI_ADDR=http://loki-stack:3100
```

.

LogCLI から Loki サーバにラベル情報を問い合わせます

```bash
logcli labels
```

## a.5 Loki サーバへのクエリ

Loki サーバが収集したログに付与されている "job" ラベルの値セットを取得します

```bash
logcli labels job
```

.

サンプルアプリケーションを実行している名前空間のログを取得します

```bash
logcli query '{namespace="app"}'
```

## a.6 テンポラリコンテナの終了

LogCLI を実行しているコンテナから離脱します

```bash
exit
```

## 4 片付け

課金対象のリソースを削除します

.

プロジェクトごと削除する場合はこちら

[\[リソースの管理\]ページに移動](https://console.cloud.google.com/cloud-resource-manager?hl=ja)  

## 4.1 GKE クラスタの削除

GKE クラスタを削除します

```bash
gcloud container clusters delete loki-handson-cluster --zone $COMPUTE_ZONE --async
```

## 4.2 クラスタの停止確認

GKE クラスタがリストから削除されたことを確認します

```bash
gcloud container clusters list
```

.
  
コンソールで確認する場合は以下をクリック

[Display on the Console](https://console.cloud.google.com/kubernetes/list)

## 終了

おつかれさまでした！

このチュートリアルでは以下を行いました：

- GKE で Grafana Loki のロギングスタックを構成する
- Loki を使った Kubernetes のクラスタレベルロギング
- ログに基づくメトリクス(Aggregatable Events)の可視化

（Appendix）

- LogCLI による CLI ベースの クラスタレベルロギング
