+++
title = "AWS free で　IPF定量的プロジェクトインストール"
date = 2014-02-20T09:22:28+09:00
lastmod = 2014-02-22T09:22:28+09:00
images = []
tags = ["AWS", "ipf", "CentOS:5.6"]
categories = []
+++

# 目的

やりたい事は、AWSで無料のインターネット接続可能なプロジェクト管理画面を使いたい。

# 注意事項

 **AWS無料範囲でやろうとしている最中の投稿になります。** 
 **この投稿を参考に課金されても自己責任でお願い致します。** 

現状課金されていて、課金されないように変更の真っ最中です。

# 手順

## インスタンスの作成
以下でCentOS5.6のインスタンスを作成します。
IPFの簡易インストーラーは、上記バージョンでないとインストールに失敗しますので、CentOS5.6を利用する事をお勧めします。

- AWS の無料トライアルを申込
- Key pair を作成。
- VPSを構築。手始めに蛸壺モデルで。
- EC2でInstanceを作成。Community AMIsで、CentOS5.6を検索。Sizeがt1.microで構築出来る事を確認して先に進む。
- Security Groups には、inboundにhttpを追加。
- Elastic IPs を追加。
- EBSはstanderd で8GBから30GBに拡張。

## CentOS5.6環境の構築
構築したサーバにsshログイン。接続方法は、対象インスタンスを選択してconnectボタンで接続方法が記載されています。

[インスタンスへの接続](http://docs.aws.amazon.com/ja_jp/AWSEC2/latest/UserGuide/ec2-connect-to-instance-linux.html "インスタンスへの接続")

### 拡張したファイルシステムのresize

```
# df -h
Filesystem          サイズ  使用  残り 使用% マウント位置
/dev/sda1             7.9G  2.8G  4.7G  38% /
none                  308M     0  308M   0% /dev/shm

# resize2fs /dev/sda1
resize2fs 1.39 (29-May-2006)
Filesystem at /dev/sda1 is mounted on /; on-line resizing required
Performing an on-line resize of /dev/sda1 to 7864320 (4k) blocks.
The filesystem on /dev/sda1 is now 7864320 blocks long.

# df -h
Filesystem          サイズ  使用  残り 使用% マウント位置
/dev/sda1              30G  2.8G   26G  10% /
none                  308M     0  308M   0% /dev/shm

```
* ご参考
[ボリュームのストレージ領域を拡張する](http://docs.aws.amazon.com/ja_jp/AWSEC2/latest/UserGuide/ebs-expand-volume.html "ボリュームのストレージ領域を拡張する")

### localtimeの設定。

```
# rm /etc/localtime
# cp -p /usr/share/zoneinfo/Japan /etc/localtime
```

### ntpの設定
ntpのインストール

```
# yum -y install ntp
```

### ntpが起動しない場合。
* ご参考。
[EC2(CentOS)で時刻同期(NTP)](http://blog.suz-lab.com/2011/05/ec2centosntp.html "EC2(CentOS)で時刻同期(NTP)")

### /var/log/messagesにmodprobeのFATALが出ている場合。
* ご参考。
[AMI(Linux)のカーネルを"2.6.18-xenU-ec2-v1.5"にしてみた](http://blog.suz-lab.com/2011/04/amilinux2618-xenu-ec2-v15.html "AMI(Linux)のカーネルを\"2.6.18-xenU-ec2-v1.5\"にしてみた") 

# IPFの定量的プロジェクト管理ツール（EPM-X）のインストール
構築したインスタンスにIPFのプロジェクト管理ツールをインストールします。今回はRedmineバージョンをインストールします。

### wget のインストール
wget がインストールされていない場合インストールします。

```
# yum -y install wget
```

## インストール

- wgetでIPFのインストールメディアをダウンロードします。

```
# wget http://www.ipa.go.jp/files/000005063.bin
```

- インストーラーの実行

```
# bash 000005063.bin
```

* リポジトリは好みで。
* サンプルプロジェクトも好みで。ただし、サンプルプロジェクトを作成している最中に失敗する場合があるので、その時はサンプルプロジェクト無しでインストールしなおしてください。
