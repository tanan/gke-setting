# gke-setting

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

## 3. wordpress + MySQL環境構築
### 永続化用ディスク作成
```
$ gcloud compute disks create --size 200GB mysql-disk
$ gcloud compute disks create --size 200GB wordpress-disk
```

### secret作成
* mysqlのパスワードをsecretに登録（YOUR_PASSWORDは各自のパスワードを設定）
```
$ kubectl create secret generic mysql --from-literal=password=<YOUR_PASSWORD>
```

### サービス作成
* 当gitリポジトリをclone
```
$ git clone https://github.com/tanan/gke-setting.git
```
* mysql, wordpressサービス作成
```
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
```
http://<EXTERNAL-IP>/
```

## 4. fluentdイメージ作成
* GCPのgitリポジトリをclone
```
$ git clone https://github.com/GoogleCloudPlatform/k8s-stackdriver.git
```
* Gemfile編集
  * https://gist.github.com/tanan/73bf196f94b0639c35e7a86928995813 の通り修正
```
$ cd k8s-stackdriver/fluentd-gcp-image
$ vi Gemfile
```
* docker build & push
```
$ docker build . --tag <image_name>
$ docker push <image_name>
```


## 5. BigQuery設定
* BigQueryのWeb管理画面から下記を作成する
  * dataset : kubernetes
  * table : logs
  * schema : time(timestamp), log(string), stream(string)

## 6. ConfigMap設定
* configmap.yaml編集
  * matchディレクティブ内のjson_keyにprivate_key、client_emailを設定
  * matchディレクティブ内のprojectにGCPのproject名を設定

```
$ cd gke-setting/fluentd-gcp/k8s-cluster
$ vi configmap.yaml
```
* yaml適用
```
$ kubectl apply -f configmap.yaml
```

## 7. DaemonSet設定
* daemonset.yaml編集
  * 25行目のimageに手順「4. fluentdイメージ作成」で作成したfluentdのイメージ名を設定
  ```
  $ vi daemonset.yaml
  ```
* yaml適用
```
$ kubectl apply -f daemonset.yaml
```

## 8. 動作確認
* wordpressのログがBigQueryで取得できるか確認
