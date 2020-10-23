# Step0
事前確認。このStepでは以降のStepを行うための準備と確認を行います。

## VirtualBox、Vagrantを以下の公式サイトからインストールしましょう

[VirtualBox公式](https://www.virtualbox.org/)

[Vagrant公式](https://www.vagrantup.com/)

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
  config.vm.box = "centos/8"
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
  config.vm.box = "centos/8"
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
[vagrant@test ~]$ exit

$ vagrant destroy
    default: Are you sure you want to destroy the 'default' VM? [y/N] y
==> default: Destroying VM and associated drives...
```

ここまででStep0は完了です。
