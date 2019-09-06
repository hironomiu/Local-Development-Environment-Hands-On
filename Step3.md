# Step3
Step2で構築したLAMP環境のMySQLをDockerで置き換えます

## Docker&Docker-compose
MySQLをdockerで利用するため、dockerのインストール、起動を行いましょう

インストール

```
# yum install -y docker
```

起動と自動起動の設定

```
# systemctl start docker.service
# systemctl enable docker.service
```

バージョンの確認

```
# docker version
Client:
 Version:         1.13.1
 API version:     1.26
 Package version: docker-1.13.1-102.git7f2769b.el7.centos.x86_64
 Go version:      go1.10.3
 Git commit:      7f2769b/1.13.1
 Built:           Mon Aug  5 15:09:42 2019
 OS/Arch:         linux/amd64

Server:
 Version:         1.13.1
 API version:     1.26 (minimum version 1.12)
 Package version: docker-1.13.1-102.git7f2769b.el7.centos.x86_64
 Go version:      go1.10.3
 Git commit:      7f2769b/1.13.1
 Built:           Mon Aug  5 15:09:42 2019
 OS/Arch:         linux/amd64
 Experimental:    false
```

docker-composeのインストールとファイル権限変更

[docker compose公式](https://docs.docker.com/compose/install/)

```
# curl -L https://github.com/docker/compose/releases/download/1.24.1/docker-compose-`uname -s`-`uname -m` -o /usr/local/bin/docker-compose
# chmod +x /usr/local/bin/docker-compose
```

バージョンの確認

```
# docker-compose -version
docker-compose version 1.24.1, build 4667896b
```

## Docker MySQL

dockerイメージの確認

```
# docker images
REPOSITORY          TAG                 IMAGE ID            CREATED             SIZE
```

MySQLのdockerイメージをpullします

```
# docker pull mysql:latest
```

dockerイメージの確認

```
# docker images
REPOSITORY          TAG                 IMAGE ID            CREATED             SIZE
docker.io/mysql     latest              62a9f311b99c        3 weeks ago         445 MB
```

### コンテナの起動と確認
MySQLイメージからMYSQL_ROOT_PASSWORD=mysqlにてパスワードを設定しコンテナを起動します。今回はポートフォワードを3307から3306で行います。docker run後docker psでコンテナが立ち上がってることを確認しましょう

確認

```
# docker ps
CONTAINER ID        IMAGE               COMMAND             CREATED             STATUS              PORTS 
```

コンテナ起動

```
# docker run --name mysqld -e MYSQL_ROOT_PASSWORD=mysql -d -p 3307:3306 mysql
```

確認(CONTAINER IDを確認)

```
# docker ps
CONTAINER ID        IMAGE               COMMAND                  CREATED             STATUS              PORTS                                         NAMES
1c0bb6af2ee4        mysql               "docker-entrypoint..."   30 seconds ago      Up 29 seconds       3306/tcp, 33060/tcp, 0.0.0.0:3307->3306/tcp   mysqld
```

### bashモードで接続
コンテナに直接接続します。docker psにて出力された**CONTAINER ID**を指定しましょう。

```
# docker exec -it 1c0bb6af2ee4 bash
root@1c0bb6af2ee4:/# mysql -u root -p
Enter password:

mysql> exit
Bye

root@1c0bb6af2ee4:/# exit
exit
```

### 仮想サーバからの接続
仮想サーバのMySQL Clientから接続

```
# mysql -u root -p --port=3307 -h127.0.0.1
Enter password:

mysql>
mysql> show databases;
+--------------------+
| Database           |
+--------------------+
| information_schema |
| mysql              |
| performance_schema |
| sys                |
+--------------------+
4 rows in set (0.10 sec)

mysql> exit
```

### docker MySQLコンテナの停止と削除
確認したCONTAINER IDからdocker MySQLコンテナの停止と削除

```
# # docker ps
CONTAINER ID        IMAGE               COMMAND                  CREATED             STATUS              PORTS                               NAMES
4fc496c089a1        mysql               "docker-entrypoint..."   About an hour ago   Up About an hour    33060/tcp, 0.0.0.0:3307->3306/tcp   mysqld

# docker stop 4fc496c089a1
# docker rm mysqld
# docker ps -a
CONTAINER ID        IMAGE               COMMAND             CREATED             STATUS              PORTS               NAMES
```

## docker-compose

```
 cd /Vagrant
# mkdir docker_mysql
# vi docker-compose.yml
version: '3'
services:
  mysql:
    build: ./mysql/
    ports:
      - "3307:3306"
    environment:
      - MYSQL_DATABASE=wordpress
      - MYSQL_ROOT_PASSWORD=mysql

# mkdir mysql
# cd mysql
# vi Dockerfile
FROM mysql:latest
ADD ./my.cnf /etc/mysql/conf.d/my.cnf

# vi my.cnf
[mysqld]
innodb-buffer-pool-size=128M

# cd ..

# docker-compose build
# docker images
# docker-compose up -d
# docker ps
# docker exec -it 84336a6fb964 bash
# mysql -u root -p -h127.0.0.1 --port=3307
Enter password:
Welcome to the MySQL monitor.  Commands end with ; or \g.
Your MySQL connection id is 11
Server version: 8.0.17 MySQL Community Server - GPL

Copyright (c) 2000, 2019, Oracle and/or its affiliates. All rights reserved.

Oracle is a registered trademark of Oracle Corporation and/or its
affiliates. Other names may be trademarks of their respective
owners.

Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.

mysql> show databases;
+--------------------+
| Database           |
+--------------------+
| information_schema |
| mysql              |
| performance_schema |
| sys                |
| wordpress          |
+--------------------+
5 rows in set (0.00 sec)

mysql> exit
Bye
#
```
### PORTの設定
今回のミドルウェアで外部からアクセスさせるPORTの80,3306,5000を解放しましょう

#### 確認
[参考 redhat:4.5.1. firewalld の概要](https://access.redhat.com/documentation/ja-JP/Red_Hat_Enterprise_Linux/7/html/Security_Guide/sec-Using_Firewalls.html)

```
# firewall-cmd --list-all
public (default, active)
  interfaces: enp0s3 enp0s8
  sources:
  services: dhcpv6-client ssh
  ports:
  masquerade: no
  forward-ports:
  icmp-blocks:
  rich rules:
```

#### ポートの解放
```
# firewall-cmd --add-service=http --permanent
# firewall-cmd --permanent --add-port=3306/tcp
# firewall-cmd --permanent --add-port=5000/tcp
# firewall-cmd --reload
```

#### 確認
```
# firewall-cmd --list-all
public (default, active)
  interfaces: enp0s3 enp0s8
  sources:
  services: dhcpv6-client http ssh
  ports: 3306/tcp 5000/tcp
  masquerade: no
  forward-ports:
  icmp-blocks:
  rich rules:
```

### 起動shellの作成
ここまで作成した環境をshellにおこし、Vagrantを初回起動した際に自動で作成できるようにしましょう

#### vagrantのおさらい

起動(初回時のみVagrantfileを反映、以降はオプション指定が必要)
```
$ vagrant up
```

接続
```
$ vagrant ssh
```

停止
```
$ vagrant halt
```

破棄
```
$ vagrant destroy
```

#### Vagrantfileの変更
Vagrantfile内でシェルスクリプトの呼び出しを記載します

```
$ vi Vagrantfile
11-12行目の間に
+ config.vm.provision :shell, :path => "bootstrap.sh"
```
Vagrantfileに追記したbootstrap.shを作成します
```
$ vi bootstrap.sh
$ cat bootstrap.sh
yum install -y git
yum install -y docker
systemctl start docker
yum install -y vim
yum install -y emacs
curl -L https://github.com/docker/compose/releases/download/1.16.1/docker-compose-`uname -s`-`uname -m` -o /usr/local/bin/docker-compose
chmod +x /usr/local/bin/docker-compose
firewall-cmd --add-service=http --permanent
firewall-cmd --permanent --add-port=3306/tcp
firewall-cmd --permanent --add-port=5000/tcp
firewall-cmd --reload
```
#### 再構築
現在のvagrant環境を破棄し再作成しましょう

破棄
```
$ vagrant destroy
```

確認
```
$ vagrant status
Current machine states:

default                   not created (virtualbox)
```

作成
```
$ vagrant up
```

確認
```
$ vagrant status
Current machine states:

default                   running (virtualbox)
```

余談
`/Users/miurahironori/Vagrant/1day`は任意のパス(みなさんが今回1day用に作成したパス)、`-p 2200`も`vagrant ssh-config`で出力されたポートフォワード用のポートになります

```
$ ssh -i /Users/miurahironori/Vagrant/1day/.vagrant/machines/default/virtualbox/private_key -o StrictHostKeyChecking=no -o PasswordAuthentication=no -p 2200 vagrant@127.0.0.1
```

#### 確認
各ミドルウェアがインストールされているか今回はバージョンの出力でコマンドラインから確認し、起動確認はsystemctlで確認しましょう
```
# docker -v
Docker version 1.12.6, build 0fdc778/1.12.6
# docker-compose version
docker-compose version 1.16.1, build 6d1ac21
docker-py version: 2.5.1
CPython version: 2.7.13
OpenSSL version: OpenSSL 1.0.1t  3 May 2016

# git --version
git version 1.8.3.1

# systemctl status docker
● docker.service - Docker Application Container Engine
   Loaded: loaded (/usr/lib/systemd/system/docker.service; disabled; vendor preset: disabled)
   Active: active (running) since Thu 2017-09-14 09:01:57 UTC; 2min 9s ago
     Docs: http://docs.docker.com
 Main PID: 6427 (dockerd-current)
   CGroup: /system.slice/docker.service
           ├─6427 /usr/bin/dockerd-current --add-runtime docker-runc=/usr/libexec/docker/docker-runc-current --default-runtime=docker-runc --exec-opt native.cgroupdriver=systemd --userla...
           └─6432 /usr/bin/docker-containerd-current -l unix:///var/run/docker/libcontainerd/docker-containerd.sock --shim docker-containerd-shim --metrics-interval=0 --start-timeout 2m ...

Sep 14 09:01:57 1day.local dockerd-current[6427]: time="2017-09-14T09:01:57.285985382Z" level=info msg="Graph migration to content-addressability took 0.00 seconds"
Sep 14 09:01:57 1day.local dockerd-current[6427]: time="2017-09-14T09:01:57.286219153Z" level=warning msg="mountpoint for pids not found"
Sep 14 09:01:57 1day.local dockerd-current[6427]: time="2017-09-14T09:01:57.286363671Z" level=info msg="Loading containers: start."
Sep 14 09:01:57 1day.local dockerd-current[6427]: time="2017-09-14T09:01:57.298511181Z" level=info msg="Firewalld running: true"
Sep 14 09:01:57 1day.local dockerd-current[6427]: time="2017-09-14T09:01:57.428360733Z" level=info msg="Default bridge (docker0) is assigned with an IP address 172.17.0.0/16. ...IP address"
Sep 14 09:01:57 1day.local dockerd-current[6427]: time="2017-09-14T09:01:57.528733860Z" level=info msg="Loading containers: done."
Sep 14 09:01:57 1day.local dockerd-current[6427]: time="2017-09-14T09:01:57.528816184Z" level=info msg="Daemon has completed initialization"
Sep 14 09:01:57 1day.local dockerd-current[6427]: time="2017-09-14T09:01:57.528828382Z" level=info msg="Docker daemon" commit="0fdc778/1.12.6" graphdriver=devicemapper version=1.12.6
Sep 14 09:01:57 1day.local systemd[1]: Started Docker Application Container Engine.
Sep 14 09:01:57 1day.local dockerd-current[6427]: time="2017-09-14T09:01:57.546709417Z" level=info msg="API listen on /var/run/docker.sock"
Hint: Some lines were ellipsized, use -l to show in full.
```

### Question
設定したポートが利用可能かどうか確認しましょう。これまでのオペレーションで確認コマンドは実行しています

### MySQL(Docker)
今回MySQL環境はCentOS環境に直接インストールせずDockerイメージを利用します

#### イメージのpull
MySQLのdockerイメージをpullします

確認
```
# docker images
REPOSITORY          TAG                 IMAGE ID            CREATED             SIZE
```

MySQL imageのpull
```
# docker pull mysql
Using default tag: latest
Trying to pull repository docker.io/library/mysql ...
latest: Pulling from docker.io/library/mysql
aa18ad1a0d33: Pull complete
1324423d52e9: Pull complete
ee152a81987c: Pull complete
b054cf37bcae: Pull complete
eb0cbad90353: Pull complete
710fc2dae571: Pull complete
5ff098d10285: Pull complete
79f9dfa6f98f: Pull complete
5d033d38431e: Pull complete
bdde10ea566e: Pull complete
1a36bb8deb0e: Pull complete
Digest: sha256:dfaabbd5466dafe04c5af0ccb1a43d8a18a9604ac23a95534bb0626b3321034d
```

確認
```
# docker images
REPOSITORY                TAG                 IMAGE ID            CREATED             SIZE
docker.io/mysql           latest              aeaed9976244        41 hours ago        412.3 MB
```
#### コンテナの起動と確認
MySQLイメージから`MYSQL_ROOT_PASSWORD=mysql`にてパスワードを設定しコンテナを起動します。今回はポートフォワードを3306から3306で行います。docker run後docker psでコンテナが立ち上がってることを確認しましょう

確認
```
# docker ps
CONTAINER ID        IMAGE               COMMAND             CREATED             STATUS              PORTS               NAMES
```

コンテナ起動
```
# docker run --name mysqld -e MYSQL_ROOT_PASSWORD=mysql -d -p 3306:3306 mysql
```

確認(CONTAINER IDは重要)
```
# docker ps
CONTAINER ID        IMAGE               COMMAND                  CREATED             STATUS              PORTS               NAMES
54caa9126c32        mysql               "docker-entrypoint.sh"   4 seconds ago       Up 3 seconds        3306/tcp            mysqld
```

#### bashモードで接続
コンテナに直接接続します。`docker ps`にて出力された**`CONTAINER ID`**を指定しましょう。
```
# docker exec -it 54caa9126c32 bash

root@54caa9126c32:/# ★に接続(以下プロンプトは消去

# mysql -u root -p
Enter password:
Welcome to the MySQL monitor.  Commands end with ; or \g.
Your MySQL connection id is 4
Server version: 5.7.19 MySQL Community Server (GPL)

Copyright (c) 2000, 2017, Oracle and/or its affiliates. All rights reserved.

Oracle is a registered trademark of Oracle Corporation and/or its
affiliates. Other names may be trademarks of their respective
owners.

Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.

mysql>
mysql> exit;
Bye
# exit
exit
#
```

#### MySQLクライアントの導入
GUEST OS(CentOS)上からMySQLコマンドを実行してみましょう。クライアントが存在しないのでエラーになるはずです。MySQL dockerコンテナにGUESTから接続できるよう設定していきます。

確認
```
# mysql -u root -p
-bash: mysql: command not found
```

クライアントをインストールします(クライアントが存在するmysqlサーバリポジトリをインストール)
```
# yum install -y mysql
```

MySQLコンテナのipアドレスの確認 その１  
`inet 172.17.0.2`を確認。ただし`ip`コマンドが使用できない場合は割愛し、その２を行う。
```
# docker exec -it 54caa9126c32 bash
root@54caa9126c32:/# ip a
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
    inet6 ::1/128 scope host
       valid_lft forever preferred_lft forever
5: eth0@if6: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc noqueue state UP group default
    link/ether 02:42:ac:11:00:02 brd ff:ff:ff:ff:ff:ff
    inet 172.17.0.2/16 scope global eth0
       valid_lft forever preferred_lft forever
    inet6 fe80::42:acff:fe11:2/64 scope link
       valid_lft forever preferred_lft forever
root@54caa9126c32:/# exit
exit
```

MySQLコンテナのipアドレスの確認 その2  
`"IPv4Address": "172.17.0.2/16",`を確認
```
# docker network inspect bridge
[
    {
        "Name": "bridge",
        "Id": "17df6c9580e6c2b6c6458b0a7a4136cdb288744c9bf96954c1344cf7fb1c2650",
        "Scope": "local",
        "Driver": "bridge",
        "EnableIPv6": false,
        "IPAM": {
            "Driver": "default",
            "Options": null,
            "Config": [
                {
                    "Subnet": "172.17.0.0/16"
                }
            ]
        },
        "Internal": false,
        "Containers": {
            "ae3ee5732057705518f78def29ddad4f26ca1f19185d336c042b91b8a5a04b0a": {
                "Name": "mysqld",
                "EndpointID": "d1212cb09110e81c0a887d85996cb937dabfa1bb83f63bd5d7eb4022026d4c88",
                "MacAddress": "02:42:ac:11:00:02",
                "IPv4Address": "172.17.0.2/16",
                "IPv6Address": ""
            }
        },
        "Options": {
            "com.docker.network.bridge.default_bridge": "true",
            "com.docker.network.bridge.enable_icc": "true",
            "com.docker.network.bridge.enable_ip_masquerade": "true",
            "com.docker.network.bridge.host_binding_ipv4": "0.0.0.0",
            "com.docker.network.bridge.name": "docker0",
            "com.docker.network.driver.mtu": "1500"
        },
        "Labels": {}
    }
]
```

確認したIPで接続
```
# mysql -u root -p -h172.17.0.2
Enter password:
Welcome to the MariaDB monitor.  Commands end with ; or \g.
Your MySQL connection id is 5
Server version: 5.7.19 MySQL Community Server (GPL)

Copyright (c) 2000, 2017, Oracle, MariaDB Corporation Ab and others.

Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.

MySQL [(none)]>
MySQL [(none)]> show databases;
+--------------------+
| Database           |
+--------------------+
| information_schema |
| mysql              |
| performance_schema |
| sys                |
+--------------------+
4 rows in set (0.00 sec)

MySQL [(none)]> exit
Bye
```

ループバックアドレスでも可能
```
# mysql -u root -p -h127.0.0.1
Enter password:
Welcome to the MariaDB monitor.  Commands end with ; or \g.
Your MySQL connection id is 5
Server version: 5.7.19 MySQL Community Server (GPL)

Copyright (c) 2000, 2017, Oracle, MariaDB Corporation Ab and others.

Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.

MySQL [(none)]>
```

