---
title: "ブログ記事のプルリクエストレビュー化"
date: 2020-04-30T18:16:05+09:00
tags: ["blog","textlint","ci","github action", "reviewdog", "netlify"]
draft: false
---

文章校正は [textlint](https://textlint.github.io/) を使い、実行はGithub Actionで行い、Reviewdogにコメントさせます。
また、netlifyのBranch deploy controlsを使ってBranch deployを行いDeploy PreviewしてからProduction deployさせるようにします。

## Netlifyの設定変更

[Branch deploy controls](https://docs.netlify.com/site-deploys/overview/#branch-deploy-controls)の通り設定を行います。


## 文章校正のCI化

[Run textlint with reviewdog](https://github.com/marketplace/actions/run-textlint-with-reviewdog)を使います。

1. GitHub Actionの設定

    ```bash
    mkdir -p .github/workflows
    ```

    ```yaml:.github/workflows/reviewdog.yaml
    ---
    name: reviewdog
    on: [pull_request]
    jobs:
      textlint:
        name: runner / textlint
        runs-on: ubuntu-latest
        steps:
          - uses: actions/checkout@v1
            with:
              submodules: true
          - name: textlint-github-pr-check
            uses: tsuyoshicho/action-textlint@v1
            with:
              github_token: ${{ secrets.github_token }}
              reporter: github-pr-check
              textlint_flags: "content/**"
          - name: textlint-github-check
            uses: tsuyoshicho/action-textlint@v1
            with:
              github_token: ${{ secrets.github_token }}
              reporter: github-check
              textlint_flags: "content/**"
          - name: textlint-github-pr-review
            uses: tsuyoshicho/action-textlint@v1
            with:
              github_token: ${{ secrets.github_token }}
              reporter: github-pr-review
              textlint_flags: "content/**"
    
    ```

1. [prh/rules](https://github.com/prh/rules) をサブモジュールとして追加

    ```bash
    git submodule add https://github.com/prh/rules.git .prh-rules
    ```

1. .textlintrcの作成

   設定内容は好みで修正します。

    ```
    {
      "plugins": {
        "@textlint/markdown": {
          "extensions": [".md"]
        }
      },
      "rules": {
        "spellcheck-tech-word": true,
        "prh": {
          "rulePaths": ["./.prh.yml"]
        },
        "preset-ja-technical-writing": {
          "max-ten": {
            "max": 3
          },
          "no-mix-dearu-desumasu": true,
          "ja-no-abusage": true,
          "ja-no-redundant-expression": true,
          "no-unmatched-pair": true,
          "no-double-negative-ja": true,
          "no-doubled-conjunctive-particle-ga": true,
        }
      }
    }
    ```

1. prh.yml の作成

   ```
   ---
   version: 1
   
   imports:
     - ./.prh-rules/media/techbooster.yml
     - ./.prh-rules/files/markdown.yml
   ```

以上。終わり。
