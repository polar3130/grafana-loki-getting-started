# Tutorial

Grafana Loki を使って Kubernetes のクラスタレベルロギングを学ぶ、ハンズオンワークショップのチュートリアルです

## Agenda

0. 環境準備
1. Loki スタックの概要
2. Explore / LogQL 入門
3. Aggregatable Events の可視化
4. 片付け 

## 0 環境準備

この章では、GKE クラスタを作成し、Grafana Loki をベースとしたロギングスタックとサンプルアプリケーションをデプロイします

チュートリアルを開始する前にGCPのプロジェクトを新規に作成してください

## 0.1 プロジェクトの作成/確認

**ハンズオンに使用するプロジェクトを未だ作成していない場合**

GCP プロジェクトを新規作成します

[\[リソースの管理\]ページに移動](https://console.cloud.google.com/cloud-resource-manager?hl=ja)  

---

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
gcloud services enable cloudapis.googleapis.com container.googleapis.com
```

## 0.5 GKE クラスタの作成

GKE クラスタの作成をリクエストします

（Cloud Shell のコントロールはクラスタの作成完了を待たずにユーザに戻ります）

```bash
gcloud container clusters create loki-handson-cluster --num-nodes 2 --zone $COMPUTE_ZONE --async
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

---

**Helm のバージョンを変更したくない(Helm v2 を利用したい)場合は、上記に代わり、以下の手順を実行してください**

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
helm install wordpress --namespace app stable/wordpress --set service.type=ClusterIP
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

## 1 Loki スタックの概要

この章では、Grafana Loki をベースとしたロギングスタックの構成と、各コンポーネントの関係性を確認します

## 1.1 Loki スタックの状態確認

展開したリソースの状態を確認します

```bash
kubectl get all -n loki
```

## 1.2 Grafana の Web コンソールへのポートフォワード

Grafana にログインするためのパスワードを確認します

```bash
kubectl get secret loki-stack-grafana -n loki -o jsonpath="{.data.admin-password}" | base64 -d
```

.

Grafana の Web コンソールにアクセスするためのポートフォワードを行います

```bash
kubectl port-forward -n loki $(kubectl get pod -n loki --selector="app=grafana,release=loki-stack" --output jsonpath='{.items[0].metadata.name}') 8080:3000
```

## 1.3 Grafana へのログイン

ここからは Grafana の Web コンソールを利用します

Cloud Shell の画面右上にある **\[ウェブでプレビュー\]** のアイコンをクリックし、サブメニューの **\[ポート 8080 でプレビュー\]** をクリックします

.

ログイン画面で以下のとおりに入力してログインします

- username: **admin**
- password: **\<手順 1.2 で確認したパスワード\>**

## 1.4 Datasource の設定

Grafana のコンソール左端のメニューから **Configuration(歯車のマーク)** をクリックし、サブメニューの **Data Sources** をクリックします

Data Souces に Loki が追加されていることを確認します

## 2 Explore / LogQL 入門

この章では、Grafana の Explore を利用したアドホックなログ検索を行います

## 2.1 はじめての LogQL

Grafana のコンソール左端のメニューから **Explore(コンパスのマーク)** をクリックします

.

Explore の画面が表示されたら、画面上部のテキストボックス（**Log labels ▼** の右側）に次のクエリを入力し、画面右上の **Run Query** ボタンをクリックします

```
{namespace="app"}
```

## 2.2 Explore におけるログ検索

手順 2.1 のログ検索結果をもとに、Explore の機能や表示について説明します

- 定期的なリロード と Live tail
- Severity フィルタ
- 各スイッチの説明

## 2.3 ストリームセレクタとフィルタ式

Grafana の Explore に次のクエリを入力し、**Run Query** ボタンをクリックします

```
{namespace="loki",app="promtail"} |= "wordpress"
```

## 2.4 ラベルベースのインデックス管理

手順 2.3 のログ検索結果をもとに、Loki のデータモデルと LogQL の基礎について説明します

- 非構造化データの取り扱い
- 単行ログと複数行ログ
- ログの抽出上限
- Explore の Detail パネル

## 3 Aggregatable Events の可視化

この章では、ログに基づくメトリクス(Aggregatable Events)を可視化します

## 3.1 可視化するログストリームの確認

Grafana の Explore に次のクエリを入力し、**Run Query** ボタンをクリックします

手順 2.3 と同じストリームセレクタで、フィルタ式を除いたクエリとなっています

```
{namespace="loki",app="promtail"}
```

## 3.2 検索モードの切り替え

Explore の画面上部、Data Source に **Loki** と表示されているプルダウンメニューの右にある **Metrics** のボタンをクリックします

Loki の検索モードがメトリクスに切り替わります

.

クエリのテキストボックスに選択したラベルセットが残っていることを確認し、クエリを以下のように修正します

```
sum(count_over_time({namespace="loki",app="promtail"}[10s])) by (instance)
```

## 3.3 Range Vector セレクタとアグリゲーション

手順 3.2 のメトリクス検索結果をもとに、ログの集約について説明します

- Range Vector セレクタ
- ログアグリゲーション

## Appendix

時間に余裕のある方むけに、追加のハンズオンをご用意しています

以下のコマンドでチュートリアルを起動してください

```bash
cloudshell launch-tutorial -d tutorial-1-2.md
```

チュートリアルを終了する場合はこのセクションをスキップしてください

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