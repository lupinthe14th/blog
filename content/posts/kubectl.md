---
title: "kubectlでのクラスタ切替"
date: 2019-09-04T09:59:00+09:00
lastmod: 2019-09-04T10:04:00+09:00
tags: ["Kubernetes", "Memo", "Tool"]
---

kubectl config を使う。

- 一覧

        kubectl config get-context

- 切替

        kubectl config use-context $CLUSTER_NAME

- Currentの表示

        kubectl config current-context


SeeAlso: https://qiita.com/sonots/items/f82912367693d717ff06
