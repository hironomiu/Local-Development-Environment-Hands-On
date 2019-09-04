# Step2
このStepではStep1で作成した仮装環境をLAMP環境にカスタマイズしていきます。

## LAMP環境の構築
Step1で構築した環境にapache、php、mysqlをインストールしLAMP環境を構築します

## PHP7、httpd
```
yum -y install epel-release
yum -y update
yum -y install http://rpms.famillecollet.com/enterprise/remi-release-7.rpm
yum -y update

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

