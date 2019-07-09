# Local-Development-Environment-Hands-On

## 概要
ローカルPCにVirtualBox、Vagrantを用いてLinux開発環境を構築し、その環境にDockerコンテナを立ち上げてVirtualBox、Vagrant,Dockerの基本的な扱い方を学びます。

## 前提環境
VirtualBox、Vagrantがインストールされていること。DockerはVirtualBox、Vagrantで構築したLinux開発環境で利用するのでインストールは必須ではありません。

[VirtualBox公式](https://www.virtualbox.org/)

[Vagrant公式](https://www.vagrantup.com/)

ssh接続可能なターミナルが利用できること

## 前提知識
一般的なLinuxコマンド(ディレクトリ遷移、ファイル移動、ファイル編集)が扱えること。ハンズオン中ファイル編集はvimにて実演するため自身が扱うファイル編集方法に読み替えられること

## 事前確認
任意のディレクトリで以下のコマンどを実行しエラーとならず`Vagrantfile`が作成されていること

```
$ vagrant init
$ ls -la
-rw-r--r--  1 xxxx  staff  3011  7  9 16:02 Vagrantfile
```

Vagrantfileを以下に書き換える
```
# -*- mode: ruby -*-
# vi: set ft=ruby :

Vagrant.configure("2") do |config|
  config.vm.box = "centos/7"
  config.vm.hostname = "test.local"
  config.vm.network :private_network, ip: "192.168.56.60"
  config.vm.provider :virtualbox do |vb|
    vb.name = "test"
    vb.customize ["modifyvm", :id, "--memory", "768"]
  end
end
```

書き換えの確認
```
$ cat Vagrantfile
# -*- mode: ruby -*-
# vi: set ft=ruby :

Vagrant.configure("2") do |config|
  config.vm.box = "centos/7"
  config.vm.hostname = "test.local"
  config.vm.network :private_network, ip: "192.168.56.60"
  config.vm.provider :virtualbox do |vb|
    vb.name = "test"
    vb.customize ["modifyvm", :id, "--memory", "768"]
  end
end
```

実行しエラーとならずssh接続できること
```
$ vagrant up

==> default: Setting hostname...
==> default: Configuring and enabling network interfaces...
==> default: Rsyncing folder: /test/ => /vagrant

$ vagrant ssh
[vagrant@test ~]$
```

## 表記、用語について
- GUEST(ゲスト) OSは仮想サーバと同じ意味です
- HOST(ホスト) OSは持参したPC(のターミナル)を指します  
- 「#」はGUEST OS中でのrootユーザでのオペレーションを指します
- 「$」はHOST OSと表記が無い限りはGUEST OS vagrantユーザでのオペレーションを指します
- インラインコード中、行の先頭に「+」「-」がある場合「+」は以降にある文字列を追加、「-」は存在する文字列を消去と読み取ってください
