# Step4
このStepではこれまでのStepで作成した環境をスクリプトで自動生成します

(このStepはネットワーク帯域に余裕があり、時間がある場合のみ行う)

## 設定ファイルのコピー
Step3までに作成したミドルウェアの設定ファイルをコピー

```
$ cd 1day
$ vagrant ssh
$ sudo su -
# cd /vagrant
# mkdir conf

# cp /etc/httpd/conf/httpd.conf conf/.

# cp /etc/php.ini conf/.

# cp /etc/my.cnf conf/.
# exit
$ exit
```

## ハンズオン用ディレクトリの作成
HOST OS上の任意のディレクトリで`1day-script`ディレクトリを作成し、遷移しましょう。以降`1day-script`ディレクトリをカレントディレクトリとします。

```
$ mkdir 1day-script
$ cd 1day-script
```

## Vagrant設定ファイル(Vagrantfile)のコピー
Step3までに利用した`Vagrantfile`をコピー

```
```
