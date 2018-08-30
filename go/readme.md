# 
## 1.Dockerイメージのビルド

- docker image build Dockerイメージを作成する
- 「-t」オプション：任意のイメージ名を指定できる（必ず指定する）

```
# docker image build -t example/echo:latest .
```

```
# docker image build -t example/echo:latest .
Sending build context to Docker daemon  4.096kB
Step 1/4 : FROM golang:1.9
1.9: Pulling from library/golang
55cbf04beb70: Download complete
1607093a898c: Download complete
9a8ea045c926: Download complete
d4eee24d4dac: Downloading [==================================================>]  50.06MB/50.06MB
9c35c9787a2f: Downloading [==================================================>]  57.59MB/57.59MB
8b376bbb244f: Downloading [==================================================>]  118.3MB/118.3MB
0d4eafcc732a: Download complete
186b06a99029: Download complete
write /var/lib/docker/tmp/GetImageBlob022449396: no space left on device
[root@localhost go]# write /var/lib/docker/tmp/GetImageBlob022449396: no space left on device
```

「no space left on device」というエラーが出た。
https://qiita.com/f-katkit/items/f4b4c5649b97258de5cb

コンテナイメージを全て消す
```
docker rmi -f $(docker images -q)
```

## 再度実行する

```
# docker image build -t example/echo:latest .
Sending build context to Docker daemon   5.12kB
Step 1/4 : FROM golang:1.9
1.9: Pulling from library/golang
55cbf04beb70: Pull complete
1607093a898c: Pull complete
9a8ea045c926: Pull complete
d4eee24d4dac: Pull complete
9c35c9787a2f: Pull complete
8b376bbb244f: Pull complete
0d4eafcc732a: Pull complete
186b06a99029: Pull complete
Digest: sha256:8b5968585131604a92af02f5690713efadf029cc8dad53f79280b87a80eb1354
Status: Downloaded newer image for golang:1.9
 ---> ef89ef5c42a9
Step 2/4 : RUN mkdir /echo
 ---> Running in 1398c753c1ea
Removing intermediate container 1398c753c1ea
 ---> bd48891453a9
Step 3/4 : COPY main.go /echo
 ---> 578c3f0b5bf4
Step 4/4 : CMD ["go", "run", "/echo/main.go"]
 ---> Running in 4efd12447a68
Removing intermediate container 4efd12447a68
 ---> 7654cbd5cb1c
Successfully built 7654cbd5cb1c
Successfully tagged example/echo:latest
```


## 2.作成されたイメージの確認

```
# docker container ls
```

## 3.Dockerコンテナの実行

Dockerコンテナをバックグラウンドで実行します

```
# docker container run -d example/echo:latest
```

ローカル環境で8080ポートに向けてcurlでGETリクエストを送信しても
コンテナの外からはアクセスできない。

```
# curl http://192.168.33.12:8080
curl: (7) Failed to connect to 192.168.33.12 port 8080: Connection refused
```

一度コンテナを停止する

```
# docker container stop $(docker container ls --filter "ancestor=example/echo" -q)
```

ホスト側の9000番ポートをコンテナ側の8080ポートにポートフォワーディングさせる設定も入れて
Dockerコンテナを再度実行します。

```
# docker container run -d -p 9000:8080 example/echo:latest
```

下記、「Hello Docker!!」が出力されました。

```
# curl http://192.168.33.12:9000/
Hello Docker!!
```

