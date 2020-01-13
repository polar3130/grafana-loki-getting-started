# Grafana Loki Getting Started

## Agenda

1. GKE クラスタの作成
1. Loki スタックの作成
1. アプリケーションのデプロイ
1. アプリケーションのアップデート
1. 2つ目のアプリケーションのデプロイ

# 1. GKE クラスターの作成

GKEのクラスターを作成します。


## 1.1 ProjectID の指定

```bash
gcloud config set project "自身のプロジェクト名"
```

- ※ 自身のプロジェクト名を指定してください
- ※ 途中でやり直す場合はこちらを確認ください

```bash
export PROJECT_ID=$(gcloud config get-value project)
```

プロジェクトの確認

```bash
echo $PROJECT_ID
```

## 1.2 必要なAPIの有効化

必要になるAPIを有効化します。

```bash
gcloud services enable \
  cloudapis.googleapis.com \
  container.googleapis.com \
  containerregistry.googleapis.com \
  cloudbuild.googleapis.com
```

## 1.3 GKE クラスタの作成

GKEのクラスターを作成します。ハンズオンなのでノードは1台にしてあります。

```bash
gcloud container clusters create my-hands-on-cluster --enable-ip-alias --num-nodes=1 --zone=asia-east1-b --async
```

## 1.4 クラスターの確認

作成されてたクラスターを確認します。

```bash
gcloud container clusters list
```

[GKEのコンソール](https://console.cloud.google.com/kubernetes/list)でも確認することができます。

しばらくすると、STATUSがRUNNINGになります。



curl https://raw.githubusercontent.com/helm/helm/master/scripts/get-helm-3 | bash