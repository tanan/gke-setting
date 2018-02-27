# 概要
kubernetesのmetricsをPrometheusを利用して収集する


* 事前準備
* k8s構築
* コンテナ起動
* node-exporter起動
* namespace設定
* configmap設定
  * Prometheus
  * Prometheus-rules
  * external-url
* deployment設定
  * Prometheus
* service設定
  * Prometheus
* Ingress設定


## 1. 事前準備
* GCPのプロジェクトを作成する
* gcloudコマンドが使えるようにしておく
* gcloud auth loginしておく

## 2. クラスター構築
* クラスター作成
```
$ gcloud container clusters create sample-cluster
```

* UI起動
  下記実施後、ブラウザからlocalhost:8001/uiにアクセスする
```
$ gcloud container clusters get-credentials sample-cluster
$ kubectl proxy
```

## 3. コンテナ起動
* namespace作成
```
$ kubectl apply -f namespace.yaml
```

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

* ingress設定
```
$ kubectl -n monitoring apply -f ingress.yaml
```

