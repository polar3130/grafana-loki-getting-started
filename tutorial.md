# Tutorial

「Grafana Loki を使って Kubernetes のクラスタレベルロギングを学ぶハンズオンワークショップ」のチュートリアル

## Agenda

0. 環境準備
1. Loki スタックの概要
2. LogQL 入門
3. Aggregatable Events の可視化
4. 片付け 

## 0. 環境準備

この章では、GKE クラスタを作成し、Grafana Loki をベースとしたロギングスタックとサンプルアプリケーションをデプロイします

チュートリアルを開始する前にGCPのプロジェクトを新規に作成してください

## 0.1 プロジェクトの作成/確認

**ハンズオンに使用するプロジェクトを未だ作成していない場合**

GCP プロジェクトを新規作成します

[\[リソースの管理\]ページに移動](https://console.cloud.google.com/cloud-resource-manager?hl=ja)  

.

**ハンズオンに使用するプロジェクトを作成済みの場合**

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
export PROJECT_ID=$(gcloud config get-value project); echo $PROJECT_ID
```

## 0.3 Zone の設定

今回使用するゾーンを Cloud Shell のデフォルトゾーンに設定します

```bash
gcloud config set compute/zone asia-northeast1-b
```

.

後続のステップで使用するため、利用するゾーンを export しておきます

```bash
export COMPUTE_ZONE=$(gcloud config get-value compute/zone); echo $COMPUTE_ZONE
```

## 0.4 APIの有効化

GKE クラスタの作成に必要となる API を有効化します

（有効化には数分かかる場合があります）

```bash
gcloud services enable \
  cloudapis.googleapis.com \
  container.googleapis.com
```

## 0.5 GKE クラスタの作成

GKE クラスタの作成をリクエストします

（Cloud Shell のコントロールはクラスタの作成完了を待たずにユーザに戻ります）

```bash
gcloud container clusters create loki-handson-cluster --enable-ip-alias --num-nodes 2 --zone $COMPUTE_ZONE --async
```

.

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

Kubernetes のパッケージマネージャである Helm v3 をインストールします

```bash
curl https://raw.githubusercontent.com/helm/helm/master/scripts/get-helm-3 | bash
```

.

**Helm のバージョンを変更したくない(Helm v2 を利用したい)場合は、上記に代わり以下の手順を実行してください**

Tiller のサービスアカウントを作成します

```bash
kubectl -n kube-system create serviceaccount tiller
```

.


Tiller のサービスアカウントにクラスタロールをバインドします

```bash
kubectl create clusterrolebinding tiller --clusterrole cluster-admin --serviceaccount=kube-system:tiller
```

.


Tiller を GKE クラスタにインストールします

```bash
helm init --service-account=tiller
```

## 0.9 namespace の作成

Loki などのロギングスタックを展開するための namespace を作成します

```bash
kubectl create ns loki
```

.

サンプルアプリケーションを展開するための namespace を作成します

```bash
kubectl create ns app
```

## 0.10 リポジトリ設定の追加

Helm に stable のリポジトリを登録します

```bash
helm repo add stable https://kubernetes-charts.storage.googleapis.com/ 
```

.

Helm に Grafana Loki のリポジトリを登録します

```bash
helm repo add loki https://grafana.github.io/loki/charts/
```

.

リポジトリをアップデートします

```bash
helm repo update
```

## 0.11 サンプルアプリケーションのインストール

サンプルアプリケーションの Wordpress を GKE クラスタにインストールします

```bash
helm install wordpress --namespace app stable/wordpress
```

## 0.12 環境の確認

展開したサンプルアプリケーションの状態を確認します

```bash
kubectl get all --namespace app
```

## 0.13 Loki スタックのインストール

Grafana Loki をベースとしたロギングスタック を GKE クラスタにインストールします

```bash
helm install loki-stack --namespace loki loki/loki-stack --set grafana.enabled=true,grafana.image.tag=6.6.0-beta1
```

.

イントール結果の確認は次章で行います

## 1.0 Loki スタックの概要

この章では Grafana Loki をベースとした
Loki スタックの構成

## 1.1 Loki スタックの状態確認

展開したロギングスタックの状態を確認します

```bash
kubectl get all -n loki
```

## 1.2

Grafana にログインするためのパスワードを確認します

```bash
kubectl get secret loki-stack-grafana -n loki -o jsonpath="{.data.admin-password}" | base64 -d
```

.

Grafana の Web コンソールにアクセスするためのポートフォワードを行います

```bash
kubectl port-forward -n loki $(kubectl get pod -n loki --selector="app=grafana,release=loki-stack" --output jsonpath='{.items[0].metadata.name}') 8080:3000
```

## 1.x

ここからは Grafana の Web コンソールを利用します

Cloud Shell の画面右上にある **\[ウェブでプレビュー\]** のアイコンをクリックし、サブメニューの **\[ポート 8080 でプレビュー\]** をクリックします

.

ログイン画面が表示されたら、
以下の
- username:
- 