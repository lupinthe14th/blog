---
title: "Arch Linux ARM on Rasberry Pi2 インストール"
summary: "Rasberry Pi2 に Arch Linux ARM のインストールを行い、SSH接続まで確認する。"
date: 2016-02-28T18:00:00+09:00
lastmod: 2016-10-17T22:55:00+09:00
images: []
tags: ["Rasberry Pi2", "Arm Linux ARM", "Lenovo", "USB boot", "lubuntu 14.04.2 LTS"]
draft: false
---


## ゴール
Rasberry Pi2 に Arch Linux ARM のインストールを行い、ssh にて接続できることをゴールとする。


### 前提
次の機器を用いている。

- Lenovo G570
- MacBook Air
- USBメモリ（2GByte）これはlubuntu 14.04.2 LTSの起動メディアとして作成済み
- [Raspberry Pi2 Model B ボード＆ケースセット- Physical Computing Lab](https://www.amazon.co.jp/dp/B00TBKFAI2)
- [Amazonベーシック ハイスピードHDMIケーブル 0.9m (タイプAオス - タイプAオス、イーサネット、3D、4K、オーディオリターン、PS3、PS4、Xbox360対応)](https://www.amazon.co.jp/dp/B014I8SIJY)
- モニタ（REGZA）
- [iClever 2.4GHzミニワイヤレスキーボード(IC-RF01)/超便利MiniワイヤレスBluetoothキーボード](https://www.amazon.co.jp/dp/B00HMXIKCS)
- [iBUFFALO カードリーダー/ライター microSD対応 超コンパクト](https://www.amazon.co.jp/dp/B001MQBRJO)
- [東芝 Toshiba microSDHC UHS-I 16GB EXCERIA超高速95MB/秒 日本製、パッケージ品 並行輸入品](https://www.amazon.co.jp/dp/B00C67VPGI)


## インストール手順

### 前準備

1. Lenovo G570のBIOSをUSBメモリからbootするように変更する
1. USBメモリを挿し、電源ONでlubuntuを起動させる
1. bsdtarをインストールする

   ```
   apt-get install bsdtar
   ```

1. カードリーダー/ライターにmicroSDカードを挿入して、Lenobo G570に挿入

### インストール

- [Raspberry Pi 2 | Arch Linux ARM](https://archlinuxarm.org/platforms/armv7/broadcom/raspberry-pi-2)
のとおりインストールを行う。

- 対象デバイスは ```ls -al /dev/sd*``` でデバイスを確認し、 ```fdisk``` で中身を確認して間違えないようにする。

### 起動

作成したSDカードをRasberry Pi2に挿入し電源を入れ、モニターで起動を確認する。


### ログイン

アカウントとパスワードは
[Raspberry Pi 2 | Arch Linux ARM](https://archlinuxarm.org/platforms/armv7/broadcom/raspberry-pi-2)
のとおり。

### IPアドレスの調査

DHCPで割り当てられたIPアドレスを調べる為、 ```ifconfig``` でIPアドレスを確認する。

### SSHログイン

確認したIPアドレスに対してMacBook AirのiTerm2からSSH接続する。

アカウントはrootじゃない方で。

```
% ssh alarm@192.168.0.2
alarm@192.168.0.2's password:
Welcome to Arch Linux ARM

     Website: http://archlinuxarm.org
       Forum: http://archlinuxarm.org/forum
         IRC: #archlinux-arm on irc.Freenode.net
```

無事SSH接続できました。

## 所感

当初はMacBook AirでなんとかArchLinuxのSDカードを作ろうとしていました。
しかし手元にあるLenobo G570とlubuntuのUSBメモリでLinuxが利用できることに気がついたので、オフィシャルサイトの手順で作成できるこの方法に落ち着きました。
イメージファイルをダウンロードして ```dd``` で作成しようとしましたが、イメージファイルが古いのしか見つけられなかったのもこの方法を選択した要因でもあります。

## 参考文献

- [Arch Linux ARM](https://archlinuxarm.org)
- [Raspberry Pi 2にArch Linuxを設定する](https://tkamada.blogspot.jp/2016/01/settting-up-arch-linux-on-aspberry-pi-2.html)
