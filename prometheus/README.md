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
  * alertmanager
  * alertmanager-templates
* deployment設定
  * Prometheus
  * alertmanager
* service設定
  * Prometheus
  * alertmanager
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
$ kubectl proxy
```
