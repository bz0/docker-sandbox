## docker-compose
volumes：ホスト・コンテナ間でファイルを共有する。ホストから扱いやすくするために利用する

## slaveコンテナを作る

docker-composeでmasterコンテナを作る

```
# docker-compose up -d
```

masterがslaveにsshで接続できるようにMasterコンテナのSSH Keyを作成します

```
# docker container exec -it master ssh-keygen -t rsa -C ""
Generating public/private rsa key pair.
Enter file in which to save the key (/var/jenkins_home/.ssh/id_rsa):
Created directory '/var/jenkins_home/.ssh'.
Enter passphrase (empty for no passphrase):
Enter same passphrase again:
Your identification has been saved in /var/jenkins_home/.ssh/id_rsa.
Your public key has been saved in /var/jenkins_home/.ssh/id_rsa.pub.
The key fingerprint is:
SHA256:cndJ58lVw44n/pkLmzoi/bVI9LRjYh9G6XHYSidKQ5I
The key's randomart image is:
+---[RSA 2048]----+
|              ...|
|          .    .o|
|         E .. + .|
|          o. B+= |
|      . S .++O*+ |
|       o .o.O.B  |
|        .  = %. o|
|       . oo.B B+ |
|        . o+o= ..|
+----[SHA256]-----+
```

## SSH接続の準備

- SSH接続するslave用途のDockerイメージ：jenkinsci/ssh-slave
- 環境変数「JENKINS_SLAVE_SSH_PUBKEY」を設定：MasterからのSSH接続が許可されます
- エラー：ERROR: failed to register layer: ApplyLayer exit status 1 stdout:  stderr: write /usr/share/mime/globs2: no space left on device
	- https://github.com/moby/moby/issues/29508
	- スペースがないとの事なので「docker rmi -f $(docker images -q)」で既存のdockerイメージをとりあえず削除して再実行したら解決した