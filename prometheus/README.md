# 概要
KubernetesのmetricsをPrometheusを利用して収集する

* 事前準備
* クラスター構築
* RBAC設定
* コンテナ起動
* namespace設定
* configmap設定
  * Prometheus
  * external-url
* deployment設定
  * Prometheus
* service設定
  * Prometheus
* node-exporter起動


## 1. 事前準備
* GCPのプロジェクトを作成する
* gcloudコマンドが使えるようにしておく
* gcloud auth loginしておく

## 2. クラスター構築
* クラスター作成
```
$ gcloud container clusters create sample-cluster
```

## 3. namespace作成
* monitoring用namespace作成
```
$ kubectl apply -f namespace.yaml
```

## 4. RBAC設定
* 自身のserviceaccountに権限付与
```
$ gcloud info | grep Account
$ kubectl create clusterrolebinding sample-cluster-admin-binding \
  --clusterrole=cluster-admin \
  --user=<gcloud infoで取得したアカウント情報>
```

* serviceaccount作成
```
$ kubectl create serviceaccount prometheus --namespace monitoring
```

* clusterRole作成 / Binding
```
$ kubectl apply -f role.yaml
$ kubectl create clusterrolebinding prometheus --clusterrole=all-reader --serviceaccount=monitoring:prometheus
```

## 5. コンテナ起動
* configmap設定
```
$ kubectl -n monitoring apply -f prometheus-configmap.yaml
$ kubectl -n monitoring apply -f external-url-configmap.yaml
```

* deployment設定
```
$ kubectl -n monitoring apply -f prometheus-deployment.yaml
```

* service設定
```
$ kubectl -n monitoring apply -f prometheus-service.yaml
```

## 6. firewall設定
* prometheusポート解放
VPCネットワーク > ファイアウォールルール > 新規作成
  * 名前： allow-default-prometheus
  * ネットワーク：default
  * ターゲット：ネットワーク上のすべてのインスタンス
  * ソースフィルタ(IP範囲)： `<your client ip>` /32
  * プロトコルとポート： tcp:9090

## 7. 動作確認
* Prometheusが稼働しているノードのGlobalIPを取得する
* `http://<globalip>:30001` へアクセス