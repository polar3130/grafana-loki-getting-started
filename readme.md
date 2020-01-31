# Grafana Loki ではじめる Kubernetes ロギング ハンズオン

## Open this repo in Google Cloud Shell

[![Open in Cloud Shell](https://gstatic.com/cloudssh/images/open-btn.svg)](https://ssh.cloud.google.com/cloudshell/open?git_repo=https://github.com/polar3130/grafana-loki-getting-started.git&page=editor&tutorial=tutorial.md)

マニュアルでリポジトリを clone した場合や、途中から再開する場合は、次のコマンドでチュートリアルを起動してください

```
cloudshell launch-tutorial -d tutorial.md
```

## チュートリアル目次

0. 環境準備
1. Loki スタックの概要
2. Explore / LogQL 入門
3. Aggregatable Events の可視化
4. 片付け 

## 投影資料

https://speakerdeck.com/polar3130/grafana-loki-getting-started

## notes

- 各自個別のプロジェクトをご用意ください（複数人でひとつのプロジェクトを共用すると、手順内でリソースを作成する際に名前が重複してしまうため）
- 同一端末で複数の Google アカウントを利用している場合は、ブラウザのシークレットウィンドウやゲストモードを使って Cloud Shell を起動してください

## Author

Masahiko Utsunomiya

@polar3130
