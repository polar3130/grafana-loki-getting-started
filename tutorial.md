# Tutorial

「Grafana Loki を使って Kubernetes のクラスタレベルロギングを学ぶハンズオンワークショップ」のチュートリアル

## Agenda

0. 環境準備
1. Loki スタックの概要
2. LogQL 入門
3. Aggregatable Events の可視化
4. 片付け 

## 0. 環境準備

この章では、GKE クラスタを作成し、Grafana Loki※ をベースとしたロギングスタックとサンプルアプリケーションをデプロイします

チュートリアルを開始する前にGCPのプロジェクトを新規に作成してください

※ 以後、Loki と略記する場合があります

## 0.1 プロジェクトの作成/確認

### ハンズオンに使用するプロジェクトを未だ作成していない場合
GCP プロジェクトを新規作成します

[\[リソースの管理\]ページに移動](https://console.cloud.google.com/cloud-resource-manager?hl=ja)  

### ハンズオンに使用するプロジェクトを作成済みの場合
GCP プロジェクトのプロジェクト ID を表示します

```bash
gcloud projects list
```

## 0.2 Project ID の設定

今回使用するプロジェクトを Cloud Shell のデフォルトプロジェクトに設定します

```bash
gcloud config set project <YOUR-PROJECT-ID>
```  

.

後続のステップで使用するため、プロジェクト ID を export しておきます

```bash
export PROJECT_ID=$(gcloud config get-value project)
```

.

export した内容を確認します

```bash
echo $PROJECT_ID
```

## 0.3 Zone の設定

今回使用するゾーンを Cloud Shell のデフォルトゾーンに設定します

```bash
gcloud config set compute/zone asia-northeast1-b
```

.

後続のステップで使用するため、利用するゾーンを export しておきます

```bash
export COMPUTE_ZONE=$(gcloud config get-value compute/zone)
```

.

export した内容を確認します

```bash
echo $COMPUTE_ZONE
```

## 0.4 APIの有効化

GKE クラスタの作成に必要となる API を有効化します  
（有効化には数分を要する場合があります）

```bash
gcloud services enable \
  cloudapis.googleapis.com \
  container.googleapis.com
```

## 0.5 GKE クラスタの作成

GKE クラスタの作成をリクエストします  
（Cloud Shell のコントロールはクラスタの作成完了を待たずにユーザに戻ります）

```bash
gcloud container clusters create loki-handson-cluster --enable-ip-alias --num-nodes 3 --zone $COMPUTE_ZONE --async
```

<br />  

GKE クラスタのステータスが **PROVISIONING** になっていることを確認します

## 0.6 クラスタの起動確認

作成した GKE クラスタのステータスが **RUNNING** になっていることを確認します

```bash
gcloud container clusters list
```

.
  
コンソールで確認する場合は以下をクリック

[Display on the Console](https://console.cloud.google.com/kubernetes/list)

## 0.7 Credential の取得

GKE クラスタへ接続するための Credential を取得します

```bash
gcloud container clusters get-credentials loki-handson-cluster --zone $COMPUTE_ZONE --project $PROJECT_ID
```

## 0.8 Helm のインストール

Kubernetes のパッケージマネージャである Helm のバージョンを確認します

```bash
helm version
```

  
$ kubectl -n kube-system create serviceaccount tiller
$ kubectl create clusterrolebinding tiller --clusterrole cluster-admin --serviceaccount=kube-system:tiller
$ helm init --service-account=tiller



## 0.9 namespace の作成

Loki などのロギングスタックをインストールする namespace を作成します

```bash
kubectl create ns loki
```

## 0.10

Helm に Grafana Loki のリポジトリを登録します

```bash
helm repo add loki https://grafana.github.io/loki/charts
```

  
リポジトリをアップデートします

```bash
helm repo update
```

## 0.11 Loki スタックのインストール

Loki スタック※ を GKE クラスタにインストールします
※ Grafana, Grafana Loki, Promtail によるロギングスタック

```bash
helm install loki-stack --namespace loki loki/loki-stack --set grafana.enabled=true --set grafana.image.tag=master --set prometheus.enabled=true
```


## 0.x

cd

git clone https://github.com/GoogleCloudPlatform/microservices-demo.git

cd microservices-demo

skaffold run --default-repo=gcr.io/$PROJECT_ID

## 1.0 Loki スタックの概要

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
