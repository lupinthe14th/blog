---
title: "Kubernetes Trouble Shooting"
date: 2020-05-16T07:52:20+09:00
tags: ["Kubernetes", "Memo", "arm", "single board computer", "version upgrade"]
draft: false
---

## kubeletが起動しない

### きっかけ

`sudo pacman -Syu`でパッケージをアップデートした後に起動しなくなった。

### ログ

何はともあれログを確認します。


```fish
journalctl -u kubelet -S "2020-05-15 21:29:21" -U "2020-05-15 21:29:29"
-- Logs begin at Wed 2020-04-22 20:45:40 JST, end at Sat 2020-05-16 08:52:36 JST. --
May 15 21:29:21 knalarmpi2001 systemd[1]: kubelet.service: Scheduled restart job, restart counter is at 128.
May 15 21:29:21 knalarmpi2001 systemd[1]: Stopped kubelet: The Kubernetes Node Agent.
May 15 21:29:21 knalarmpi2001 systemd[1]: Started kubelet: The Kubernetes Node Agent.
May 15 21:29:21 knalarmpi2001 kubelet[6810]: Flag --resolv-conf has been deprecated, This parameter should be set via the config file specified by the Kubelet's --config flag. See https://kubernetes.io/docs/tasks/administer-cluster/kubelet-config-file/ for more information.
May 15 21:29:21 knalarmpi2001 kubelet[6810]: Flag --cgroup-driver has been deprecated, This parameter should be set via the config file specified by the Kubelet's --config flag. See https://kubernetes.io/docs/tasks/administer-cluster/kubelet-config-file/ for more information.
May 15 21:29:21 knalarmpi2001 kubelet[6810]: Flag --resolv-conf has been deprecated, This parameter should be set via the config file specified by the Kubelet's --config flag. See https://kubernetes.io/docs/tasks/administer-cluster/kubelet-config-file/ for more information.
May 15 21:29:21 knalarmpi2001 kubelet[6810]: Flag --cgroup-driver has been deprecated, This parameter should be set via the config file specified by the Kubelet's --config flag. See https://kubernetes.io/docs/tasks/administer-cluster/kubelet-config-file/ for more information.
May 15 21:29:22 knalarmpi2001 kubelet[6810]: I0515 21:29:22.017327    6810 server.go:417] Version: v1.18.2
May 15 21:29:22 knalarmpi2001 kubelet[6810]: I0515 21:29:22.019606    6810 plugins.go:100] No cloud provider specified.
May 15 21:29:22 knalarmpi2001 kubelet[6810]: I0515 21:29:22.019810    6810 server.go:837] Client rotation is on, will bootstrap in background
May 15 21:29:22 knalarmpi2001 kubelet[6810]: I0515 21:29:22.047136    6810 certificate_store.go:130] Loading cert/key pair from "/var/lib/kubelet/pki/kubelet-client-current.pem".
May 15 21:29:22 knalarmpi2001 kubelet[6810]: W0515 21:29:22.061659    6810 server.go:615] failed to get the kubelet's cgroup: mountpoint for memory not found.  Kubelet system container metrics may be missing.
May 15 21:29:22 knalarmpi2001 kubelet[6810]: E0515 21:29:22.073222    6810 machine.go:331] failed to get cache information for node 0: open /sys/devices/system/cpu/cpu0/cache: no such file or directory
May 15 21:29:22 knalarmpi2001 kubelet[6810]: I0515 21:29:22.076499    6810 server.go:646] --cgroups-per-qos enabled, but --cgroup-root was not specified.  defaulting to /
May 15 21:29:22 knalarmpi2001 kubelet[6810]: I0515 21:29:22.078666    6810 container_manager_linux.go:266] container manager verified user specified cgroup-root exists: []
May 15 21:29:22 knalarmpi2001 kubelet[6810]: I0515 21:29:22.078807    6810 container_manager_linux.go:271] Creating Container Manager object based on Node Config: {RuntimeCgroupsName: SystemCgroupsName: KubeletCgroupsName: ContainerRuntime:remote CgroupsPerQOS:true CgroupRoot:/ CgroupDriver:systemd KubeletRootDir:/var/lib/kubelet ProtectKernelDefaults:false NodeAllocatableConfig:{KubeReservedCgroupName: SystemReservedCgroupName: ReservedSystemCPUs: EnforceNodeAllocatable:map[pods:{}] KubeReserved:map[] SystemReserved:map[] HardEvictionThresholds:[{Signal:imagefs.available Operator:LessThan Value:{Quantity:<nil> Percentage:0.15} GracePeriod:0s MinReclaim:<nil>} {Signal:memory.available Operator:LessThan Value:{Quantity:100Mi Percentage:0} GracePeriod:0s MinReclaim:<nil>} {Signal:nodefs.available Operator:LessThan Value:{Quantity:<nil> Percentage:0.1} GracePeriod:0s MinReclaim:<nil>} {Signal:nodefs.inodesFree Operator:LessThan Value:{Quantity:<nil> Percentage:0.05} GracePeriod:0s MinReclaim:<nil>}]} QOSReserved:map[] ExperimentalCPUManagerPolicy:none ExperimentalCPUManagerReconcilePeriod:10s ExperimentalPodPidsLimit:-1 EnforceCPULimits:true CPUCFSQuotaPeriod:100ms ExperimentalTopologyManagerPolicy:none}
May 15 21:29:22 knalarmpi2001 kubelet[6810]: I0515 21:29:22.079708    6810 topology_manager.go:126] [topologymanager] Creating topology manager with none policy
May 15 21:29:22 knalarmpi2001 kubelet[6810]: I0515 21:29:22.079810    6810 container_manager_linux.go:301] [topologymanager] Initializing Topology Manager with none policy
May 15 21:29:22 knalarmpi2001 kubelet[6810]: I0515 21:29:22.079872    6810 container_manager_linux.go:306] Creating device plugin manager: true
May 15 21:29:22 knalarmpi2001 kubelet[6810]: W0515 21:29:22.080475    6810 util_unix.go:103] Using "/run/containerd/containerd.sock" as endpoint is deprecated, please consider using full url format "unix:///run/containerd/containerd.sock".
May 15 21:29:22 knalarmpi2001 kubelet[6810]: I0515 21:29:22.080929    6810 remote_runtime.go:59] parsed scheme: ""
May 15 21:29:22 knalarmpi2001 kubelet[6810]: I0515 21:29:22.081012    6810 remote_runtime.go:59] scheme "" not registered, fallback to default scheme
May 15 21:29:22 knalarmpi2001 kubelet[6810]: I0515 21:29:22.081299    6810 passthrough.go:48] ccResolverWrapper: sending update to cc: {[{/run/containerd/containerd.sock  <nil> 0 <nil>}] <nil> <nil>}
May 15 21:29:22 knalarmpi2001 kubelet[6810]: I0515 21:29:22.081387    6810 clientconn.go:933] ClientConn switching balancer to "pick_first"
May 15 21:29:22 knalarmpi2001 kubelet[6810]: W0515 21:29:22.081725    6810 util_unix.go:103] Using "/run/containerd/containerd.sock" as endpoint is deprecated, please consider using full url format "unix:///run/containerd/containerd.sock".
May 15 21:29:22 knalarmpi2001 kubelet[6810]: I0515 21:29:22.081913    6810 remote_image.go:50] parsed scheme: ""
May 15 21:29:22 knalarmpi2001 kubelet[6810]: I0515 21:29:22.082021    6810 remote_image.go:50] scheme "" not registered, fallback to default scheme
May 15 21:29:22 knalarmpi2001 kubelet[6810]: I0515 21:29:22.082158    6810 passthrough.go:48] ccResolverWrapper: sending update to cc: {[{/run/containerd/containerd.sock  <nil> 0 <nil>}] <nil> <nil>}
May 15 21:29:22 knalarmpi2001 kubelet[6810]: I0515 21:29:22.082276    6810 clientconn.go:933] ClientConn switching balancer to "pick_first"
May 15 21:29:22 knalarmpi2001 kubelet[6810]: I0515 21:29:22.082869    6810 kubelet.go:292] Adding pod path: /etc/kubernetes/manifests
May 15 21:29:22 knalarmpi2001 kubelet[6810]: I0515 21:29:22.083198    6810 kubelet.go:317] Watching apiserver
May 15 21:29:28 knalarmpi2001 kubelet[6810]: E0515 21:29:28.390431    6810 aws_credentials.go:77] while getting AWS credentials NoCredentialProviders: no valid providers in chain. Deprecated.
May 15 21:29:28 knalarmpi2001 kubelet[6810]:         For verbose messaging see aws.Config.CredentialsChainVerboseErrors
May 15 21:29:28 knalarmpi2001 kubelet[6810]: I0515 21:29:28.395512    6810 kuberuntime_manager.go:211] Container runtime containerd initialized, version: v1.3.4.m, apiVersion: v1alpha2
May 15 21:29:28 knalarmpi2001 kubelet[6810]: I0515 21:29:28.400056    6810 server.go:1125] Started kubelet
May 15 21:29:28 knalarmpi2001 kubelet[6810]: I0515 21:29:28.408115    6810 server.go:145] Starting to listen on 0.0.0.0:10250
May 15 21:29:28 knalarmpi2001 kubelet[6810]: E0515 21:29:28.409431    6810 cri_stats_provider.go:375] Failed to get the info of the filesystem with mountpoint "/var/lib/containerd/io.containerd.snapshotter.v1.overlayfs": unable to find data in memory cache.
May 15 21:29:28 knalarmpi2001 kubelet[6810]: E0515 21:29:28.409681    6810 kubelet.go:1305] Image garbage collection failed once. Stats initialization may not have completed yet: invalid capacity 0 on image filesystem
May 15 21:29:28 knalarmpi2001 kubelet[6810]: I0515 21:29:28.415931    6810 fs_resource_analyzer.go:64] Starting FS ResourceAnalyzer
May 15 21:29:28 knalarmpi2001 kubelet[6810]: I0515 21:29:28.425632    6810 server.go:393] Adding debug handlers to kubelet server.
May 15 21:29:28 knalarmpi2001 kubelet[6810]: I0515 21:29:28.437824    6810 volume_manager.go:265] Starting Kubelet Volume Manager
May 15 21:29:28 knalarmpi2001 kubelet[6810]: I0515 21:29:28.442385    6810 desired_state_of_world_populator.go:139] Desired state populator starts to run
May 15 21:29:28 knalarmpi2001 kubelet[6810]: I0515 21:29:28.467209    6810 clientconn.go:106] parsed scheme: "unix"
May 15 21:29:28 knalarmpi2001 kubelet[6810]: I0515 21:29:28.469070    6810 clientconn.go:106] scheme "unix" not registered, fallback to default scheme
May 15 21:29:28 knalarmpi2001 kubelet[6810]: I0515 21:29:28.471059    6810 passthrough.go:48] ccResolverWrapper: sending update to cc: {[{unix:///run/containerd/containerd.sock  <nil> 0 <nil>}] <nil> <nil>}
May 15 21:29:28 knalarmpi2001 kubelet[6810]: I0515 21:29:28.472944    6810 clientconn.go:933] ClientConn switching balancer to "pick_first"
May 15 21:29:28 knalarmpi2001 kubelet[6810]: I0515 21:29:28.546612    6810 kubelet_node_status.go:294] Setting node annotation to enable volume controller attach/detach
May 15 21:29:28 knalarmpi2001 kubelet[6810]: I0515 21:29:28.548289    6810 kuberuntime_manager.go:978] updating runtime config through cri with podcidr 10.244.1.0/24
May 15 21:29:28 knalarmpi2001 kubelet[6810]: I0515 21:29:28.562789    6810 kubelet_network.go:77] Setting Pod CIDR:  -> 10.244.1.0/24
May 15 21:29:28 knalarmpi2001 kubelet[6810]: I0515 21:29:28.565886    6810 kubelet_node_status.go:70] Attempting to register node knalarmpi2001
May 15 21:29:28 knalarmpi2001 kubelet[6810]: I0515 21:29:28.621991    6810 kubelet_node_status.go:112] Node knalarmpi2001 was previously registered
May 15 21:29:28 knalarmpi2001 kubelet[6810]: I0515 21:29:28.623839    6810 kubelet_node_status.go:73] Successfully registered node knalarmpi2001
May 15 21:29:28 knalarmpi2001 kubelet[6810]: I0515 21:29:28.694076    6810 status_manager.go:158] Starting to sync pod status with apiserver
May 15 21:29:28 knalarmpi2001 kubelet[6810]: I0515 21:29:28.694384    6810 kubelet.go:1821] Starting kubelet main sync loop.
May 15 21:29:28 knalarmpi2001 kubelet[6810]: E0515 21:29:28.694783    6810 kubelet.go:1845] skipping pod synchronization - [container runtime status check may not have completed yet, PLEG is not healthy: pleg has yet to be successful]
May 15 21:29:28 knalarmpi2001 kubelet[6810]: I0515 21:29:28.756036    6810 cpu_manager.go:184] [cpumanager] starting with none policy
May 15 21:29:28 knalarmpi2001 kubelet[6810]: I0515 21:29:28.756251    6810 cpu_manager.go:185] [cpumanager] reconciling every 10s
May 15 21:29:28 knalarmpi2001 kubelet[6810]: I0515 21:29:28.756504    6810 state_mem.go:36] [cpumanager] initializing new in-memory state store
May 15 21:29:28 knalarmpi2001 kubelet[6810]: I0515 21:29:28.757833    6810 state_mem.go:88] [cpumanager] updated default cpuset: ""
May 15 21:29:28 knalarmpi2001 kubelet[6810]: I0515 21:29:28.758065    6810 state_mem.go:96] [cpumanager] updated cpuset assignments: "map[]"
May 15 21:29:28 knalarmpi2001 kubelet[6810]: I0515 21:29:28.761260    6810 policy_none.go:43] [cpumanager] none policy: Start
May 15 21:29:28 knalarmpi2001 kubelet[6810]: F0515 21:29:28.765602    6810 kubelet.go:1383] Failed to start ContainerManager system validation failed - Following Cgroup subsystem not mounted: [memory]
May 15 21:29:28 knalarmpi2001 systemd[1]: kubelet.service: Main process exited, code=exited, status=255/EXCEPTION
May 15 21:29:28 knalarmpi2001 systemd[1]: kubelet.service: Failed with result 'exit-code'.
```

