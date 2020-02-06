---
title: Macでシリアル接続
date: 2016-10-11T00:00:00+09:00
categories: ["memo"]
tags: ["Mac OS X", "シリアル", "USBシリアルケーブル", "Console", "memo"]
author:
  given_name: Hideo
  family_name: Suzuki
  display_name: Hideo Suzuki
summary: "Mac OS Xからコンソール接続するときの方法をすぐ忘れるのでメモ"
---

## 接続方法

1. デバイスの確認

   ```console
   % ls /dev/tty.usb*
   /dev/tty.usbserial-FTK1SOHS
   ```

1. 接続

   ```console
   % screen /dev/tty.usbserial-FTK1SOHS
   ```

1. 切断

   `Control-a k` で、 `Really kill this window [y/n]` と表示されるので、 y を押して screen を終了する。

## 参考資料

- [Mac の screen コマンドでシリアル通信](http://qiita.com/hideyuki/items/9258f33180d98ad0cb1e)
