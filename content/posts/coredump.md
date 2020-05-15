---
title: "Coredumpでお亡くなり"
date: 2020-05-15T10:56:41+09:00
tags: ["coredump", "archlinux", "Nanopc T4"]
draft: false
---

## きっかけ
Ansibleで管理を楽にしようとしてインベントリに対して`pacman -Syu`を実行させたら特定のノードでSSH接続がタイムアウトになり始めた。

なんだろなと思いながら、何度かリトライしましたが確実に再現することがわかりました。

原因を調べるため`journalctl`でログを確認します。


```fish
journalctl -S 2020-05-15
(snip)
 5月 15 09:53:55 knalarmpi4001 systemd[1]: Started Process Core Dump (PID 968/UID 0).
 5月 15 09:53:55 knalarmpi4001 audit[1]: SERVICE_START pid=1 uid=0 auid=4294967295 ses=4294967295 msg='unit=systemd-coredump@13-968-0 comm="systemd" exe="/usr/lib/systemd/systemd" hostname=? addr=? terminal=? res=success'
 5月 15 09:53:55 knalarmpi4001 audit: AUDIT1334 prog-id=40 op=UNLOAD
 5月 15 09:53:55 knalarmpi4001 audit: AUDIT1334 prog-id=39 op=UNLOAD
 5月 15 09:53:55 knalarmpi4001 systemd-coredump[973]: Process 967 (dmidecode) of user 0 dumped core.

                                                       Stack trace of thread 967:
                                                       #0  0x0000ffff8254c528 memcpy (libc.so.6 + 0x81528)
                                                       #1  0x0000aaaadd105cd8 n/a (dmidecode + 0x11cd8)
                                                       #2  0x0000aaaadd0fbfe8 n/a (dmidecode + 0x7fe8)
                                                       #3  0x0000ffff824eed90 __libc_start_main (libc.so.6 + 0x23d90)
                                                       #4  0x0000aaaadd0fc230 n/a (dmidecode + 0x8230)
                                                       #5  0x0000aaaadd0fc230 n/a (dmidecode + 0x8230)
-- Reboot --
(snip)
```


coredumpしてリブートしている…。久しぶりにコアを吐いてお亡くなり状態をみた。

## coredumpを確認

coredump解析か〜と思いつつ、coreを確認します。

```fish
coredumpctl list
TIME                            PID   UID   GID SIG COREFILE  EXE
Sun 2019-12-22 02:19:05 JST    2955     0     0   6 missing   /usr/bin/fio
Fri 2020-05-15 02:47:19 JST     629     0     0   7 error     /usr/bin/dmidecode
Fri 2020-05-15 02:47:20 JST     645     0     0   7 error     /usr/bin/dmidecode
Fri 2020-05-15 02:47:20 JST     648     0     0   7 error     /usr/bin/dmidecode
Fri 2020-05-15 09:50:53 JST     662     0     0   7 error     /usr/bin/dmidecode
Fri 2020-05-15 09:50:54 JST     677     0     0   7 error     /usr/bin/dmidecode
Fri 2020-05-15 09:50:54 JST     684     0     0   7 error     /usr/bin/dmidecode
Fri 2020-05-15 09:50:54 JST     693     0     0   7 error     /usr/bin/dmidecode
Fri 2020-05-15 09:50:55 JST     704     0     0   7 error     /usr/bin/dmidecode
Fri 2020-05-15 09:50:55 JST     711     0     0   7 error     /usr/bin/dmidecode
Fri 2020-05-15 09:50:55 JST     719     0     0   7 error     /usr/bin/dmidecode
Fri 2020-05-15 09:50:56 JST     726     0     0   7 error     /usr/bin/dmidecode
Fri 2020-05-15 09:50:56 JST     733     0     0   7 error     /usr/bin/dmidecode
Fri 2020-05-15 09:53:53 JST     942     0     0   7 error     /usr/bin/dmidecode
Fri 2020-05-15 09:53:54 JST     945     0     0   7 error     /usr/bin/dmidecode
Fri 2020-05-15 09:53:54 JST     953     0     0   7 error     /usr/bin/dmidecode
Fri 2020-05-15 09:53:55 JST     960     0     0   7 error     /usr/bin/dmidecode
Fri 2020-05-15 09:53:55 JST     967     0     0   7 error     /usr/bin/dmidecode
```

dmidecodeが悪さしている様です。

## 対策と結果

dmidecodeとついでにfioを使うことも無いのでアンインストールします。[^1]

[^1]: SSDの性能テストを行うためにfioとdmidecodeをインストールしたのですがもう必要ありません。

```fish
sudo pacman -Rdd fio dmidecode
```

その後、Ansibleで事象発生したPlaybookを実行しても再現しなくなりました。

## 原因

わかりません。このハードウェアが`dmidecode`をサポートしていない可能性が高いです。[^2]

[^2]: SSDのベンチマークを楽にしようと[ezFIO](https://github.com/earlephilhower/ezfio)を使ったのですが、内部的に`dmidecode`を呼んでいたのでインストールしました。しかし`dmidecode`で欲しい情報が取得できない状態でした。


## 参考文献

- [systemd-coredump環境で暮らす](https://rheb.hatenablog.com/entry/systemd-coredump)
