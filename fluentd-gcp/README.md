# gke-setting

## configmap.yaml
* edit a json_key("private_key", "client_email") to your own key.
* edit a project to your own project.

## daemonset.yaml
* edit an image name to your own image.

## 事前準備
* GCPのプロジェクトを作成する
* gcloudコマンドが使えるようにしておく
* gcloud auth loginしておく

## クラスター構築
* クラスター作成
```
$ gcloud container clusters create sample-cluster
```

* UI起動
  下記実施後、ブラウザからlocalhost:8001/uiにアクセスする
```
$ kubectl proxy
```

## wordpress + MySQL環境構築
### 永続化用ディスク作成
```
$ gcloud compute disks create --size 200GB mysql-disk
$ gcloud compute disks create --size 200GB wordpress-disk
```

### secret作成
```
$ kubectl create secret generic mysql --from-literal=password=YOUR_PASSWORD
```

### サービス作成
```
$ git clone https://github.com/tanan/gke-setting.git
$ cd gke-setting/fluentd-gcp/container
$ kubectl create -f mysql.yaml
$ kubectl get pod -l app=mysql
NAME                     READY     STATUS              RESTARTS   AGE
mysql-3368603707-nhrpj   0/1       ContainerCreating   0          9s
$ kubectl create -f mysql-service.yaml
$ kubectl get service mysql
NAME      TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)    AGE
mysql     ClusterIP   10.55.242.142   <none>        3306/TCP   15s
$ kubectl create -f wordpress.yaml
$ kubectl get pod -l app=wordpress
NAME                         READY     STATUS              RESTARTS   AGE
wordpress-3479901767-c7q21   0/1       ContainerCreating   0          5s
$ kubectl create -f wordpress-service.yaml
$ kubectl get svc -l app=wordpress
NAME        TYPE           CLUSTER-IP      EXTERNAL-IP     PORT(S)        AGE
wordpress   LoadBalancer   10.55.250.244   <EXTERNAL-IP>   80:30403/TCP   1m
```
### wordpress起動確認
* 下記にアクセスして、wordpressが起動するか確認する
http://<EXTERNAL-IP>/

### fluentdイメージ作成
