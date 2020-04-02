---
title: "おうちKubernetes ~ はじめに ~"
date: 2019-08-22T13:15:10+09:00
tags:
- Kubernetes
- Memo
- arm
- single board computer
draft: false
summary: "single board computerを使っておうちKubernetes環境を構築"
---

Rasberry Piなどのsingle board computerを使っておうちKubernetes環境を構築します。


## なぜ構築する？

- [StatefulSets](https://kubernetes.io/docs/concepts/workloads/controllers/statefulset/) 使いたかったから

  利用しているDocker Desktop for Macのバージョン [^1] では、使いたいStatefulSetは1.5より前のKubernetesリリースでは使用できないため。 [^2]

  [^1]: [Docker Desktop Community 2.1.0.1](https://docs.docker.com/docker-for-mac/release-notes/#docker-desktop-community-2101)
  [^2]: [Docker Desktop Community 2.2.0.0](https://docs.docker.com/docker-for-mac/release-notes/#docker-desktop-community-2200) でKubernetes1.15.5に対応したようです。

- 所有欲を満たしたい

  クラウドサービスを使えばいいのでしょうが、所有したい世代なもので。


## 構想

1. ハードの追加を容易にしたい

    新しくsingle board computerを追加する際、電源入れたらもうnodeに追加されている状態とかにしたい。

    - [cybozu-go/sabakan](https://github.com/cybozu-go/sabakan)
    
      構想するにいたったネタ。

    - [Neco プロジェクトのスキルシート](https://gist.github.com/ymmt2005/bd92296166e52d1beba9df8ac516a9db)

      必要な技術スタックが羅列されている。

    - [MatchBoxを使ってCoreOS Container Linuxをベアメタルにプロビジョンする全体像 - Qiita](https://qiita.com/kanga/items/de223e4ba3c389777886)

      MatchBox使うとやりたいことが実現できそう。

    - [KRIBを使用してDigital Rebar Provision (DRP)と共にKubernetesをインストールする](https://kubernetes.io/ja/docs/setup/on-premises-metal/krib/)
    
      Digital Rebar Provisionを使ってもできそう。

2. 分散ストレージつかいたい

   単なる興味で使ってみたい。

    - gluster

        ~~これが第一候補。しかし構築が少しむずかしそう。~~

        頻繁に更新されるような場合は適切ではない。

        - [Gopee Design Studio](https://www.gopeedesignstudio.com/2018/07/13/glusterfs-on-arm/)

        - [Gluster | Storage for your Cloud](https://www.gluster.org)

        - [Volumes](https://kubernetes.io/docs/concepts/storage/volumes/#glusterfs)

        - [Gopee Design Studio](https://www.gopeedesignstudio.com/2018/07/13/glusterfs-on-arm/)

    - Ceph

        こちらを採択する。Rookを使ってdeployする。

        - [Rook](https://rook.io/)

        - [Homepage - Ceph](https://ceph.io/)

        - [Volumes](https://kubernetes.io/docs/concepts/storage/volumes/#cephfs)

    - DRDB

      Kubernetesクラスタのマスタ・ノードそれぞれで作DRDBクラスタ？よくわかっていない。4台構成が限界なのかな？

      - [DRBD Community](https://www.linbit.com/en/drbd-community/)
      - [index | DRBD](https://drbd.jp/)


## 実践

[おうちKubernetes ~ 環境編 ~]({{< ref "/posts/myhomekubernetes/env.md" >}})へ続く。

ハードの追加を容易にしたい構想は、試してみましたが私には実現することができませんでした。[^3] ARMなsingle board computer を使う前提を変えればできたかもしれませんが今回はやめました。

[^3]: ARMなsingle board computerでCoreOS Container Linuxをbootできなかった
