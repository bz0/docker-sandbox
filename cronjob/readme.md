
## WARNING

「apt update」するときに出た警告で、びっくりしたけど特に気にしなくていいぽい

```
WARNING: apt does not have a stable CLI interface. Use with caution in scripts.
```

```
# docker image build -t example/cronjob:latest .
Sending build context to Docker daemon  4.096kB
Step 1/7 : FROM ubuntu:16.04
16.04: Pulling from library/ubuntu
3b37166ec614: Pull complete
ba077e1ddb3a: Pull complete
34c83d2bc656: Pull complete
84b69b6e4743: Pull complete
0f72e97e1f61: Pull complete
Digest: sha256:a218d8dacc99e49cbdc5862d5c6ee105eb40d4e87bec2e4dac37361c30171ce0
Status: Downloaded newer image for ubuntu:16.04
 ---> 52b10959e8aa
Step 2/7 : RUN apt update
 ---> Running in eb725a6913d5

WARNING: apt does not have a stable CLI interface. Use with caution in scripts.

Get:1 http://security.ubuntu.com/ubuntu xenial-security InRelease [107 kB]
Get:2 http://archive.ubuntu.com/ubuntu xenial InRelease [247 kB]
Get:3 http://security.ubuntu.com/ubuntu xenial-security/universe Sources [89.7 kB]
Get:4 http://archive.ubuntu.com/ubuntu xenial-updates InRelease [109 kB]
Get:5 http://archive.ubuntu.com/ubuntu xenial-backports InRelease [107 kB]
Get:6 http://archive.ubuntu.com/ubuntu xenial/universe Sources [9802 kB]
Get:7 http://security.ubuntu.com/ubuntu xenial-security/main amd64 Packages [702 kB]
Get:8 http://security.ubuntu.com/ubuntu xenial-security/restricted amd64 Packages [12.7 kB]
Get:9 http://security.ubuntu.com/ubuntu xenial-security/universe amd64 Packages [467 kB]
Get:10 http://security.ubuntu.com/ubuntu xenial-security/multiverse amd64 Packages [3748 B]
Get:11 http://archive.ubuntu.com/ubuntu xenial/main amd64 Packages [1558 kB]
Get:12 http://archive.ubuntu.com/ubuntu xenial/restricted amd64 Packages [14.1 kB]
Get:13 http://archive.ubuntu.com/ubuntu xenial/universe amd64 Packages [9827 kB]
Get:14 http://archive.ubuntu.com/ubuntu xenial/multiverse amd64 Packages [176 kB]
Get:15 http://archive.ubuntu.com/ubuntu xenial-updates/universe Sources [276 kB]
Get:16 http://archive.ubuntu.com/ubuntu xenial-updates/main amd64 Packages [1087 kB]
Get:17 http://archive.ubuntu.com/ubuntu xenial-updates/restricted amd64 Packages [13.1 kB]
Get:18 http://archive.ubuntu.com/ubuntu xenial-updates/universe amd64 Packages [877 kB]
Get:19 http://archive.ubuntu.com/ubuntu xenial-updates/multiverse amd64 Packages [18.8 kB]
Get:20 http://archive.ubuntu.com/ubuntu xenial-backports/main amd64 Packages [7343 B]
Get:21 http://archive.ubuntu.com/ubuntu xenial-backports/universe amd64 Packages [8086 B]
Fetched 25.5 MB in 33s (771 kB/s)
Reading package lists...
Building dependency tree...
Reading state information...
All packages are up to date.
Removing intermediate container eb725a6913d5
 ---> a72e3ed9e507
Step 3/7 : RUN apt install -y cron
 ---> Running in 5a259e9ef8c0

WARNING: apt does not have a stable CLI interface. Use with caution in scripts.

Reading package lists...
Building dependency tree...
Reading state information...
Suggested packages:
  anacron logrotate checksecurity exim4 | postfix | mail-transport-agent
The following NEW packages will be installed:
  cron
0 upgraded, 1 newly installed, 0 to remove and 0 not upgraded.
Need to get 68.4 kB of archives.
After this operation, 249 kB of additional disk space will be used.
Get:1 http://archive.ubuntu.com/ubuntu xenial/main amd64 cron amd64 3.0pl1-128ubuntu2 [68.4 kB]
debconf: delaying package configuration, since apt-utils is not installed
Fetched 68.4 kB in 1s (42.1 kB/s)
Selecting previously unselected package cron.
(Reading database ... 4768 files and directories currently installed.)
Preparing to unpack .../cron_3.0pl1-128ubuntu2_amd64.deb ...
Unpacking cron (3.0pl1-128ubuntu2) ...
Processing triggers for systemd (229-4ubuntu21.4) ...
Setting up cron (3.0pl1-128ubuntu2) ...
Adding group `crontab' (GID 106) ...
Done.
update-rc.d: warning: start and stop actions are no longer supported; falling back to defaults
update-rc.d: warning: stop runlevel arguments (1) do not match cron Default-Stop values (none)
invoke-rc.d: could not determine current runlevel
invoke-rc.d: policy-rc.d denied execution of start.
Processing triggers for systemd (229-4ubuntu21.4) ...
Removing intermediate container 5a259e9ef8c0
 ---> a5442eeeea2d
Step 4/7 : COPY task.sh /usr/local/bin/
 ---> 83ad94bd1916
Step 5/7 : COPY cron-example /etc/cron.d/
 ---> 651749a43a91
Step 6/7 : RUN chmod 0644 /etc/cron.d/cron-example
 ---> Running in b447e0dd4414
Removing intermediate container b447e0dd4414
 ---> 63ad4fd1a34f
Step 7/7 : CMD ["cron", "-f"]
 ---> Running in 4db7d85535a3
Removing intermediate container 4db7d85535a3
 ---> 6aaf4301897b
Successfully built 6aaf4301897b
Successfully tagged example/cronjob:latest
```

## イメージを実行

```
# docker container run -d --rm --name cronjob example/cronjob:latest
805c511903b18f9a866ab7dcc1c95c56de3118cd63a526e4613f9135d64a13a6
```

## エラー

1分ごとに実行されるので、１分待って下記実行したが、「/var/log/cron.log」ファイルが生成されていませんでした。

```
# docker container exec -it cronjob tail -f /var/log/cron.log
tail: cannot open '/var/log/cron.log' for reading: No such file or directory
tail: no files remaining
```

cronjobのコンテナに入ります

```
# docker exec -it cronjob bash
```

実際にtask.shを実行してみるとエラーが出ました

```
# /usr/local/bin/task.sh
bash: /usr/local/bin/task.sh: /bin/sh^M: bad interpreter: No such file or directory
```


改行コードが「\r\n」で保存されてしまったためのようなので「\n」にします
http://totech.hateblo.jp/entry/2014/03/19/174129

```
# sed -i 's/\r//' /usr/local/bin/task.sh
```

今度はうまく実行されました！

```
# docker container exec -it cronjob tail -f /var/log/cron.log
[Wed Sep  5 15:01:22 UTC 2018] Hello!
```