`Failed to start ContainerManager system validation failed - Following Cgroup subsystem not mounted: [memory]`とのことです。


### 調査

1. 事象発生機器

   Raspberry Pi 2 Model B Rev 1.1

1. cgroupsの現在値を確認

   `memory` が有効になっていません。

   ```fish
   cat /proc/cgroups
   #subsys_name    hierarchy       num_cgroups     enabled
   cpuset  3       1       1
   cpu     6       22      1
   cpuacct 6       22      1
   blkio   7       22      1
   memory  0       24      0
   devices 4       22      1
   freezer 8       1       1
   net_cls 2       1       1
   perf_event      5       1       1
   net_prio        2       1       1
   pids    9       23      1
   ```

1. カーネルの設定値を確認

   カーネルの設定値では有効になっています。

   ```fish
   zgrep -i cgroup /proc/config.gz
   CONFIG_CGROUPS=y
   CONFIG_BLK_CGROUP=y
   CONFIG_CGROUP_WRITEBACK=y
   CONFIG_CGROUP_SCHED=y
   CONFIG_CGROUP_PIDS=y
   # CONFIG_CGROUP_RDMA is not set
   CONFIG_CGROUP_FREEZER=y
   CONFIG_CGROUP_DEVICE=y
   CONFIG_CGROUP_CPUACCT=y
   CONFIG_CGROUP_PERF=y
   CONFIG_CGROUP_BPF=y
   # CONFIG_CGROUP_DEBUG is not set
   CONFIG_SOCK_CGROUP_DATA=y
   # CONFIG_BLK_CGROUP_IOLATENCY is not set
   # CONFIG_BLK_CGROUP_IOCOST is not set
   # CONFIG_BFQ_CGROUP_DEBUG is not set
   CONFIG_NETFILTER_XT_MATCH_CGROUP=m
   CONFIG_NET_CLS_CGROUP=m
   CONFIG_CGROUP_NET_PRIO=y
   CONFIG_CGROUP_NET_CLASSID=y
   ```

