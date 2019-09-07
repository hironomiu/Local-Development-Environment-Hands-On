# Local-Development-Environment-Hands-On

## 概要
ローカルPCにVirtualBox、Vagrantを用いてLinux開発環境を構築し、その環境にDockerコンテナを立ち上げてVirtualBox、Vagrant,Dockerの基本的な扱い方を学びます。

[Step1](https://github.com/hironomiu/Local-Development-Environment-Hands-On/blob/master/Step1.md)  
[Step2](https://github.com/hironomiu/Local-Development-Environment-Hands-On/blob/master/Step2.md)    
[Step3](https://github.com/hironomiu/Local-Development-Environment-Hands-On/blob/master/Step3.md)  
[Step4](https://github.com/hironomiu/Local-Development-Environment-Hands-On/blob/master/Step4.md)  

## 前提環境
VirtualBox、Vagrantがインストールされていること。DockerはVirtualBox、Vagrantで構築したLinux開発環境で利用するのでインストールは必須ではありません。

[VirtualBox公式](https://www.virtualbox.org/)

[Vagrant公式](https://www.vagrantup.com/)

SSHターミナルが利用できること

**重要 上記環境を整え事前に[Step0](https://github.com/hironomiu/Local-Development-Environment-Hands-On/blob/master/Step1.md)を行うこと**

## 前提知識
一般的なLinuxコマンド(ディレクトリ遷移、ファイル移動、ファイル編集)が扱えること。ハンズオン中ファイル編集はvimにて実演するため自身が扱うファイル編集方法に読み替えられること

## 表記、用語について
- GUEST(ゲスト) OSは仮想サーバと同じ意味です
- HOST(ホスト) OSは持参したPC(のターミナル)を指します  
- 「#」はGUEST OS中でのrootユーザでのオペレーションを指します
- 「$」はHOST OSと表記が無い限りはGUEST OS vagrantユーザでのオペレーションを指します
- インラインコード中、行の先頭に「+」「-」がある場合「+」は以降にある文字列を追加、「-」は存在する文字列を消去と読み取ってください
