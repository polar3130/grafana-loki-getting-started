# Tutorial

## Agenda

0. 環境準備
1. LogQL 入門
2. Aggregatable Events の可視化
3. 片付け 

# 0. GKE クラスタの作成

GKE クラスタを作成します

## 0.1 Project ID の設定

GCP プロジェクトのプロジェクト ID を表示します

```bash
gcloud projects list
```

今回使用するプロジェクトを Cloud Shell のデフォルトプロジェクトに設定します

```bash
gcloud config set project "プロジェクト名"
```

## 0.2 APIの有効化

GKE クラスタの作成に必要となる API を有効化します

```bash
gcloud services enable \
  cloudapis.googleapis.com \
  container.googleapis.com
```

## 0.3 GKE クラスタの作成

GKE のクラスタを作成します

```bash
gcloud container clusters create loki-handson-cluster --enable-ip-alias --num-nodes=1 --zone=asia-east1-b --async
```

## 0.4 クラスタの起動確認

作成した GKE クラスタのステータスが **RUNNING** になっていることを確認します

```bash
gcloud container clusters list
```

コンソールで確認する場合は以下をクリック

[Display on the Console](https://console.cloud.google.com/kubernetes/list)

## 0.5 Helm のインストール

Kubernetes のパッケージマネージャである Helm をインストールします

```bash
curl https://raw.githubusercontent.com/helm/helm/master/scripts/get-helm-3 | bash
```

## 0.6 namespace の作成

Loki などのロギングスタックをインストールする namespace を作成します

```bash
kubectl create namespace loki-stack
```

## 0.7 Loki スタックのデプロイ

Loki スタック※ を GKE クラスタにデプロイします
※ Grafana, Grafana Loki, Promtail によるロギングスタック

```bash
helm upgrade --install loki-stack --namespace=loki loki/loki-stack
```

