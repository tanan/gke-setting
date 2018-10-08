# 概要

Kubernetesにistioをインストールして、blue/green Deploymentを実現する

* 事前準備
* クラスター構築
* RBAC設定
* コンテナ起動

## 1. 事前準備

* GCPのプロジェクトを作成する
* gcloudコマンドインストール
* gcloud auth loginする

## 2. クラスター構築

* クラスター作成

```
$ gcloud container clusters create istio-cluster \
  --cluster-version="1.10.7-gke.2" \
  --num-nodes=3 \
  --no-enable-legacy-authorization
```

## 3. RBAC設定

* 自身のserviceaccountに権限付与

```
$ kubectl create clusterrolebinding cluster-admin-binding \
  --clusterrole=cluster-admin \
  --user="$(gcloud config get-value core/account)"
```

## 4. Istioインストール

```
$ kubectl apply -f install/kubernetes/istio-demo-auth.yaml
$ kubectl get service -n istio-system
$ kubectl get pods -w -n istio-system
```

## 5. 各種リソースの作成

* namespace

```
$ kubectl apply -f namespace.yaml
```

* deployment

```
$ kubectl apply -f deployment.yaml
```

* service

```
$ kubectl apply -f service.yaml
```

## 6. 各種Istioリソースの作成

* gateway

```
$ kubectl apply -f gateway.yaml
```


* virtual service

```
$ kubectl apply -f virtual-service.yaml
```

* destination rule

```
$ kubectl apply -f destination-rule.yaml
```

## 6. hostsファイル設定

* hostsファイルにサンプルのドメイン名設定(本来はDNSサーバに登録する)

```
kubectl xxxx
$ sudo echo "xxx.xxx.xxx.xxx sample.hoge.com" >> /etc/hosts
$ sudo echo "xxx.xxx.xxx.xxx sample.foo.com" >> /etc/hosts
```

## 7. アクセス確認

```
$ curl -v http://sample.hoge.com
$ curl -v http://sample.foo.com
```