### 対策

`cgroup_enable=cpuset cgroup_enable=memory` を `/boot/cmdline.txt` に追記してシステム再起動します。

```text
cat /boot/cmdline.txt
root=/dev/mmcblk0p2 rw rootwait console=tty1 selinux=0 plymouth.enable=0 smsc95xx.turbo_mode=N dwc_otg.lpm_enable=0 kgdboc=ttyAMA0,115200 elevator=noop cgroup_enable=cpuset cgroup_enable=memory
```

```fish
sudo reboot
```

### 確認

kubeletは起動する様になりました。
別のエラーが発生していますが、それはまた別に調査します。たぶん。


```fish
● kubelet.service - kubelet: The Kubernetes Node Agent
     Loaded: loaded (/etc/systemd/system/kubelet.service; enabled; vendor preset: disabled)
    Drop-In: /etc/systemd/system/kubelet.service.d
             └─0-containerd.conf, 10-kubeadm.conf
     Active: active (running) since Sat 2020-05-16 10:27:18 JST; 18min ago
       Docs: http://kubernetes.io/docs/
   Main PID: 331 (kubelet)
      Tasks: 23 (limit: 2155)
     Memory: 30.3M
     CGroup: /system.slice/kubelet.service
             └─331 /usr/bin/kubelet --bootstrap-kubeconfig=/etc/kubernetes/bootstrap-kubelet.conf --kubeconfig=/etc/kubernetes/kubelet.conf --config=/var/lib/kubelet/config.yaml --container-runtime=remote --container-runtime-endpoint=/run/containerd/containerd.sock --resolv-conf=/run/systemd/resolve/resolv.conf --cgroup-driver=systemd

May 16 10:43:53 knalarmpi2001 kubelet[331]: E0516 10:43:53.862557     331 remote_runtime.go:128] StopPodSandbox "c0efcf96eafd018ea1586bf3a7eb6d2cbd24b0beb93cb4330eed51b4409a0786" from runtime service failed: rpc error: code = Unknown desc = failed to destroy network for sandbox "c0efcf96eafd018ea1586bf3a7eb6d2cbd24b0beb93cb4330eed51b4409a0786": invalid version "": the version is empty
May 16 10:43:53 knalarmpi2001 kubelet[331]: E0516 10:43:53.862912     331 kuberuntime_gc.go:170] Failed to stop sandbox "c0efcf96eafd018ea1586bf3a7eb6d2cbd24b0beb93cb4330eed51b4409a0786" before removing: rpc error: code = Unknown desc = failed to destroy network for sandbox "c0efcf96eafd018ea1586bf3a7eb6d2cbd24b0beb93cb4330eed51b4409a0786": invalid version "": the version is empty
May 16 10:44:53 knalarmpi2001 kubelet[331]: E0516 10:44:53.899819     331 remote_runtime.go:128] StopPodSandbox "5bd1aae3aa8f368bb37677a68d9087d4ae123683a792b90422b6d6101adc3e03" from runtime service failed: rpc error: code = Unknown desc = failed to destroy network for sandbox "5bd1aae3aa8f368bb37677a68d9087d4ae123683a792b90422b6d6101adc3e03": invalid version "": the version is empty
May 16 10:44:53 knalarmpi2001 kubelet[331]: E0516 10:44:53.900124     331 kuberuntime_gc.go:170] Failed to stop sandbox "5bd1aae3aa8f368bb37677a68d9087d4ae123683a792b90422b6d6101adc3e03" before removing: rpc error: code = Unknown desc = failed to destroy network for sandbox "5bd1aae3aa8f368bb37677a68d9087d4ae123683a792b90422b6d6101adc3e03": invalid version "": the version is empty
May 16 10:44:53 knalarmpi2001 kubelet[331]: E0516 10:44:53.925654     331 remote_runtime.go:128] StopPodSandbox "7a65f7c09fca749168611c0911df99554280f332c965265958cef987b3bf3f7b" from runtime service failed: rpc error: code = Unknown desc = failed to destroy network for sandbox "7a65f7c09fca749168611c0911df99554280f332c965265958cef987b3bf3f7b": invalid version "": the version is empty
May 16 10:44:53 knalarmpi2001 kubelet[331]: E0516 10:44:53.925948     331 kuberuntime_gc.go:170] Failed to stop sandbox "7a65f7c09fca749168611c0911df99554280f332c965265958cef987b3bf3f7b" before removing: rpc error: code = Unknown desc = failed to destroy network for sandbox "7a65f7c09fca749168611c0911df99554280f332c965265958cef987b3bf3f7b": invalid version "": the version is empty
May 16 10:44:53 knalarmpi2001 kubelet[331]: E0516 10:44:53.951488     331 remote_runtime.go:128] StopPodSandbox "4ddd6c8467c94dbe75941b2bee84020475d0de678fd2a871ab015af64b1edb01" from runtime service failed: rpc error: code = Unknown desc = failed to destroy network for sandbox "4ddd6c8467c94dbe75941b2bee84020475d0de678fd2a871ab015af64b1edb01": invalid version "": the version is empty
May 16 10:44:53 knalarmpi2001 kubelet[331]: E0516 10:44:53.951791     331 kuberuntime_gc.go:170] Failed to stop sandbox "4ddd6c8467c94dbe75941b2bee84020475d0de678fd2a871ab015af64b1edb01" before removing: rpc error: code = Unknown desc = failed to destroy network for sandbox "4ddd6c8467c94dbe75941b2bee84020475d0de678fd2a871ab015af64b1edb01": invalid version "": the version is empty
May 16 10:44:53 knalarmpi2001 kubelet[331]: E0516 10:44:53.977624     331 remote_runtime.go:128] StopPodSandbox "c0efcf96eafd018ea1586bf3a7eb6d2cbd24b0beb93cb4330eed51b4409a0786" from runtime service failed: rpc error: code = Unknown desc = failed to destroy network for sandbox "c0efcf96eafd018ea1586bf3a7eb6d2cbd24b0beb93cb4330eed51b4409a0786": invalid version "": the version is empty
May 16 10:44:53 knalarmpi2001 kubelet[331]: E0516 10:44:53.977855     331 kuberuntime_gc.go:170] Failed to stop sandbox "c0efcf96eafd018ea1586bf3a7eb6d2cbd24b0beb93cb4330eed51b4409a0786" before removing: rpc error: code = Unknown desc = failed to destroy network for sandbox "c0efcf96eafd018ea1586bf3a7eb6d2cbd24b0beb93cb4330eed51b4409a0786": invalid version "": the version is empty
```

