---
title: "kubectlチートシート"
date: 2019-09-04T09:59:00+09:00
lastmod: 2019-09-04T10:04:00+09:00
tags: ["Kubernetes", "Memo", "Tool"]
---

## Kubernetesでポッド内のコンテナを実行

一つの方法として、`kubectl run` を用いる。

```bash
kubectl run nginx --image=nginx
```

## 状態を表示

`kubectl get` を用いる。

- Pod

   ```bash
   kubectl get pods 
   ```

   `kubectl get po` でも可。

- Service

   ```bash
   kubectl get service
   ```

   `kubectl get svc` でも可。

- Deployment

   ```bash
   kubectl get deployment
   ```

   `kubectl get deploy` でも可。


## クラスタ切替

kubectl config を使う。

- 一覧

   ```bash
   kubectl config get-context
   ```

- 切替

   ```bash
   kubectl config use-context $CLUSTER_NAME
   ```

- Currentの表示

   ```bash
   kubectl config current-context
   ```

SeeAlso
: https://qiita.com/sonots/items/f82912367693d717ff06
