# busybox コンテナ起動

```
# docker image build -t example/mysql-data:latest .
Sending build context to Docker daemon  2.048kB
Step 1/3 : FROM busybox
latest: Pulling from library/busybox
8c5a7da1afbc: Pull complete
Digest: sha256:cb63aa0641a885f54de20f61d152187419e8f6b159ed11a251a09d115fdff9bd
Status: Downloaded newer image for busybox:latest
 ---> e1ddd7948a1c
Step 2/3 : VOLUME /var/lib/mysql
 ---> Running in 7fbfc6e153d4
Removing intermediate container 7fbfc6e153d4
 ---> 0b25e4b6b0c5
Step 3/3 : CMD ["bin/true"]
 ---> Running in f77a1e728e8a
Removing intermediate container f77a1e728e8a
 ---> dbbdb76c2624
Successfully built dbbdb76c2624
Successfully tagged example/mysql-data:latest
```

## Data Volume コンテナを実行

```
# docker container run -d --name mysql-data example/mysql-data:latest
0b39becbc6a2825c424359a6c25e93e3c034361aca65dd01251ad6430858fb0c
```

## MYSQLコンテナを実行

Data VolumeコンテナをMySQLコンテナにマウントするので、MySQLコンテナに/var/lib/mysqlのデータは保存されなくなる

```
# docker container run -d --rm --name mysql \
> -e "MYSQL_ALLOW_EMPTY_PASSWORD=yes" \
> -e "MYSQL_DATABASE=volume_test" \
> -e "MYSQL_USER=example" \
> -e "MYSQL_PASSWORD=example" \
> --volumes-from mysql-data \
> mysql:5.7
Unable to find image 'mysql:5.7' locally
5.7: Pulling from library/mysql
802b00ed6f79: Pull complete
30f19a05b898: Pull complete
3e43303be5e9: Pull complete
94b281824ae2: Pull complete
51eb397095b1: Pull complete
54567da6fdf0: Pull complete
bc57ddb85cce: Pull complete
c7c0a9c25d8a: Pull complete
cce6c47ac3fc: Pull complete
499b9c7376c8: Pull complete
6c5e08e005ea: Pull complete
Digest: sha256:1d8f471c7e2929ee1e2bfbc1d16fc8afccd2e070afed24805487e726ce601a6d
Status: Downloaded newer image for mysql:5.7
8c7b47af6710d18e047176cbced70ae177d8be85b5ced8aa660787ba85034f51
```

# mysqlにデータを投入

```
# docker container exec -it mysql mysql -u root -p volume_test
Enter password:
Welcome to the MySQL monitor.  Commands end with ; or \g.
Your MySQL connection id is 4
Server version: 5.7.23 MySQL Community Server (GPL)

Copyright (c) 2000, 2018, Oracle and/or its affiliates. All rights reserved.

Oracle is a registered trademark of Oracle Corporation and/or its
affiliates. Other names may be trademarks of their respective
owners.

Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.

mysql> CREATE TABLE user(
    -> id int PRIMARY KEY AUTO_INCREMENT,
    -> name VARCHAR(255)
    -> ) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE utf8mb4_unicode_ci;
Query OK, 0 rows affected (0.53 sec)

mysql> INSERT INTO user (name) VALUES ('gihyo'), ('docker'), ('Solomon Hykes');
Query OK, 3 rows affected (0.14 sec)
Records: 3  Duplicates: 0  Warnings: 0
```

## MySQLコンテナを停止

```
# docker container stop mysql
mysql
```

## 再度MySQLコンテナを起動

```
# docker container run -d --rm --name mysql \
> -e "MYSQL_ALLOW_EMPTY_PASSWORD=yes" \
> -e "MYSQL_DATABASE=volume_test" \
> -e "MYSQL_USER=example" \
> -e "MYSQL_PASSWORD=example" \
> --volumes-from mysql-data \
> mysql:5.7
9b5f614ea1c8080512f97691bc4c8a670b4bb4caaa15e5aec0375f376e66f975
```

MySQLのデータが残っていることが分かる

```
# docker container exec -it mysql mysql -u root -p volume_test
Enter password:
Reading table information for completion of table and column names
You can turn off this feature to get a quicker startup with -A

Welcome to the MySQL monitor.  Commands end with ; or \g.
Your MySQL connection id is 2
Server version: 5.7.23 MySQL Community Server (GPL)

Copyright (c) 2000, 2018, Oracle and/or its affiliates. All rights reserved.

Oracle is a registered trademark of Oracle Corporation and/or its
affiliates. Other names may be trademarks of their respective
owners.

Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.

mysql> SELECT * FROM user;
+----+---------------+
| id | name          |
+----+---------------+
|  1 | gihyo         |
|  2 | docker        |
|  3 | Solomon Hykes |
+----+---------------+
3 rows in set (0.04 sec)
```