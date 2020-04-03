---
title: "おうちKubernetes ~ 環境編 ~"
date: 2020-04-02T11:48:45+09:00
tags:
- Kubernetes
- Memo
- arm
- single board computer
draft: false
---

さて、ざっくりした方針がたったので、環境を整えます。
まずは、おうちKubernetes環境を作るための準備です。

構築時点でRasberry Pi 4は日本国内未販売でしたので、手元にあったものと買い足して３台構成で構築します。[^1]

追加費用は2万円いかないくらいだったかと。

[^1]: LANポートが足りないのでついでに [スイッチ](https://www.amazon.co.jp/gp/product/B004BQCKXO/ref=ppx_yo_dt_b_asin_title_o00_s00?ie=UTF8&psc=1) 買ったりもしてた。

## 物理

|Name|Quantity|Note|
|---|---|---|
|Raspberry Pi 3B+|1|新規購入|
|Raspberry Pi 3B|1||
|Raspberry Pi 2|1||
|[4段積層式ケース](https://www.amazon.co.jp/dp/B01F8AHNBA/ref=cm_sw_em_r_mt_dp_U_YEJxDbZ2515AG)|1|Rasberry Pi 4を買ったら積載するつもり|
|[SDカード](https://www.amazon.co.jp/gp/product/B01G6RJDUS/ref=ppx_yo_dt_b_asin_title_o01_s00?ie=UTF8&psc=1)|3||
|[USB充電器](https://www.amazon.co.jp/gp/product/B00YS3ZYWY/ref=ppx_yo_dt_b_asin_title_o08_s00?ie=UTF8&psc=1)|1||
|LANケーブル|4|Cat6a 0.5m|
|USB 電源ケーブル|3|5V2A|

## 論理

|SBC|OS|Note|
|---|---|---|
|Rasberry Pi 3B+|ArchLinux ARM AArch64||
|Rasberry Pi 3B|ArchLinux ARM AArch64||
|Rasberry Pi 2|ArchLinux ARM||


[おうちKubernetes ~ 構築編 ~]({{< ref "/posts/myhomekubernetes/build.md" >}})へ続く。
