---
title: "GitHub Actions Tips"
date: 2020-04-16T17:41:48+09:00
tags: ["Memo","tips","GitHub Actions"]
draft: false
---

GitHub Actionsに関連するメモとかTipとか。

## 手動でGitHub Actionを再実行する方法

### ユースケース

GitHub Actionでテストを回していると時折 ステータスチェックが `Expected — Waiting for status to be reported` のままとなる。

再実行したくても、GitHubから手動操作はできない様子。

### 回避策

空コミット `git commit --allow-empty -m "wakey wakey GitHub Actions"` [^1] してプッシュする。


[^1]: wakey wakey とは起きろ!の意

### 参考
- [Manually restart actions and entire workflows?](https://github.community/t5/GitHub-Actions/Manually-restart-actions-and-entire-workflows/td-p/31875)
