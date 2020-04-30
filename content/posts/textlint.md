---
title: "Textlint"
date: 2020-04-30T18:16:05+09:00
tags: ["blog","textlint","ci","github action", "reviewdog", "netlify"]
draft: true
---

ブログ記事をコードレビュー形式にします。

文章校正は [textlint]を使い、実行はGithub Actionで行い、Reviewdogにコメントさせます。
また、netlifyのBranch deploy controlsを使ってBranch deployを行いDeploy PreviewしてからProduction deployさせるようにします。



## 文章校正のCI化

[Run textlint with reviewdog](https://github.com/marketplace/actions/run-textlint-with-reviewdog)を使います。

.textlintrcは好みで修正します。

以上。終わり。
