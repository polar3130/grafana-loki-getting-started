## Appendix

この追加のチュートリアルでは、LogCLI という Grafana Loki クライアントを用いて、CLI ベースでの Kubernetes クラスタレベルロギングを行います

## a.1 

todo

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
