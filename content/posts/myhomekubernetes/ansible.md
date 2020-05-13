---
title: "Ansibleを使う〜準備編〜"
date: 2020-05-13T22:29:13+09:00
tags:
- Kubernetes
- Memo
- arm
- single board computer
- Ansible
- Poetry
draft: false
---

新しくRasberry Pi4b買って初期構築したらなんだかんだと3時間くらいかかりました。
それと今まで構築したのがいろいろバラバラになりつつあるので少しでも楽するべく[Ansible](https://github.com/ansible/ansible)を使える様にします。

## [Poetry](https://python-poetry.org)

[Ansible](https://github.com/ansible/ansible)をインストールする際に依存関係を分離したいので、Pythonのパッケージマネージャーを使います。

PythonのパッケージマネージャーはPyenv+vertualenvやPipenvがありますが、よい評判をTwitterで読んだ[Poetry](https://python-poetry.org)を使ってみます。

### Install

[ドキュメント](https://python-poetry.org/docs/#installation)のとおりインストールします。

```fish
curl -sSL https://raw.githubusercontent.com/python-poetry/poetry/master/get-poetry.py | python
```

パスを通します。

```fish
vi ~/.config/fish/config.fish
```

```vi
set -g fish_user_paths "$HOME/.poetry/bin" $fish_user_paths
```

```fish
source ~/.config/fish/config.fish
```

```fish
poetry --version
Poetry version 1.0.5
```

### [Enable tab completion](https://python-poetry.org/docs/#enable-tab-completion-for-bash-fish-or-zsh)

```fish
poetry completions fish > ~/.config/fish/completions/poetry.fish
```

### poetry init

依存関係を分離したいだけで、プロジェクトを作成する必要は無いので`poetry init`します。

```fish
poetry init

This command will guide you through creating your pyproject.toml config.

Package name [hke]:
Version [0.1.0]:
Description []:  Home Kubernetes Engine
Author [lupinthe14th <hideosuzuki@ordinarius-fectum.net>, n to skip]:
License []:  MIT
Compatible Python versions [^3.8]:

Would you like to define your main dependencies interactively? (yes/no) [yes]
You can specify a package in the following forms:
  - A single name (requests)
  - A name and a constraint (requests ^2.23.0)
  - A git url (git+https://github.com/python-poetry/poetry.git)
  - A git url with a revision (git+https://github.com/python-poetry/poetry.git#develop)
  - A file path (../my-package/my-package.whl)
  - A directory (../my-package/)
  - An url (https://example.com/packages/my-package-0.1.0.tar.gz)

Search for package to add (or leave blank to continue):

Would you like to define your development dependencies interactively? (yes/no) [yes]
Search for package to add (or leave blank to continue):

Generated file

[tool.poetry]
name = "hke"
version = "0.1.0"
description = "Home Kubernetes Engine"
authors = ["lupinthe14th <hideosuzuki@ordinarius-fectum.net>"]
license = "MIT"

[tool.poetry.dependencies]
python = "^3.8"

[tool.poetry.dev-dependencies]

[build-system]
requires = ["poetry>=0.12"]
build-backend = "poetry.masonry.api"


Do you confirm generation? (yes/no) [yes]
```


### poetry add

ansibleを追加します。
仮想環境はなければ勝手に追加してくれます。

```fish
poetry add ansible
Creating virtualenv hke-0Ffczry6-py3.8 in /Users/hideoSuzuki/Library/Caches/pypoetry/virtualenvs
Using version ^2.9.9 for ansible

Updating dependencies
Resolving dependencies... (19.9s)

Writing lock file


Package operations: 1 install, 0 updates, 0 removals

  - Installing ansible (2.9.9)

```
