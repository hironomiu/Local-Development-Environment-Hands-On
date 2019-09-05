# Step2
このStepではStep1で作成した仮装環境をLAMP環境にカスタマイズしていきます。

## LAMP環境の構築
Step1で構築した環境にapache、php、mysqlをインストールしLAMP環境を構築します

## yum repo
これから各種ミドルウェア(apache、MySQL）やPHPをインストールする際に利用するリポジトリの設定を行う

**`/etc/yum.repos.d/`ディレクトリと内容について簡単に確認**

```
# ll /etc/yum.repos.d/
total 96
-rw-r--r--. 1 root root 1664 Nov 23  2018 CentOS-Base.repo
-rw-r--r--. 1 root root 1309 Nov 23  2018 CentOS-CR.repo
-rw-r--r--. 1 root root  649 Nov 23  2018 CentOS-Debuginfo.repo
-rw-r--r--. 1 root root  314 Nov 23  2018 CentOS-fasttrack.repo
-rw-r--r--. 1 root root  630 Nov 23  2018 CentOS-Media.repo
-rw-r--r--. 1 root root 1331 Nov 23  2018 CentOS-Sources.repo
-rw-r--r--. 1 root root 5701 Nov 23  2018 CentOS-Vault.repo

# cat /etc/yum.repos.d/CentOS-Base.repo

# CentOS-Base.repo
#
# The mirror system uses the connecting IP address of the client and the
# update status of each mirror to pick mirrors that are updated to and
# geographically close to the client.  You should use this for CentOS updates
# unless you are manually picking other mirrors.
#
# If the mirrorlist= does not work for you, as a fall back you can try the
# remarked out baseurl= line instead.
#
#

[base]
name=CentOS-$releasever - Base
mirrorlist=http://mirrorlist.centos.org/?release=$releasever&arch=$basearch&repo=os&infra=$infra
#baseurl=http://mirror.centos.org/centos/$releasever/os/$basearch/
gpgcheck=1
gpgkey=file:///etc/pki/rpm-gpg/RPM-GPG-KEY-CentOS-7

#released updates
[updates]
name=CentOS-$releasever - Updates
mirrorlist=http://mirrorlist.centos.org/?release=$releasever&arch=$basearch&repo=updates&infra=$infra
#baseurl=http://mirror.centos.org/centos/$releasever/updates/$basearch/
gpgcheck=1
gpgkey=file:///etc/pki/rpm-gpg/RPM-GPG-KEY-CentOS-7

#additional packages that may be useful
[extras]
name=CentOS-$releasever - Extras
mirrorlist=http://mirrorlist.centos.org/?release=$releasever&arch=$basearch&repo=extras&infra=$infra
#baseurl=http://mirror.centos.org/centos/$releasever/extras/$basearch/
gpgcheck=1
gpgkey=file:///etc/pki/rpm-gpg/RPM-GPG-KEY-CentOS-7

#additional packages that extend functionality of existing packages
[centosplus]
name=CentOS-$releasever - Plus
mirrorlist=http://mirrorlist.centos.org/?release=$releasever&arch=$basearch&repo=centosplus&infra=$infra
#baseurl=http://mirror.centos.org/centos/$releasever/centosplus/$basearch/
gpgcheck=1
enabled=0
gpgkey=file:///etc/pki/rpm-gpg/RPM-GPG-KEY-CentOS-7
```

**epel**

```
# yum -y install epel-release
# yum -y update
```

**remi**

```
# yum -y install http://rpms.famillecollet.com/enterprise/remi-release-7.rpm
# yum -y update
```

**`/etc/yum.repos.d/`ディレクトリと内容について簡単に確認**

```
# ll /etc/yum.repos.d/
total 96
-rw-r--r--. 1 root root 1664 Nov 23  2018 CentOS-Base.repo
-rw-r--r--. 1 root root 1309 Nov 23  2018 CentOS-CR.repo
-rw-r--r--. 1 root root  649 Nov 23  2018 CentOS-Debuginfo.repo
-rw-r--r--. 1 root root  314 Nov 23  2018 CentOS-fasttrack.repo
-rw-r--r--. 1 root root  630 Nov 23  2018 CentOS-Media.repo
-rw-r--r--. 1 root root 1331 Nov 23  2018 CentOS-Sources.repo
-rw-r--r--. 1 root root 5701 Nov 23  2018 CentOS-Vault.repo
-rw-r--r--. 1 root root  951 Oct  2  2017 epel.repo
-rw-r--r--. 1 root root 1050 Oct  2  2017 epel-testing.repo
-rw-r--r--. 1 root root  446 Mar  8 07:34 remi-glpi91.repo
-rw-r--r--. 1 root root  446 Mar  8 07:34 remi-glpi92.repo
-rw-r--r--. 1 root root  446 Mar  8 07:34 remi-glpi93.repo
-rw-r--r--. 1 root root  446 Mar  8 07:34 remi-glpi94.repo
-rw-r--r--. 1 root root  855 Mar  8 07:34 remi-modular.repo
-rw-r--r--. 1 root root  456 Mar  8 07:34 remi-php54.repo
-rw-r--r--. 1 root root 1314 Mar  8 07:34 remi-php70.repo
-rw-r--r--. 1 root root 1314 Mar  8 07:34 remi-php71.repo
-rw-r--r--. 1 root root 1314 Mar  8 07:34 remi-php72.repo
-rw-r--r--. 1 root root 1314 Mar  8 07:34 remi-php73.repo
-rw-r--r--. 1 root root 2605 Mar  8 07:34 remi.repo
-rw-r--r--. 1 root root  750 Mar  8 07:34 remi-safe.repo
```

## PHP7、apache(httpd)
PHP7、apache(httpd)のインストールを行う

```
yum -y install --enablerepo=remi,remi-php73 php php-devel php-mbstring php-pdo php-gd php-xml php-mcrypt php-mysql

php -v

yum list installed | grep httpd

systemctl start httpd.service
systemctl enable httpd.service

curl localhost
```

## MySQL

```
rpm -qa | grep mariadb

yum -y remove mariadb-libs

rm -rf /var/lib/mysql/

rpm -ivh https://dev.mysql.com/get/mysql80-community-release-el7-3.noarch.rpm

yum repolist all | grep mysql

yum -y install mysql-community-server

systemctl start mysqld.service

systemctl enable mysqld.service

パスワードをログから確認
cat /var/log/mysqld.log
grep password /var/log/mysqld.log


mysql_secure_installation
Enter password for user root: 確認したパスワードを入力
New password: 新しいパスワードを入力(例 i1db+abd8kD )
Re-enter new password: 新しいパスワードを入力(例 i1db+abd8kD )
Change the password for root ? ((Press y|Y for Yes, any other key for No) : Yを入力
Remove anonymous users? (Press y|Y for Yes, any other key for No) : Yを入力
Disallow root login remotely? (Press y|Y for Yes, any other key for No) : Yを入力
Remove test database and access to it? (Press y|Y for Yes, any other key for No) : Yを入力
Reload privilege tables now? (Press y|Y for Yes, any other key for No) : Yを入力

# mysql -u root -p
Enter password:

mysql> show databases;
+--------------------+
| Database           |
+--------------------+
| information_schema |
| mysql              |
| performance_schema |
| sys                |
+--------------------+
4 rows in set (0.00 sec)

mysql>
```

## サンプルアプリのデプロイ

```
cd /var/www/html
yum -y install wget
wget http://wordpress.org/latest.tar.gz
tar -xzvf latest.tar.gz

vi /etc/httpd/conf/httpd.conf
DocumentRoot "/var/www/html/wordpress"
<Directory "/var/www/html/wordpress">

systemctl restart httpd.service
```