ノードの状態も　NotReadyからReadyに変わりました。

```fish
kubectl get node
NAME            STATUS   ROLES    AGE    VERSION
kmalarmpi3b01   Ready    master   254d   v1.18.2
knalarmpi2001   Ready    worker   254d   v1.18.2
knalarmpi3001   Ready    worker   254d   v1.18.2
```

システム再起動後、cgroupsの値を確認します。

`memory`が有効になりました。

```fish
cat /proc/cgroups
#subsys_name    hierarchy       num_cgroups     enabled
cpuset  3       13      1
cpu     7       35      1
cpuacct 7       35      1
blkio   8       35      1
memory  2       52      1
devices 5       35      1
freezer 9       13      1
net_cls 4       13      1
perf_event      6       13      1
net_prio        4       13      1
pids    10      36      1
```

カーネルの設定値は変わりありません。

```fish
zgrep -i cgroup /proc/config.gz
CONFIG_CGROUPS=y
CONFIG_BLK_CGROUP=y
CONFIG_CGROUP_WRITEBACK=y
CONFIG_CGROUP_SCHED=y
CONFIG_CGROUP_PIDS=y
# CONFIG_CGROUP_RDMA is not set
CONFIG_CGROUP_FREEZER=y
CONFIG_CGROUP_DEVICE=y
CONFIG_CGROUP_CPUACCT=y
CONFIG_CGROUP_PERF=y
CONFIG_CGROUP_BPF=y
# CONFIG_CGROUP_DEBUG is not set
CONFIG_SOCK_CGROUP_DATA=y
# CONFIG_BLK_CGROUP_IOLATENCY is not set
# CONFIG_BLK_CGROUP_IOCOST is not set
# CONFIG_BFQ_CGROUP_DEBUG is not set
CONFIG_NETFILTER_XT_MATCH_CGROUP=m
CONFIG_NET_CLS_CGROUP=m
CONFIG_CGROUP_NET_PRIO=y
CONFIG_CGROUP_NET_CLASSID=y
```

