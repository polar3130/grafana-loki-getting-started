# Tutorial

「Grafana Loki を使って Kubernetes のクラスタレベルロギングを学ぶハンズオンワークショップ」のチュートリアル

## Agenda

1. 環境準備
2. LogQL 入門
3. Aggregatable Events の可視化
4. 片付け 

# 1. GKE クラスタの作成

GKE クラスタを作成します

## 1.1 Project ID の設定

GCP プロジェクトのプロジェクト ID を表示します

```bash
gcloud projects list
```


今回使用するプロジェクトを Cloud Shell のデフォルトプロジェクトに設定します

```bash
gcloud config set project <YOUR PROJECT NAME>
```


後続のステップで使用するため、プロジェクト ID を export しておきます

```bash
export PROJECT_ID=$(gcloud config get-value project)
```


export した内容を確認します

```
echo $PROJECT_ID
```

## 1.2 Zone の設定

今回使用するゾーンを Cloud Shell のデフォルトゾーンに設定します

```bash
gcloud config set compute/zone asia-northeast1-b
```


後続のステップで使用するため、利用するゾーン を export しておきます

```bash
export COMPUTE_ZONE=$(gcloud config get-value compute/zone)
echo $COMPUTE_ZONE
```

## 1.2 APIの有効化

GKE クラスタの作成に必要となる API を有効化します
（有効化には数分を要する場合があります）

```bash
gcloud services enable \
  cloudapis.googleapis.com \
  container.googleapis.com
```


有効化が完了してから次へ進んでください

## 1.3 GKE クラスタの作成

GKE クラスタの作成をリクエストします
（Cloud Shell のコントロールはクラスタの作成完了を待たずにユーザに戻ります）

```bash
gcloud container clusters create loki-handson-cluster --enable-ip-alias --num-nodes 1 --zone $COMPUTE_ZONE --async
```

## 1.4 クラスタの起動確認

作成した GKE クラスタのステータスが **RUNNING** になっていることを確認します

```bash
gcloud container clusters list
```

コンソールで確認する場合は以下をクリック

[Display on the Console](https://console.cloud.google.com/kubernetes/list)

## 1.5 Credential の取得

GKE クラスタへ接続するための Credential を取得します

```bash
gcloud container clusters get-credentials loki-handson-cluster --zone $COMPUTE_ZONE --project $PROJECT_ID
```

## 1.6 Helm のインストール

Kubernetes のパッケージマネージャである Helm をインストールします

```bash
curl https://raw.githubusercontent.com/helm/helm/master/scripts/get-helm-3 | bash
```

## 1.7 namespace の作成

Loki などのロギングスタックをインストールする namespace を作成します

```bash
kubectl create ns loki
```

## 1.8

Helm に Grafana Loki のリポジトリを登録します

```bash
helm repo add loki https://grafana.github.io/loki/charts
```


リポジトリをアップデートします

```bash
helm repo update
```

## 1.9 Loki スタックのインストール

Loki スタック※ を GKE クラスタにインストールします
※ Grafana, Grafana Loki, Promtail によるロギングスタック

```bash
helm install loki-stack --namespace loki loki/loki-stack --set grafana.enabled=true --set grafana.sidecar.datasources.enabled=false --set grafana.image.tag=master
```


インストールした Helm のリリースを確認します

```bash
helm ls -n loki
```

## 2.0 Loki スタックの概要

```bash
kubectl get all -n loki
```


```bash
kubectl get secret loki-stack-grafana -n loki -o jsonpath="{.data.admin-password}" | base64 -d
```


```bash
gcloud container clusters get-credentials loki-handson-cluster --zone $COMPUTE_ZONE --project $PROJECT_ID \
 && kubectl port-forward --namespace loki $(kubectl get pod --namespace loki --selector="app=grafana,release=loki-stack" --output jsonpath='{.items[0].metadata.name}') 8080:3000
```