---
title: "Ansibleを使う"
date: 2020-05-13T22:29:13+09:00
tags: ["Memo", "Ansible", "Pipenv"]
draft: false
---

新しくRasberry Pi4b買って初期構築したらなんだかんだと3時間くらいかかりました。
もちろんイメージ作成や、キッティングも含めてなので仕方ない部分もありますが作業完了後、結構時間かかったなと思ったのが正直なところでして。

少しでも楽するべく[Ansible](https://github.com/ansible/ansible)で構築できる様にします。

これからも少しずつ台数増やすでしょうし、今まで構築したのがいろいろバラバラになりつつあるので再構築が楽になるでしょう。

## Preparation

### Install

AnsibleをMacBookにインストールして[Control node](https://docs.ansible.com/ansible/latest/user_guide/basic_concepts.html#control-node)とします。
依存関係を排除したいのでパッケージマネージャー[Pipenv](https://github.com/pypa/pipenv)を使います。

#### Pipenv

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

#### Ansible

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

これで[Control node](https://docs.ansible.com/ansible/latest/user_guide/basic_concepts.html#control-node)へのAnsibleのインストールが完了しました。


## Configuration

### [Inventory](https://docs.ansible.com/ansible/latest/user_guide/basic_concepts.html#inventory)の作成

管理対象ノードのリストを作成します。

```fish
vi inventory
```

```
[target]
knalarmpi2001
knalarmpi3001
knalarmpi4001
knalarmpi4002
kmalarmpi3b01
kmalarmpi4b01
```

### [Managed nodes](https://docs.ansible.com/ansible/latest/user_guide/basic_concepts.html#managed-nodes)の設定

1. 前提

   管理対象ノードは次の2点を満たす必要があります。

   - [Control node](https://docs.ansible.com/ansible/latest/user_guide/basic_concepts.html#control-node)からSSH接続できる
   - [Managed nodes](https://docs.ansible.com/ansible/latest/user_guide/basic_concepts.html#managed-nodes)にはPythonがインストールされている

1. テスト

   pingモジュールを使用して、インベントリ内のすべてのノードにpingします。

   ```fish
   ansible all -m ping -i inventory
   ```

   <details>
   <summary>実行ログ</summary>

   何やらWARNINGが出ていますが気にしないで進めます。
   ```fish
   [WARNING]: Platform linux on host knalarmpi4002 is using the discovered Python interpreter at /usr/bin/python, but future installation of another Python interpreter could
   change this. See https://docs.ansible.com/ansible/2.9/reference_appendices/interpreter_discovery.html for more information.
   knalarmpi4002 | SUCCESS => {
       "ansible_facts": {
           "discovered_interpreter_python": "/usr/bin/python"
       },
       "changed": false,
       "ping": "pong"
   }
   [WARNING]: Platform linux on host knalarmpi4001 is using the discovered Python interpreter at /usr/bin/python, but future installation of another Python interpreter could
   change this. See https://docs.ansible.com/ansible/2.9/reference_appendices/interpreter_discovery.html for more information.
   knalarmpi4001 | SUCCESS => {
       "ansible_facts": {
           "discovered_interpreter_python": "/usr/bin/python"
       },
       "changed": false,
       "ping": "pong"
   }
   [WARNING]: Platform linux on host kmalarmpi4b01 is using the discovered Python interpreter at /usr/bin/python, but future installation of another Python interpreter could
   change this. See https://docs.ansible.com/ansible/2.9/reference_appendices/interpreter_discovery.html for more information.
   kmalarmpi4b01 | SUCCESS => {
       "ansible_facts": {
           "discovered_interpreter_python": "/usr/bin/python"
       },
       "changed": false,
       "ping": "pong"
   }
   [WARNING]: Platform linux on host knalarmpi3001 is using the discovered Python interpreter at /usr/bin/python, but future installation of another Python interpreter could
   change this. See https://docs.ansible.com/ansible/2.9/reference_appendices/interpreter_discovery.html for more information.
   knalarmpi3001 | SUCCESS => {
       "ansible_facts": {
           "discovered_interpreter_python": "/usr/bin/python"
       },
       "changed": false,
       "ping": "pong"
   }
   [WARNING]: Platform linux on host knalarmpi2001 is using the discovered Python interpreter at /usr/bin/python, but future installation of another Python interpreter could
   change this. See https://docs.ansible.com/ansible/2.9/reference_appendices/interpreter_discovery.html for more information.
   knalarmpi2001 | SUCCESS => {
       "ansible_facts": {
           "discovered_interpreter_python": "/usr/bin/python"
       },
       "changed": false,
       "ping": "pong"
   }
   [WARNING]: Platform linux on host kmalarmpi3b01 is using the discovered Python interpreter at /usr/bin/python, but future installation of another Python interpreter could
   change this. See https://docs.ansible.com/ansible/2.9/reference_appendices/interpreter_discovery.html for more information.
   kmalarmpi3b01 | SUCCESS => {
       "ansible_facts": {
           "discovered_interpreter_python": "/usr/bin/python"
       },
       "changed": false,
       "ping": "pong"
   }
   ```
   </details>

   これで事前準備は完了です。

## 基本設定

### ansible-galaxy

ansible-galaxyを使ってロールを作成します。

1. init

   ```fish
   ansible-galaxy init basic
   ```

1. modify

   [Arch Linux ARM on Rasberry Pi2 の初期設定]({{< ref "/posts/ArchLinux/setting.md" >}})に記載の内容を`./basic/task/main.yml`に定義します。

   ```yaml
   ---
   # tasks file for basic
   - name: Run the equivalent of "pacman -Syu" as a separate step
     pacman:
       update_cache: yes
       upgrade: yes

   - name: Ensure a locale exists
     locale_gen:
       name: ja_JP.UTF-8
       state: present

   - name: Install package vim from repo
     pacman:
       name: vim
       state: present

   - name: Install package tmux from repo
     pacman:
       name: tmux
       state: present

   - name: Install package sudo from repo
     pacman:
       name: sudo
       state: present

   ```

## ansible-playbook

1. Playbookの作成

   作成したロールを呼び出すPlaybookを作成します。

   ```yaml
   ---
   - hosts: all
     become: true
     roles:
       - basic
   ```

1. Playbookの実行

   ```fish
   ansible-playbook -i inventory playbook.yml -K
   ```

   sudoのパスワードの入力を求められますので入力してください。



### 付録

### ansible-galaxy

ansible-galaxyを使ってコミュニティで公開されているロールを利用できます。

1. Search

   archlinuxのロールが無いか検索してみます。

   ```fish
   ansible-galaxy search archlinux


   Found 165 roles matching your search:

    Name                                       Description
    ----                                       -----------
    1davidmichael.ansible-role-nginx           Nginx installation for Linux, FreeBSD and OpenBSD.
    abaez.sudo                                 Sudo user permission structure based on archlinux sudo wiki.
    abaez.user                                 A user client provisioner
    (snip)
    bmcclure.python                            Manages Python 2 and 3 on Archlinux
    bmcclure.radarr                            Installs radarr from the AUR and manages its configuration in Archlinux
    bmcclure.raspberrypi                       An ansible role that requires managed software specifically for raspberry pi variants running Archlinux ARM
    bmcclure.sabnzbd                           Installs sabnzbd from the AUR and manages its configuration in Archlinux
    bmcclure.sonarr                            Installs sonarr from the AUR and manages its configuration in Archlinux
    bmcclure.sudo                              An Ansbile role to handle sudo installation and configuration on Archlinux
    (snip)
   ```

1. Install

   `bmcclure.raspberrypi` を使ってみます。
   ロールを好きなパスにダウンロードする場合は、`--roles-path`オプションにパスを指定します。

   ```fish
   ansible-galaxy install --roles-path . bmcclure.raspberrypi
   ```