SeeAlso
: - [enabling cgroup memory doesn't take effect](https://www.raspberrypi.org/forums/viewtopic.php?t=203128)
: - [カーネル/コンパイル/伝統的な方法 - ArchWiki](https://wiki.archlinux.jp/index.php/カーネル/コンパイル/伝統的な方法)


## kubelet Failed to stop sandbox

```
May 16 10:44:53 knalarmpi2001 kubelet[331]: E0516 10:44:53.977855     331 kuberuntime_gc.go:170] Failed to stop sandbox "c0efcf96eafd018ea1586bf3a7eb6d2cbd24b0beb93cb4330eed51b4409a0786" before removing: rpc error: code = Unknown desc = failed to destroy network for sandbox "c0efcf96eafd018ea1586bf3a7eb6d2cbd24b0beb93cb4330eed51b4409a0786": invalid version "": the version is empty
```

と1分毎に出力されている。


### 回避策

次のNotReadyなPodを削除すると出力され続けていたエラーはなくなりました。[^1]

```fish
sudo crictl pods
POD ID              CREATED             STATE               NAME                        NAMESPACE           ATTEMPT
6904ca3ce6607       2 hours ago         Ready               kube-flannel-ds-arm-d765v   kube-system         7
053a20f498929       2 hours ago         Ready               speaker-8n6vm               metallb-system      2
c470f85ef4a8e       2 hours ago         Ready               kube-proxy-2dklw            kube-system         2
3612798fd46c5       15 hours ago        NotReady            kube-flannel-ds-arm-d765v   kube-system         6
0d295c31a716e       15 hours ago        NotReady            speaker-8n6vm               metallb-system      1
3d75c158b98d2       15 hours ago        NotReady            kube-proxy-2dklw            kube-system         1
```

```fish
sudo crictl rmp 3d75c158b98d2 0d295c31a716e 3d75c158b98d2
```

[^1]: このNotReadyなPodsがあってもエラーが出力されないことも確認しています。
