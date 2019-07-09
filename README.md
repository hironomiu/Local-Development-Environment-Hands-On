# Local-Development-Environment-Hands-On

## 概要
ローカルPCにVirtualBox、Vagrantを用いてLinux環境を構築し、Linux環境にDockerコンテナを立ち上げてVirtualBox、Vagrant,Dockerの基本的な扱い方を学びます。

## 前提
VirtualBox、Vagrantがインストールされていること。DockerはVirtualBox、Vagrantで構築した仮想環境で利用するのでインストールは必須ではありません。

[VirtualBox公式](https://www.virtualbox.org/)

[Vagrant公式](https://www.vagrantup.com/)

## 表記、用語について
- GUEST(ゲスト) OSは仮想サーバと同じ意味です
- HOST(ホスト) OSは持参したPC(のターミナル)を指します  
- 「#」はGUEST OS中でのrootユーザでのオペレーションを指します
- 「$」はHOST OSと表記が無い限りはGUEST OS vagrantユーザでのオペレーションを指します
- インラインコード中、行の先頭に「+」「-」がある場合「+」は以降にある文字列を追加、「-」は存在する文字列を消去と読み取ってください

