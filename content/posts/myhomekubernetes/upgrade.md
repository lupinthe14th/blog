---
title: "おうちKubernetes ~ 手作業によるバージョンアップグレード編 ~"
date: 2020-04-04T12:22:54+09:00
tags:
- Kubernetes
- Memo
- arm
- single board computer
- version upgrade
draft: false
---

さて、構築して大分放っておいた [^1] ので、バージョンが現在のlatest release [^2] に比べ大分低いです。

[^1]: 構築して満足して本来の開発を行う目的を見失ってました :cry:
[^2]: [v1.18.2](https://kubernetes.io/docs/setup/release/notes/#v1-18-2)

latest release [^2] にアップグレードしていきます。

## Supported component upgrade order

[Supported component upgrade order](https://kubernetes.io/docs/setup/release/version-skew-policy/#supported-component-upgrade-order)
の通り、1.15.x -> 1.16.x -> 1.17.x -> 1.18.0の順にアップグレードしていきます。

手順を確認すると同じようなことを何度もサーバとノードに対して行うようです。
同じようなことはプログラムにやらせたいですが気持ちもありますが、手順把握のため手動実行します。

## Determine which version to upgrade to

以下の順序でアップグレードします。

1. 1.15.3 -> 1.16.9
1. 1.16.9 -> 1.17.5
1. 1.17.5 -> 1.18.2


SeeAlso
: [Upgrading kubeadm clusters](https://v1-16.docs.kubernetes.io/docs/tasks/administer-cluster/kubeadm/kubeadm-upgrade/#determine-which-version-to-upgrade-to)

## Upgrading control plane nodes

### Upgrade control plane node

まずはサーバ（マスター）を更新します。

1. On your first control plane node, upgrade kubeadm:

    作業前のバージョン

    ```bash
    kubeadm version
    ```

    ```bash
    kubeadm version: &version.Info{Major:"1", Minor:"15", GitVersion:"v1.15.3", GitCommit:"2d3c76f9091b6bec110a5e63777c332469e0cba2", GitTreeState:"clean", BuildDate:"2019-08-19T11:11:18Z", GoVersion:"go1.12.9", Compiler:"gc", Platform:"linux/arm64"}
    ```

    実行ファイルをダウンロードして配置します。

    ```bash
    VERSION="v1.16.9"
    ROLE="server"
    ARCH="arm64"
    curl -LO https://dl.k8s.io/$VERSION/kubernetes-$ROLE-linux-$ARCH.tar.gz
    tar xvfz kubernetes-$ROLE-linux-$ARCH.tar.gz
    install -p -m 755 -t /usr/bin/ kubernetes/$ROLE/bin/kubeadm
    ```

1. Verify that the download works and has the expected version:

    `kubeadm`がバージョンアップしていることを確認します。
    
    ```bash
    kubeadm version
    ```

    ```bash
    kubeadm version: &version.Info{Major:"1", Minor:"16", GitVersion:"v1.16.9", GitCommit:"a17149e1a189050796ced469dbd78d380f2ed5ef", GitTreeState:"clean", BuildDate:"2020-04-16T11:42:30Z", GoVersion:"go1.13.9", Compiler:"gc", Platform:"linux/arm64"}
    ```


1. Drain the control plane node:

    ```bash
    kubectl drain kmalarmpi3b01 --ignore-daemonsets
    ```

    ```bash
    node/kmalarmpi3b01 cordoned
    WARNING: ignoring DaemonSet-managed Pods: kube-system/kube-flannel-ds-arm64-cg2k9, kube-system/kube-proxy-vxk2g
    evicting pod "coredns-5644d7b6d9-5bpc7"
    pod/coredns-5644d7b6d9-5bpc7 evicted
    node/kmalarmpi3b01 evicted
    ```

1. On the control plane node, run:

    ```bash
    sudo kubeadm upgrade plan
    ```

    ```bash
    [upgrade/config] Making sure the configuration is correct:
    [upgrade/config] Reading configuration from the cluster...
    [upgrade/config] FYI: You can look at this config file with 'kubectl -n kube-system get cm kubeadm-config -oyaml'
    [preflight] Running pre-flight checks.
    [upgrade] Making sure the cluster is healthy:
    [upgrade] Fetching available versions to upgrade to
    [upgrade/versions] Cluster version: v1.15.3
    [upgrade/versions] kubeadm version: v1.16.9
    I0426 02:10:41.136664   13515 version.go:251] remote version is much newer: v1.18.2; falling back to: stable-1.16
    [upgrade/versions] Latest stable version: v1.16.9
    [upgrade/versions] Latest version in the v1.15 series: v1.15.11
     
    Components that must be upgraded manually after you have upgraded the control plane with 'kubeadm upgrade apply':
    COMPONENT   CURRENT       AVAILABLE
    Kubelet     3 x v1.15.3   v1.15.11
    
    Upgrade to the latest version in the v1.15 series:
    
    COMPONENT            CURRENT   AVAILABLE
    API Server           v1.15.3   v1.15.11
    Controller Manager   v1.15.3   v1.15.11
    Scheduler            v1.15.3   v1.15.11
    Kube Proxy           v1.15.3   v1.15.11
    CoreDNS              1.3.1     1.6.2
    Etcd                 3.3.10    3.3.10
    
    You can now apply the upgrade by executing the following command:
    
            kubeadm upgrade apply v1.15.11
    
    _____________________________________________________________________
    
    Components that must be upgraded manually after you have upgraded the control plane with 'kubeadm upgrade apply':
    COMPONENT   CURRENT       AVAILABLE
    Kubelet     3 x v1.15.3   v1.16.9
    
    Upgrade to the latest stable version:
    
    COMPONENT            CURRENT   AVAILABLE
    API Server           v1.15.3   v1.16.9
    Controller Manager   v1.15.3   v1.16.9
    Scheduler            v1.15.3   v1.16.9
    Kube Proxy           v1.15.3   v1.16.9
    CoreDNS              1.3.1     1.6.2
    Etcd                 3.3.10    3.3.15-0
    
    You can now apply the upgrade by executing the following command:
    
            kubeadm upgrade apply v1.16.9
    
    _____________________________________________________________________
    ``` 


1. Choose a version to upgrade to, and run the appropriate command.

    1.15.3 -> 1.16.9で上げていくつもりでしたが、v1.15系列のlatestが表示されたので念のためこちらから段階的に上げます・・・


    ```bash
    kubeadm upgrade apply v1.15.11
    ```
   
    ```bash
    [upgrade/config] Making sure the configuration is correct:
    [upgrade/config] Reading configuration from the cluster...
    [upgrade/config] FYI: You can look at this config file with 'kubectl -n kube-system get cm kubeadm-config -oyaml'
    [preflight] Running pre-flight checks.
    [upgrade] Making sure the cluster is healthy:
    [upgrade/version] You have chosen to change the cluster version to "v1.15.11"
    [upgrade/versions] Cluster version: v1.15.3
    [upgrade/versions] kubeadm version: v1.16.9
    [upgrade/version] FATAL: the --version argument is invalid due to these errors:
    
            - Kubeadm version v1.16.9 can only be used to upgrade to Kubernetes version 1.16
    
    Can be bypassed if you pass the --force flag
    To see the stack trace of this error execute with --v=5 or higher
    ```

    が、アップグレード対象のバージョンとkubeadminのバージョンが違うためエラーとなりました。
    元々の予定通り進めることにします。

    ```bash
    kubeadm upgrade apply v1.16.9
    ```

    ```bash
    [upgrade/config] Making sure the configuration is correct:
    [upgrade/config] Reading configuration from the cluster...
    [upgrade/config] FYI: You can look at this config file with 'kubectl -n kube-system get cm kubeadm-config -oyaml'
    [preflight] Running pre-flight checks.
    [upgrade] Making sure the cluster is healthy:
    [upgrade/version] You have chosen to change the cluster version to "v1.16.9"
    [upgrade/versions] Cluster version: v1.15.3
    [upgrade/versions] kubeadm version: v1.16.9
    [upgrade/confirm] Are you sure you want to proceed with the upgrade? [y/N]: y
    [upgrade/prepull] Will prepull images for components [kube-apiserver kube-controller-manager kube-scheduler etcd]
    [upgrade/prepull] Prepulling image for component etcd.
    [upgrade/prepull] Prepulling image for component kube-controller-manager.
    [upgrade/prepull] Prepulling image for component kube-scheduler.
    [upgrade/prepull] Prepulling image for component kube-apiserver.
    [apiclient] Found 0 Pods for label selector k8s-app=upgrade-prepull-etcd
    [apiclient] Found 0 Pods for label selector k8s-app=upgrade-prepull-kube-apiserver
    [apiclient] Found 0 Pods for label selector k8s-app=upgrade-prepull-kube-controller-manager
    [apiclient] Found 0 Pods for label selector k8s-app=upgrade-prepull-kube-scheduler
    [apiclient] Found 1 Pods for label selector k8s-app=upgrade-prepull-kube-apiserver
    [apiclient] Found 1 Pods for label selector k8s-app=upgrade-prepull-kube-controller-manager
    [apiclient] Found 1 Pods for label selector k8s-app=upgrade-prepull-etcd
    [apiclient] Found 1 Pods for label selector k8s-app=upgrade-prepull-kube-scheduler
    [upgrade/prepull] Prepulled image for component etcd.
    [apiclient] Error getting Pods with label selector "k8s-app=upgrade-prepull-kube-controller-manager" [the server was unable to return a response in the time allotted, but may still be processing the request (get pods)]
    [apiclient] Error getting Pods with label selector "k8s-app=upgrade-prepull-kube-apiserver" [the server was unable to return a response in the time allotted, but may still be processing the request (get pods)]
    [apiclient] Error getting Pods with label selector "k8s-app=upgrade-prepull-kube-scheduler" [the server was unable to return a response in the time allotted, but may still be processing the request (get pods)]
    [upgrade/prepull] Prepulled image for component kube-apiserver.
    [upgrade/prepull] Prepulled image for component kube-scheduler.
    [upgrade/prepull] Failed prepulled the images for the control plane components error: unable to cleanup the DaemonSet used for prepulling kube-controller-manager: Delete https://10.9.8.49:6443/apis/apps/v1/namespaces/kube-system/daemonsets/upgrade-prepull-kube-controller-manager: dial tcp 10.9.8.49:6443: connect: connection refused
    To see the stack trace of this error execute with --v=5 or higher
    ```

    うーん、失敗です。なんでだろう。[^3]
    
    [^3]: Rasberyy Pi 3Bのリソースが貧弱なため、Podのデプロイが遅くてタイムアウトしていたようです。気にせずリトライすると成功します。

    ひとまず`kubeadm` のバージョンを1.15.11にして段階的にアップデートしてみます。

    ```bash
    VERSION="v1.15.11"
    ROLE="server"
    ARCH="arm64"
    curl -LO https://dl.k8s.io/$VERSION/kubernetes-$ROLE-linux-$ARCH.tar.gz
    tar xvfz kubernetes-$ROLE-linux-$ARCH.tar.gz
    install -p -m 755 -t /usr/bin/ kubernetes/$ROLE/bin/kubeadm
    ```

    ```bash
    kubeadm version
    ```

    <details>
    <summary>実行ログ</summary>

    ```bash
    kubeadm version: &version.Info{Major:"1", Minor:"15", GitVersion:"v1.15.11", GitCommit:"d94a81c724ea8e1ccc9002d89b7fe81d58f89ede", GitTreeState:"clean", BuildDate:"2020-03-12T21:06:11Z", GoVersion:"go1.12.17", Compiler:"gc", Platform:"linux/arm64"}
    ```
    </displays>

    ```bash
    kubeadm upgrade plan
    ```

    <details>
    <summary>実行ログ</summary>

    ```bash
    [upgrade/config] Making sure the configuration is correct:
    [upgrade/config] Reading configuration from the cluster...
    [upgrade/config] FYI: You can look at this config file with 'kubectl -n kube-system get cm kubeadm-config -oyaml'
    [preflight] Running pre-flight checks.
    [upgrade] Making sure the cluster is healthy:
    [upgrade] Fetching available versions to upgrade to
    [upgrade/versions] Cluster version: v1.15.3
    [upgrade/versions] kubeadm version: v1.15.11
    I0426 02:48:09.548913    8438 version.go:248] remote version is much newer: v1.18.2; falling back to: stable-1.15
    [upgrade/versions] Latest stable version: v1.15.11
    [upgrade/versions] Latest version in the v1.15 series: v1.15.11
    
    Components that must be upgraded manually after you have upgraded the control plane with 'kubeadm upgrade apply':
    COMPONENT   CURRENT       AVAILABLE
    Kubelet     3 x v1.15.3   v1.15.11
    
    Upgrade to the latest version in the v1.15 series:
    
    COMPONENT            CURRENT   AVAILABLE
    API Server           v1.15.3   v1.15.11
    Controller Manager   v1.15.3   v1.15.11
    Scheduler            v1.15.3   v1.15.11
    Kube Proxy           v1.15.3   v1.15.11
    CoreDNS              1.3.1     1.3.1
    Etcd                 3.3.10    3.3.10
    
    You can now apply the upgrade by executing the following command:
    
            kubeadm upgrade apply v1.15.11
    
    _____________________________________________________________________
    ```
    </details>

    ```bash
    kubeadm upgrade apply v1.15.11
    ```

    <details>
    <summary>実行ログ</summary>

    ```bash
    [upgrade/config] Making sure the configuration is correct:
    [upgrade/config] Reading configuration from the cluster...
    [upgrade/config] FYI: You can look at this config file with 'kubectl -n kube-system get cm kubeadm-config -oyaml'
    [preflight] Running pre-flight checks.
    [upgrade] Making sure the cluster is healthy:
    [upgrade/version] You have chosen to change the cluster version to "v1.15.11"
    [upgrade/versions] Cluster version: v1.15.3
    [upgrade/versions] kubeadm version: v1.15.11
    [upgrade/confirm] Are you sure you want to proceed with the upgrade? [y/N]: y
    [upgrade/prepull] Will prepull images for components [kube-apiserver kube-controller-manager kube-scheduler etcd]
    [upgrade/prepull] Prepulling image for component etcd.
    [upgrade/prepull] Prepulling image for component kube-controller-manager.
    [upgrade/prepull] Prepulling image for component kube-scheduler.
    [upgrade/prepull] Prepulling image for component kube-apiserver.
    [apiclient] Found 0 Pods for label selector k8s-app=upgrade-prepull-kube-controller-manager
    [apiclient] Found 0 Pods for label selector k8s-app=upgrade-prepull-kube-apiserver
    [apiclient] Found 0 Pods for label selector k8s-app=upgrade-prepull-kube-scheduler
    [apiclient] Found 0 Pods for label selector k8s-app=upgrade-prepull-etcd
    [apiclient] Found 1 Pods for label selector k8s-app=upgrade-prepull-kube-apiserver
    [apiclient] Found 1 Pods for label selector k8s-app=upgrade-prepull-kube-controller-manager
    [apiclient] Found 1 Pods for label selector k8s-app=upgrade-prepull-kube-scheduler
    [apiclient] Found 1 Pods for label selector k8s-app=upgrade-prepull-etcd
    [upgrade/prepull] Prepulled image for component etcd.
    [upgrade/prepull] Prepulled image for component kube-controller-manager.
    [upgrade/prepull] Prepulled image for component kube-scheduler.
    [upgrade/prepull] Prepulled image for component kube-apiserver.
    [upgrade/prepull] Successfully prepulled the images for all the control plane components
    [upgrade/apply] Upgrading your Static Pod-hosted control plane to version "v1.15.11"...
    Static pod: kube-apiserver-kmalarmpi3b01 hash: 372c3d183b136f8a346942526372841a
    Static pod: kube-controller-manager-kmalarmpi3b01 hash: 6e995c30d7b1af072b8fbe4e6683b6b4
    Static pod: kube-scheduler-kmalarmpi3b01 hash: 7d5d3c0a6786e517a8973fa06754cb75
    [upgrade/etcd] Upgrading to TLS for etcd
    [upgrade/staticpods] Writing new Static Pod manifests to "/etc/kubernetes/tmp/kubeadm-upgraded-manifests749121448"
    [upgrade/staticpods] Preparing for "kube-apiserver" upgrade
    [upgrade/staticpods] Renewing apiserver certificate
    [upgrade/staticpods] Renewing apiserver-kubelet-client certificate
    [upgrade/staticpods] Renewing front-proxy-client certificate
    [upgrade/staticpods] Renewing apiserver-etcd-client certificate
    [upgrade/staticpods] Moved new manifest to "/etc/kubernetes/manifests/kube-apiserver.yaml" and backed up old manifest to "/etc/kubernetes/tmp/kubeadm-backup-manifests-2020-04-26-02-52-05/kube-apiserver.yaml"
    [upgrade/staticpods] Waiting for the kubelet to restart the component
    [upgrade/staticpods] This might take a minute or longer depending on the component/version gap (timeout 5m0s)
    Static pod: kube-apiserver-kmalarmpi3b01 hash: 372c3d183b136f8a346942526372841a
    Static pod: kube-apiserver-kmalarmpi3b01 hash: 5b249192032d3daafac7c3655e11f32b
    [apiclient] Found 1 Pods for label selector component=kube-apiserver
    [upgrade/staticpods] Component "kube-apiserver" upgraded successfully!
    [upgrade/staticpods] Preparing for "kube-controller-manager" upgrade
    [upgrade/staticpods] Renewing controller-manager.conf certificate
    [upgrade/staticpods] Moved new manifest to "/etc/kubernetes/manifests/kube-controller-manager.yaml" and backed up old manifest to "/etc/kubernetes/tmp/kubeadm-backup-manifests-2020-04-26-02-52-05/kube-controller-manager.yaml"
    [upgrade/staticpods] Waiting for the kubelet to restart the component
    [upgrade/staticpods] This might take a minute or longer depending on the component/version gap (timeout 5m0s)
    Static pod: kube-controller-manager-kmalarmpi3b01 hash: 6e995c30d7b1af072b8fbe4e6683b6b4
    Static pod: kube-controller-manager-kmalarmpi3b01 hash: 620b7ec2682bc6e9d29eb65ad0587be5
    [apiclient] Found 1 Pods for label selector component=kube-controller-manager
    [upgrade/staticpods] Component "kube-controller-manager" upgraded successfully!
    [upgrade/staticpods] Preparing for "kube-scheduler" upgrade
    [upgrade/staticpods] Renewing scheduler.conf certificate
    [upgrade/staticpods] Moved new manifest to "/etc/kubernetes/manifests/kube-scheduler.yaml" and backed up old manifest to "/etc/kubernetes/tmp/kubeadm-backup-manifests-2020-04-26-02-52-05/kube-scheduler.yaml"
    [upgrade/staticpods] Waiting for the kubelet to restart the component
    [upgrade/staticpods] This might take a minute or longer depending on the component/version gap (timeout 5m0s)
    Static pod: kube-scheduler-kmalarmpi3b01 hash: 7d5d3c0a6786e517a8973fa06754cb75
    Static pod: kube-scheduler-kmalarmpi3b01 hash: f2ba5c7eb2725c82fea73f81137ff93d
    [apiclient] Found 1 Pods for label selector component=kube-scheduler
    [upgrade/staticpods] Component "kube-scheduler" upgraded successfully!
    [upload-config] Storing the configuration used in ConfigMap "kubeadm-config" in the "kube-system" Namespace
    [kubelet] Creating a ConfigMap "kubelet-config-1.15" in namespace kube-system with the configuration for the kubelets in the cluster
    [kubelet-start] Downloading configuration for the kubelet from the "kubelet-config-1.15" ConfigMap in the kube-system namespace
    [kubelet-start] Writing kubelet configuration to file "/var/lib/kubelet/config.yaml"
    [bootstrap-token] configured RBAC rules to allow Node Bootstrap tokens to post CSRs in order for nodes to get long term certificate credentials
    [bootstrap-token] configured RBAC rules to allow the csrapprover controller automatically approve CSRs from a Node Bootstrap Token
    [bootstrap-token] configured RBAC rules to allow certificate rotation for all node client certificates in the cluster
    [addons] Applied essential addon: CoreDNS
    [addons] Applied essential addon: kube-proxy
    
    [upgrade/successful] SUCCESS! Your cluster was upgraded to "v1.15.11". Enjoy!
    
    [upgrade/kubelet] Now that your control plane is upgraded, please proceed with upgrading your kubelets if you haven't already done so.
    ```
   
   </details>

    成功しました。


1. Manually upgrade your CNI provider plugin.

    手動で使っているCNIをバージョンアップします。

    ```bash
    ARCH="arm64"
    VERSION="v0.8.5"
    curl -LO https://github.com/containernetworking/plugins/releases/download/$VERSION/cni-plugins-linux-$ARCH-$VERSION.tgz
    mkdir -p ./bin
    mv bin/* /opt/cni/bin/
    ```

    > One common issue you may encounter if you are using Flannel network is that after the upgrade, the cluster nodes remain in NotReady state. 

    って事なので、`"cniVersion": "0.8.5",` 行を追加します。

    ```bash
    cat /etc/cni/net.d/10-flannel.conflist
    {
      "name": "cbr0",
      "cniVersion": "0.8.5",
      "plugins": [
        {
          "type": "flannel",
          "delegate": {
            "hairpinMode": true,
            "isDefaultGateway": true
          }
        },
        {
          "type": "portmap",
          "capabilities": {
            "portMappings": true
          }
        }
      ]
    }
    ```

    それでは続けて1.15.11から1.16.9へのアップグレードを同じ手順で実行します。


    <details>
    <summary>実行ログ</summary>

    ```bash
    VERSION="v1.16.9"
    ROLE="server"
    ARCH="arm64"
    curl -LO https://dl.k8s.io/$VERSION/kubernetes-$ROLE-linux-$ARCH.tar.gz
    tar xvfz kubernetes-$ROLE-linux-$ARCH.tar.gz
    install -p -m 755 -t /usr/bin/ kubernetes/$ROLE/bin/kubeadm
    ```

    ```bash
    kubeadm version
    ```

    ```bash
    kubeadm version: &version.Info{Major:"1", Minor:"16", GitVersion:"v1.16.9", GitCommit:"a17149e1a189050796ced469dbd78d380f2ed5ef", GitTreeState:"clean", BuildDate:"2020-04-16T11:42:30Z", GoVersion:"go1.13.9", Compiler:"gc", Platform:"linux/arm64"}
    ```

    ```bash
    kubeadm upgrade plan
    ```

    ```bash
    [upgrade/config] Making sure the configuration is correct:
    [upgrade/config] Reading configuration from the cluster...
    [upgrade/config] FYI: You can look at this config file with 'kubectl -n kube-system get cm kubeadm-config -oyaml'
    [preflight] Running pre-flight checks.
    [upgrade] Making sure the cluster is healthy:
    [upgrade] Fetching available versions to upgrade to
    [upgrade/versions] Cluster version: v1.15.11
    [upgrade/versions] kubeadm version: v1.16.9
    I0426 03:31:00.816440    9647 version.go:251] remote version is much newer: v1.18.2; falling back to: stable-1.16
    [upgrade/versions] Latest stable version: v1.16.9
    [upgrade/versions] Latest version in the v1.15 series: v1.15.11
    
    Components that must be upgraded manually after you have upgraded the control plane with 'kubeadm upgrade apply':
    COMPONENT   CURRENT       AVAILABLE
    Kubelet     3 x v1.15.3   v1.16.9
    
    Upgrade to the latest stable version:
    
    COMPONENT            CURRENT    AVAILABLE
    API Server           v1.15.11   v1.16.9
    Controller Manager   v1.15.11   v1.16.9
    Scheduler            v1.15.11   v1.16.9
    Kube Proxy           v1.15.11   v1.16.9
    CoreDNS              1.3.1      1.6.2
    Etcd                 3.3.10     3.3.15-0
    
    You can now apply the upgrade by executing the following command:
    
            kubeadm upgrade apply v1.16.9
    
    _____________________________________________________________________
    
    ```

    ```bash
    kubeadm upgrade apply v1.16.9
    ```

    ```bash
    [upgrade/config] Making sure the configuration is correct:
    [upgrade/config] Reading configuration from the cluster...
    [upgrade/config] FYI: You can look at this config file with 'kubectl -n kube-system get cm kubeadm-config -oyaml'
    [preflight] Running pre-flight checks.
    [upgrade] Making sure the cluster is healthy:
    [upgrade/version] You have chosen to change the cluster version to "v1.16.9"
    [upgrade/versions] Cluster version: v1.15.11
    [upgrade/versions] kubeadm version: v1.16.9
    [upgrade/confirm] Are you sure you want to proceed with the upgrade? [y/N]: y
    [upgrade/prepull] Will prepull images for components [kube-apiserver kube-controller-manager kube-scheduler etcd]
    [upgrade/prepull] Prepulling image for component etcd.
    [upgrade/prepull] Prepulling image for component kube-controller-manager.
    [upgrade/prepull] Prepulling image for component kube-scheduler.
    [upgrade/prepull] Prepulling image for component kube-apiserver.
    [apiclient] Found 0 Pods for label selector k8s-app=upgrade-prepull-etcd
    [apiclient] Found 0 Pods for label selector k8s-app=upgrade-prepull-kube-scheduler
    [apiclient] Found 0 Pods for label selector k8s-app=upgrade-prepull-kube-controller-manager
    [apiclient] Found 0 Pods for label selector k8s-app=upgrade-prepull-kube-apiserver
    [apiclient] Found 1 Pods for label selector k8s-app=upgrade-prepull-kube-controller-manager
    [apiclient] Found 1 Pods for label selector k8s-app=upgrade-prepull-kube-apiserver
    [apiclient] Found 1 Pods for label selector k8s-app=upgrade-prepull-kube-scheduler
    [apiclient] Found 1 Pods for label selector k8s-app=upgrade-prepull-etcd
    [upgrade/prepull] Prepulled image for component etcd.
    [upgrade/prepull] Prepulled image for component kube-controller-manager.
    [upgrade/prepull] Prepulled image for component kube-scheduler.
    [upgrade/prepull] Prepulled image for component kube-apiserver.
    [upgrade/prepull] Successfully prepulled the images for all the control plane components
    [upgrade/apply] Upgrading your Static Pod-hosted control plane to version "v1.16.9"...
    Static pod: kube-apiserver-kmalarmpi3b01 hash: 5b249192032d3daafac7c3655e11f32b
    Static pod: kube-controller-manager-kmalarmpi3b01 hash: 620b7ec2682bc6e9d29eb65ad0587be5
    Static pod: kube-scheduler-kmalarmpi3b01 hash: f2ba5c7eb2725c82fea73f81137ff93d
    [upgrade/etcd] Upgrading to TLS for etcd
    Static pod: etcd-kmalarmpi3b01 hash: 8520442b2f36fe0dfc9f9804fdc18ebd
    [upgrade/staticpods] Preparing for "etcd" upgrade
    [upgrade/staticpods] Renewing etcd-server certificate
    [upgrade/staticpods] Renewing etcd-peer certificate
    [upgrade/staticpods] Renewing etcd-healthcheck-client certificate
    [upgrade/staticpods] Moved new manifest to "/etc/kubernetes/manifests/etcd.yaml" and backed up old manifest to "/etc/kubernetes/tmp/kubeadm-backup-manifests-2020-04-26-03-36-13/etcd.yaml"
    [upgrade/staticpods] Waiting for the kubelet to restart the component
    [upgrade/staticpods] This might take a minute or longer depending on the component/version gap (timeout 5m0s)
    Static pod: etcd-kmalarmpi3b01 hash: 8520442b2f36fe0dfc9f9804fdc18ebd
    Static pod: etcd-kmalarmpi3b01 hash: 8520442b2f36fe0dfc9f9804fdc18ebd
    Static pod: etcd-kmalarmpi3b01 hash: 00bb1f8db2d41440e3c9fc9a2c4c93a8
    [apiclient] Found 1 Pods for label selector component=etcd
    [upgrade/staticpods] Component "etcd" upgraded successfully!
    [upgrade/etcd] Waiting for etcd to become available
    [upgrade/staticpods] Writing new Static Pod manifests to "/etc/kubernetes/tmp/kubeadm-upgraded-manifests223737586"
    [upgrade/staticpods] Preparing for "kube-apiserver" upgrade
    [upgrade/staticpods] Renewing apiserver certificate
    [upgrade/staticpods] Renewing apiserver-kubelet-client certificate
    [upgrade/staticpods] Renewing front-proxy-client certificate
    [upgrade/staticpods] Renewing apiserver-etcd-client certificate
    [upgrade/staticpods] Moved new manifest to "/etc/kubernetes/manifests/kube-apiserver.yaml" and backed up old manifest to "/etc/kubernetes/tmp/kubeadm-backup-manifests-2020-04-26-03-36-13/kube-apiserver.yaml"
    [upgrade/staticpods] Waiting for the kubelet to restart the component
    [upgrade/staticpods] This might take a minute or longer depending on the component/version gap (timeout 5m0s)
    Static pod: kube-apiserver-kmalarmpi3b01 hash: 5b249192032d3daafac7c3655e11f32b
    Static pod: kube-apiserver-kmalarmpi3b01 hash: 4e214d698d9c9a7cb3f6637b551ee2ad
    [apiclient] Found 1 Pods for label selector component=kube-apiserver
    [apiclient] Error getting Pods with label selector "component=kube-apiserver" [Get https://10.9.8.49:6443/api/v1/namespaces/kube-system/pods?labelSelector=component%3Dkube-apiserver: unexpected EOF]
    [apiclient] Error getting Pods with label selector "component=kube-apiserver" [Get https://10.9.8.49:6443/api/v1/namespaces/kube-system/pods?labelSelector=component%3Dkube-apiserver: dial tcp 10.9.8.49:6443: connect: connection refused]
    [apiclient] Error getting Pods with label selector "component=kube-apiserver" [Get https://10.9.8.49:6443/api/v1/namespaces/kube-system/pods?labelSelector=component%3Dkube-apiserver: dial tcp 10.9.8.49:6443: connect: connection refused]
    [apiclient] Error getting Pods with label selector "component=kube-apiserver" [Get https://10.9.8.49:6443/api/v1/namespaces/kube-system/pods?labelSelector=component%3Dkube-apiserver: dial tcp 10.9.8.49:6443: connect: connection refused]
    [apiclient] Error getting Pods with label selector "component=kube-apiserver" [Get https://10.9.8.49:6443/api/v1/namespaces/kube-system/pods?labelSelector=component%3Dkube-apiserver: dial tcp 10.9.8.49:6443: connect: connection refused]
    [apiclient] Error getting Pods with label selector "component=kube-apiserver" [Get https://10.9.8.49:6443/api/v1/namespaces/kube-system/pods?labelSelector=component%3Dkube-apiserver: dial tcp 10.9.8.49:6443: connect: connection refused]
    [apiclient] Error getting Pods with label selector "component=kube-apiserver" [Get https://10.9.8.49:6443/api/v1/namespaces/kube-system/pods?labelSelector=component%3Dkube-apiserver: dial tcp 10.9.8.49:6443: connect: connection refused]
    [apiclient] Error getting Pods with label selector "component=kube-apiserver" [Get https://10.9.8.49:6443/api/v1/namespaces/kube-system/pods?labelSelector=component%3Dkube-apiserver: dial tcp 10.9.8.49:6443: connect: connection refused]
    [apiclient] Error getting Pods with label selector "component=kube-apiserver" [Get https://10.9.8.49:6443/api/v1/namespaces/kube-system/pods?labelSelector=component%3Dkube-apiserver: dial tcp 10.9.8.49:6443: connect: connection refused]
    [apiclient] Error getting Pods with label selector "component=kube-apiserver" [Get https://10.9.8.49:6443/api/v1/namespaces/kube-system/pods?labelSelector=component%3Dkube-apiserver: dial tcp 10.9.8.49:6443: connect: connection refused]
    [apiclient] Error getting Pods with label selector "component=kube-apiserver" [Get https://10.9.8.49:6443/api/v1/namespaces/kube-system/pods?labelSelector=component%3Dkube-apiserver: dial tcp 10.9.8.49:6443: connect: connection refused]
    [apiclient] Error getting Pods with label selector "component=kube-apiserver" [Get https://10.9.8.49:6443/api/v1/namespaces/kube-system/pods?labelSelector=component%3Dkube-apiserver: dial tcp 10.9.8.49:6443: connect: connection refused]
    [apiclient] Error getting Pods with label selector "component=kube-apiserver" [Get https://10.9.8.49:6443/api/v1/namespaces/kube-system/pods?labelSelector=component%3Dkube-apiserver: dial tcp 10.9.8.49:6443: connect: connection refused]
    [apiclient] Error getting Pods with label selector "component=kube-apiserver" [Get https://10.9.8.49:6443/api/v1/namespaces/kube-system/pods?labelSelector=component%3Dkube-apiserver: dial tcp 10.9.8.49:6443: connect: connection refused]
    [apiclient] Error getting Pods with label selector "component=kube-apiserver" [Get https://10.9.8.49:6443/api/v1/namespaces/kube-system/pods?labelSelector=component%3Dkube-apiserver: dial tcp 10.9.8.49:6443: connect: connection refused]
    [apiclient] Error getting Pods with label selector "component=kube-apiserver" [Get https://10.9.8.49:6443/api/v1/namespaces/kube-system/pods?labelSelector=component%3Dkube-apiserver: dial tcp 10.9.8.49:6443: connect: connection refused]
    [apiclient] Error getting Pods with label selector "component=kube-apiserver" [Get https://10.9.8.49:6443/api/v1/namespaces/kube-system/pods?labelSelector=component%3Dkube-apiserver: net/http: TLS handshake timeout]
    [apiclient] Error getting Pods with label selector "component=kube-apiserver" [Get https://10.9.8.49:6443/api/v1/namespaces/kube-system/pods?labelSelector=component%3Dkube-apiserver: net/http: TLS handshake timeout]
    [apiclient] Error getting Pods with label selector "component=kube-apiserver" [Get https://10.9.8.49:6443/api/v1/namespaces/kube-system/pods?labelSelector=component%3Dkube-apiserver: dial tcp 10.9.8.49:6443: i/o timeout]
    [upgrade/staticpods] Component "kube-apiserver" upgraded successfully!
    [upgrade/staticpods] Preparing for "kube-controller-manager" upgrade
    [upgrade/staticpods] Renewing controller-manager.conf certificate
    [upgrade/staticpods] Moved new manifest to "/etc/kubernetes/manifests/kube-controller-manager.yaml" and backed up old manifest to "/etc/kubernetes/tmp/kubeadm-backup-manifests-2020-04-26-03-36-13/kube-controller-manager.yaml"
    [upgrade/staticpods] Waiting for the kubelet to restart the component
    [upgrade/staticpods] This might take a minute or longer depending on the component/version gap (timeout 5m0s)
    Static pod: kube-controller-manager-kmalarmpi3b01 hash: 620b7ec2682bc6e9d29eb65ad0587be5
    Static pod: kube-controller-manager-kmalarmpi3b01 hash: e4673ecd07784130cf0e7e287a031a88
    [apiclient] Found 1 Pods for label selector component=kube-controller-manager
    [upgrade/staticpods] Component "kube-controller-manager" upgraded successfully!
    [upgrade/staticpods] Preparing for "kube-scheduler" upgrade
    [upgrade/staticpods] Renewing scheduler.conf certificate
    [upgrade/staticpods] Moved new manifest to "/etc/kubernetes/manifests/kube-scheduler.yaml" and backed up old manifest to "/etc/kubernetes/tmp/kubeadm-backup-manifests-2020-04-26-03-36-13/kube-scheduler.yaml"
    [upgrade/staticpods] Waiting for the kubelet to restart the component
    [upgrade/staticpods] This might take a minute or longer depending on the component/version gap (timeout 5m0s)
    Static pod: kube-scheduler-kmalarmpi3b01 hash: f2ba5c7eb2725c82fea73f81137ff93d
    Static pod: kube-scheduler-kmalarmpi3b01 hash: 9c97137c076a125752eccdb1c40a19bc
    [apiclient] Found 1 Pods for label selector component=kube-scheduler
    [upgrade/staticpods] Component "kube-scheduler" upgraded successfully!
    [upload-config] Storing the configuration used in ConfigMap "kubeadm-config" in the "kube-system" Namespace
    [kubelet] Creating a ConfigMap "kubelet-config-1.16" in namespace kube-system with the configuration for the kubelets in the cluster
    [kubelet-start] Downloading configuration for the kubelet from the "kubelet-config-1.16" ConfigMap in the kube-system namespace
    [kubelet-start] Writing kubelet configuration to file "/var/lib/kubelet/config.yaml"
    [bootstrap-token] configured RBAC rules to allow Node Bootstrap tokens to post CSRs in order for nodes to get long term certificate credentials
    [bootstrap-token] configured RBAC rules to allow the csrapprover controller automatically approve CSRs from a Node Bootstrap Token
    [bootstrap-token] configured RBAC rules to allow certificate rotation for all node client certificates in the cluster
    [addons] Applied essential addon: CoreDNS
    [addons] Applied essential addon: kube-proxy
    
    [upgrade/successful] SUCCESS! Your cluster was upgraded to "v1.16.9". Enjoy!
    
    [upgrade/kubelet] Now that your control plane is upgraded, please proceed with upgrading your kubelets if you haven't already done so.
    ```
    </displays>

1. Uncordon the control plane node:

    ```bash
    kubectl uncordon kmalarmpi3b01
    ```

    ```bash
    node/kmalarmpi3b01 uncordoned
    ```

### Upgrade kubelet and kubectl

続いて`kubelet` と `kubectl` をアップグレードします。

1. Upgrade the kubelet and kubectl on all control plane nodes:

    ```bash
    install -p -m 755 -t /usr/bin/ kubernetes/$ROLE/bin/kubelet
    install -p -m 755 -t /usr/bin/ kubernetes/$ROLE/bin/kubectl
    ```

1. Restart the kubelet

    ```bash
    systemctl restart kubelet
    ```
 
## Upgrade worker nodes

### Upgrade kubeadm

1. Upgrade kubeadm on all worker nodes:

    ```bash
    VERSION="v1.16.9"
    ROLE="node"
    ARCH="arm64"
    curl -LO https://dl.k8s.io/$VERSION/kubernetes-$ROLE-linux-$ARCH.tar.gz
    tar xvfz kubernetes-$ROLE-linux-$ARCH.tar.gz
    install -p -m 755 -t /usr/bin/ kubernetes/$ROLE/bin/kubeadm
    ```
 
    ```bash
    kubeadm version
    ```

    ```bash
    kubeadm version: &version.Info{Major:"1", Minor:"16", GitVersion:"v1.16.9", GitCommit:"a17149e1a189050796ced469dbd78d380f2ed5ef", GitTreeState:"clean", BuildDate:"2020-04-16T11:42:30Z", GoVersion:"go1.13.9", Compiler:"gc", Platform:"linux/arm64"}
    ```

### Drain the node

メンテナンスモードにします。

1. Prepare the node for maintenance by marking it unschedulable and evicting the workloads.

    Run:

    ```bash
    kubectl drain knalarmpi3001 --ignore-daemonsets
    ```

    You should see output similar to this:

    ```bash
    node/knalarmpi3001 cordoned
    WARNING: ignoring DaemonSet-managed Pods: kube-system/kube-flannel-ds-arm64-wmb69, kube-system/kube-proxy-87rhs
    evicting pod "prod-postgres-6b777b4b77-s559s"
    evicting pod "coredns-5644d7b6d9-62bmq"
    evicting pod "bc-app-5dbbdb754c-nzv49"
    pod/coredns-5644d7b6d9-62bmq evicted
    pod/bc-app-5dbbdb754c-nzv49 evicted
    pod/prod-postgres-6b777b4b77-s559s evicted
    node/knalarmpi3001 evicted
    ```

### Upgrade the kubelet configuration

1. Call the following command:

    ```bash
    kubeadm upgrade node
    ```

    ```bash
    [upgrade] Reading configuration from the cluster...
    [upgrade] FYI: You can look at this config file with 'kubectl -n kube-system get cm kubeadm-config -oyaml'
    [upgrade] Skipping phase. Not a control plane node[kubelet-start] Downloading configuration for the kubelet from the "kubelet-config-1.16" ConfigMap in the kube-system namespace
    [kubelet-start] Writing kubelet configuration to file "/var/lib/kubelet/config.yaml"
    [upgrade] The configuration for this node was successfully updated!
    [upgrade] Now you should go ahead and upgrade the kubelet package using your package manager.
    ```

### Upgrade kubelet and kubectl

1. Upgrade the kubelet and kubectl on all worker nodes:

    ```bash
    install -p -m 755 -t /usr/bin/ kubernetes/$ROLE/bin/kubelet
    install -p -m 755 -t /usr/bin/ kubernetes/$ROLE/bin/kubectl
    ```

1. Restart the kubelet

    ```bash
    systemctl restart kubelet
    ```

### Uncordon the node

1. Bring the node back online by marking it schedulable:Bring the node back online by marking it schedulable:

    ```bash
    kubectl uncordon knalarmpi3001
    ```

   
### Verify the status of the cluster

```bash
kubectl get nodes
```

でステータスがReadyを確認します。

問題なければ、1.16.x -> 1.17.x, 1.17.x -> 1.18.xと段階的にアップグレードします。

## Summary

- 思った以上にアップグレードは簡単だった
- 1.16.x -> 1.17.x, 1.17.x -> 1.18.xへのアップグレード手順はほぼ変わらない

  - [Upgrading kubeadm cluster from 1.16 to 1.17](https://v1-17.docs.kubernetes.io/docs/tasks/administer-cluster/kubeadm/kubeadm-upgrade/)
  - [Upgrading kubeadm cluster from 1.17 to 1.18](https://v1-18.docs.kubernetes.io/docs/tasks/administer-cluster/kubeadm/kubeadm-upgrade/)

- マスターの更新で失敗しても気にせずリトライしよう
- ansibleでの自動化は次の機会にやってみるか

続く。
