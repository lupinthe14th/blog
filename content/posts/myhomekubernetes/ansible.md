---
title: "Ansible〜準備編〜"
date: 2020-05-13T22:29:13+09:00
tags:
- Memo
- Ansible
- Pipenv
draft: false
---

新しくRasberry Pi4b買って初期構築したらなんだかんだと3時間くらいかかりました。
もちろんイメージ作成や、キッティングも含めてなので仕方ない部分もありますが作業完了後、結構時間かかったなと思ったのが正直なところでして。

少しでも楽するべく[Ansible](https://github.com/ansible/ansible)で構築できる様にします。

これからも少しずつ台数増やすでしょうし、今まで構築したのがいろいろバラバラになりつつあるので再構築が楽になるでしょう。

## Install

AnsibleはMacBookにインストールします。
パッケージマネージャーに[Pipenv](https://github.com/pypa/pipenv)を使って依存関係を排除します。

### Pipenv

```fish
pipenv shell
Creating a virtualenv for this project…
Pipfile: /Users/hideoSuzuki/Documents/git/hke/Pipfile
Using /Users/hideoSuzuki/.pyenv/versions/3.8.2/bin/python3.8 (3.8.2) to create virtualenv…
⠙ Creating virtual environment...created virtual environment CPython3.8.2.final.0-64 in 787ms
  creator CPython3Posix(dest=/Users/hideoSuzuki/.local/share/virtualenvs/hke-6JtdMdy9, clear=False, global=False)
  seeder FromAppData(download=False, pip=latest, setuptools=latest, wheel=latest, via=copy, app_data_dir=/Users/hideoSuzuki/Library/Application Support/virtualenv/seed-app-data/v1.0.1)
  activators BashActivator,CShellActivator,FishActivator,PowerShellActivator,PythonActivator,XonshActivator

✔ Successfully created virtual environment!
Virtualenv location: /Users/hideoSuzuki/.local/share/virtualenvs/hke-6JtdMdy9
Creating a Pipfile for this project…
Launching subshell in virtual environment…
 source /Users/hideoSuzuki/.local/share/virtualenvs/hke-6JtdMdy9/bin/activate.fish
~/Documents/git/hke master
❯  source /Users/hideoSuzuki/.local/share/virtualenvs/hke-6JtdMdy9/bin/activate.fish

```

### Ansible

```fish
pipenv install ansible

Installing ansible…
Adding ansible to Pipfile's [packages]…
✔ Installation Succeeded
Pipfile.lock not found, creating…
Locking [dev-packages] dependencies…
Locking [packages] dependencies…
✔ Success!
Updated Pipfile.lock (5bd702)!
Installing dependencies from Pipfile.lock (5bd702)…
  🐍   ▉▉▉▉▉▉▉▉▉▉▉▉▉▉▉▉▉▉▉▉▉▉▉▉▉▉▉▉▉▉▉▉ 8/8 — 00:00:04

```

```fish
ansible --version
ansible 2.9.9
  config file = None
  configured module search path = ['/Users/hideoSuzuki/.ansible/plugins/modules', '/usr/share/ansible/plugins/modules']
  ansible python module location = /Users/hideoSuzuki/.local/share/virtualenvs/hke-6JtdMdy9/lib/python3.8/site-packages/ansible
  executable location = /Users/hideoSuzuki/.local/share/virtualenvs/hke-6JtdMdy9/bin/ansible
  python version = 3.8.2 (default, Mar 25 2020, 14:06:55) [Clang 11.0.0 (clang-1100.0.33.17)]

```

これでAnsibleのインストールが完了しました。
