---
title: "おうちKubernetes ~ 構築編 ~"
date: 2020-04-03T15:43:21+09:00
tags: ["Kubernetes", "Memo", "arm", "single board computer"]
draft: false
---

それでは構築していきます。購入した機材はキッティングします。

## ArchLinux

下記参照。Rasberry Pi3へのインストールもダウンロードするリソースの違いがありますがほとんど同じです。

1. [Arch Linux ARM on Rasberry Pi2 インストール]({{< ref "/posts/archlinux/install.md" >}})
1. [Arch Linux ARM on Rasberry Pi2 の初期設定]({{< ref "/posts/archlinux/setting.md" >}})

## Kubernetes

構築はrootにて実施します。

### Server
#### Prerequirement

MACアドレスとproduct_uuidがすべてのノードでユニークであることを確認します。

- You can get the MAC address of the network interfaces using the command **`ip link`** or **`ifconfig -a`**
- The product_uuid can be checked by using the command **`sudo cat /sys/class/dmi/id/product_uuid`**
  `/sys/class/dmi/id/product_uuid` はArchLinux ARMでは無いので **`sudo cat /etc/machine-id`**を確認します。

SeeAlso
: [Verify the MAC address and product_uuid are unique for every node](https://kubernetes.io/docs/setup/production-environment/tools/kubeadm/install-kubeadm/#verify-the-mac-address-and-product-uuid-are-unique-for-every-node)

#### [Installing runtime](https://kubernetes.io/docs/setup/production-environment/tools/kubeadm/install-kubeadm/#installing-runtime)

Dockerのランタイムのコアなcontainerdを使います。

##### Install containerd

```fish
pacman -Sy containerd --noconfirm
```

設定は[こちら](https://kubernetes.io/docs/setup/production-environment/container-runtimes/#containerd)を参照して次のとおり実施します。

##### Prerequisites

```fish
cat > /etc/modules-load.d/containerd.conf <<EOF
overlay
br_netfilter
EOF

modprobe overlay
modprobe br_netfilter

# Setup required sysctl params, these persist across reboots.
cat > /etc/sysctl.d/99-kubernetes-cri.conf <<EOF
net.bridge.bridge-nf-call-iptables  = 1
net.ipv4.ip_forward                 = 1
net.bridge.bridge-nf-call-ip6tables = 1
EOF

sysctl --system
```

```fish
pacman -Sy libseccomp --noconfirm
```

##### Configure containerd

```fish
mkdir -p /etc/containerd
containerd config default > /etc/containerd/config.toml
sed -i 's/systemd_cgroup = false/systemd_cgroup = true/' /etc/containerd/config.toml
```

##### Restart containerd

```fish
systemctl enable containerd
systemctl start containerd
```

SeeAlso
: [Container runtimes](https://kubernetes.io/docs/setup/production-environment/container-runtimes/#systemd)

##### Installing Container Runtime Interface

criが必要なため、リソースリポジトリの[Readme](https://github.com/kubernetes-sigs/cri-tools#install-critest)のとおりインストールします。
containerdを使うため、crictlの設定でruntime-endpointの設定をcontainerdに向けます。

実行する環境のアーキテクチャに応じて、変数を**`ARCH="arm64"`** か **`ARCH="arm"`**を設定します。


```fish
VERSION="v1.15.0"
curl -LO https://github.com/kubernetes-sigs/cri-tools/releases/download/$VERSION/crictl-$VERSION-linux-$ARCH.tar.gz
sudo tar zxvf crictl-$VERSION-linux-$ARCH.tar.gz -C /usr/local/bin
rm -f crictl-$VERSION-linux-$ARCH.tar.gz

cat > /etc/crictl.yaml <<EOF
runtime-endpoint: unix:///run/containerd/containerd.sock
EOF
```

##### Installing Container Network Interface

```fish
VERSION="v0.7.1"
curl -LO https://github.com/containernetworking/plugins/releases/download/$VERSION/cni-plugins-$ARCH-$VERSION.tgz
mkdir -p ./bin
tar -C ./bin -xz -f cni-plugins-$ARCH-$VERSION.tgz
install -m 755 -d /opt/cni
install -m 755 -d /etc/cni/net.d/
mv bin/ /opt/cni
rm -f cni-plugins-$ARCH-$VERSION.tgz
```

#### Installing kubeadm, kubelet and kubectl

ArchLinuxはpacmanでインストールできなかったので、バイナリをダウンロードしてインストールします。下記は手順作成時点で最新の[v1.15.3](https://github.com/kubernetes/kubernetes/blob/master/CHANGELOG-1.15.md#v1153)を用いるので、**`VERSION="v1.15.3"`**をセットします。
また役割はサーバーなので、**`ROLE="server"`**をセットします。

2つの変数をセットした後、次のコマンドを実行してインストールします。

##### resource download and extract files from an archive

```fish
curl -LO https://dl.k8s.io/$VERSION/kubernetes-$ROLE-linux-$ARCH.tar.gz
tar xvfz kubernetes-$ROLE-linux-$ARCH.tar.gz
```

##### kubeadm

```fish
install -m 755 -d /etc/systemd/system/kubelet.service.d/
install -p -m 755 -t /usr/bin/ kubernetes/$ROLE/bin/kubeadm

curl https://raw.githubusercontent.com/kubernetes/kubernetes/$VERSION/build/debs/10-kubeadm.conf > /etc/systemd/system/kubelet.service.d/10-kubeadm.conf


cat > /etc/default/kubelet <<EOF
KUBELET_EXTRA_ARGS=--cgroup-driver=systemd
EOF

cat > /etc/modules-load.d/kubeadm.conf <<EOF
# Load br_netfilter module at boot
br_netfilter
EOF

cat > /usr/lib/sysctl.d/50-kubeadm.conf <<EOF
# The file is provided as part of the kubeadm package
net.ipv4.ip_forward = 1
EOF
```

##### kubelet

```fish
pacman -Sy ebtables ethtool socat ipvsadm --noconfirm
install -p -m 755 -t /usr/bin/ kubernetes/$ROLE/bin/kubelet
install -m 755 -d /etc/kubernetes/manifests/

curl https://raw.githubusercontent.com/kubernetes/kubernetes/$VERSION/build/debs/kubelet.service > /etc/systemd/system/kubelet.service

cat > /etc/systemd/system/kubelet.service.d/0-containerd.conf <<EOF
[Service]
Environment="KUBELET_EXTRA_ARGS=--container-runtime=remote --runtime-request-timeout=15m --container-runtime-endpoint=unix:///run/containerd/containerd.sock"
EOF

systemctl daemon-reload
systemctl enable --now kubelet
```

##### kubectl

```fish
install -p -m 755 -t /usr/bin/ kubernetes/$ROLE/bin/kubectl
```

##### clean up

```fish
rm -f kubernetes-$ROLE-linux-$ARCH.tar.gz
rm -fR kubernetes/
```

#### Cleate Cruster

後述するpod network add-onは、armおよびarm64で動くflannelを使います。flannelの場合、kubeadm init にオプション**`--pod-network-cidr=10.244.0.0/16`**を渡します（MUST）。

```fish
kubeadm init --cri-socket=/run/containerd/containerd.sock --pod-network-cidr=10.244.0.0/16

mkdir -p $HOME/.kube
sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config
```

SeeAlso
: [Creating a single control-plane cluster with kubeadm](https://kubernetes.io/docs/setup/production-environment/tools/kubeadm/create-cluster-kubeadm/)
: [Creating a single control-plane cluster with kubeadm#pod-network](https://kubernetes.io/docs/setup/production-environment/tools/kubeadm/create-cluster-kubeadm/#pod-network)

#### Installing a pod network add-on

[ここ](https://itnext.io/benchmark-results-of-kubernetes-network-plugins-cni-over-10gbit-s-network-updated-april-2019-4a9886efe9c4)によると、低リソースノードがある場合はflannelを使うとあるので、これを利用します。

```fish
kubectl apply -f https://raw.githubusercontent.com/coreos/flannel/62e44c867a2846fefb68bd5f178daf4da3095ccb/Documentation/kube-flannel.yml
```

#### Control plane node isolation

```fish
kubectl taint nodes --all node-role.kubernetes.io/master-
```

### Node

#### Prerequirement

[Server]({{< ref "#prerequirement" >}})と同様に前提条件を確認します。

#### [Installing runtime](https://kubernetes.io/docs/setup/production-environment/tools/kubeadm/install-kubeadm/#installing-runtime)

[Server]({{< ref "#installing-runtimehttpskubernetesiodocssetupproduction-environmenttoolskubeadminstall-kubeadminstalling-runtime" >}})と同様に実行します。

#### Installing kubeadm, kubelet and kubectl

インストールに利用するリソースが違うので、**`ROLE="node"`**をセットしてから[Server]({{< ref "#installing-kubeadm-kubelet-and-kubectl" >}})と同様に実行します。

#### Join your nodes

`kubeadm init`が成功した場合に出力される情報を用いてnodeをクラスターに参加させます。

```fish
kubeadm join 10.9.8.45:6443 --token zww3no.2tz9vb782dibmrfo \
--discovery-token-ca-cert-hash sha256:22cc9bc5daa488ba7bd8f2233ab9344031534c14ac499f8929f1c5b0520553fc
````

デフォルトでは24時間でトークンの有効期限が切れます。有効期限が切れた場合は以下参考にしてトークンを作成し、それを用いてクラスターに参加させます。

```fish
export KUBECONFIG=/etc/kubernetes/admin.conf
kubeadm token create
openssl x509 -pubkey -in /etc/kubernetes/pki/ca.crt | openssl rsa -pubin -outform der 2>/dev/null | \
   openssl dgst -sha256 -hex | sed 's/^.* //'

kubeadm join 10.9.8.45:6443 --token d9rpan.9pxae3hotvpywoza \
    --cri-socket /run/containerd/containerd.sock \
    --discovery-token-ca-cert-hash sha256:22cc9bc5daa488ba7bd8f2233ab9344031534c14ac499f8929f1c5b0520553fc
```

SeeAlso
: [Creating a single control-plane cluster with kubeadm](https://kubernetes.io/docs/setup/production-environment/tools/kubeadm/create-cluster-kubeadm/#join-nodes)

```fish
kubectl label node knalarmpi2001 node-role.kubernetes.io/worker=worker
```

## MetalLBの導入

Service の type LoadBalancer を使えるようにするため、bare metal Kubernetes クラスタ用のロードバランサー実装を導入します。

基本的にはオフィシャルサイトのインストール手順のとおりです。


### 準備

kube-proxyを IPVS モードで実行している場合はstrict ARP を有効にする必要があるようですが、自環境はそのモードで実行していなかったので設定は行っていません。

### マニフェストによるインストール

何も考えず[オフィシャルページ](https://metallb.universe.tf/installation/#installation-by-manifest)にあるコマンドを実行します。

```fish
kubectl apply -f https://raw.githubusercontent.com/metallb/metallb/v0.9.3/manifests/namespace.yaml
kubectl apply -f https://raw.githubusercontent.com/metallb/metallb/v0.9.3/manifests/metallb.yaml
# On first install only
kubectl create secret generic -n metallb-system memberlist --from-literal=secretkey="$(openssl rand -base64 128)"
```

### コンフィグレーション

自環境でBGP使えるか調べたのですがダメそうなので、L2の設定を行います。

```fish
apiVersion: v1
kind: ConfigMap
metadata:
  namespace: metallb-system
  name: config
data:
  config: |
    address-pools:
    - name: default
      protocol: layer2
      addresses:
      - 10.9.8.240-10.9.8.250
```

これで `kubectl port-forward` を都度行わなくて済みます。 [^1]

[^1]: とはいっても、EXTERNAL-IP をデプロイしたタイミングに `kubectl get svc` で調べる必要があります。

SeeAlso:
- [Kubernetes 1.16をオンプレ環境にインストールしてみた - MetalLBの導入](https://qiita.com/rltnm7/items/3e3948893983737f736b#metallb%E3%81%AE%E5%B0%8E%E5%85%A5)
- [オンプレK8sで使えるGoogle製External Load Balancer: MetalLB](https://qiita.com/tmatsu/items/f45f0ca07b4f8489df85)
- [KubernetesロードバランサーのMetalLBを導入した話(Necoプロジェクト体験入部)](https://blog.cybozu.io/entry/2019/03/25/093000)
- [Using Kubernetes ExternalDNS & MetalLB with a Home/Bare Metal K8S: Part 1](https://blog.cowger.us/2018/07/25/using-kubernetes-externaldns-with-a-home-bare-metal-k8s.html)
- [METALLB](https://metallb.universe.tf/)

[おうちKubernetes ~ 手作業によるバージョンアップグレード編 ~]({{< ref "/posts/myhomekubernetes/upgrade.md" >}})へ続く。
