---
title: kubeadm のインストール
content_template: templates/task
weight: 30
---

{{% capture overview %}}

<!--
<img src="https://raw.githubusercontent.com/cncf/artwork/master/kubernetes/certified-kubernetes/versionless/color/certified-kubernetes-color.png" align="right" width="150px">This page shows how to install the `kubeadm` toolbox.
For information how to create a cluster with kubeadm once you have performed this installation process,
see the [Using kubeadm to Create a Cluster](/docs/setup/independent/create-cluster-kubeadm/) page.
-->
<img src="https://raw.githubusercontent.com/cncf/artwork/master/kubernetes/certified-kubernetes/versionless/color/certified-kubernetes-color.png" align="right" width="150px">このページは `kubeadm` ツールボックスのインストール方法を案内します。このインストール手順を終えたあと、kubeadm でクラスタを作成するための情報は [kubeadm を使ってクラスタを作成](/jp/docs/setup/independent/create-cluster-kubeadm/) をご覧ください。

{{% /capture %}}

{{% capture prerequisites %}}

<!--
* One or more machines running one of:
  - Ubuntu 16.04+
  - Debian 9
  - CentOS 7
  - RHEL 7
  - Fedora 25/26 (best-effort)
  - HypriotOS v1.0.1+
  - Container Linux (tested with 1576.4.0)
* 2 GB or more of RAM per machine (any less will leave little room for your apps)
* 2 CPUs or more 
* Full network connectivity between all machines in the cluster (public or private network is fine)
* Unique hostname, MAC address, and product_uuid for every node. See [here](#verify-the-mac-address-and-product-uuid-are-unique-for-every-node) for more details.
* Certain ports are open on your machines. See [here](#check-required-ports) for more details.
* Swap disabled. You **MUST** disable swap in order for the kubelet to work properly. 
-->
* １つまたは複数のマシンでどれか１つを実行：
  - Ubuntu 16.04+
  - Debian 9
  - CentOS 7
  - RHEL 7
  - Fedora 25/26 (ベストエフォート)
  - HypriotOS v1.0.1+
  - Container Linux (1576.4.0 でテスト済み)
* マシンごとに 2 GB 以上のメモリ（これ以上少ないと、アプリケーションによっては動作しない可能性）
* 2 CPU 以上
* クラスタ上のすべてのマシン間で、完全なネットワーク接続性（パブリックよりもプライベート・ネットワークが良い）
* ユニークなホスト名、MAC アドレス、全ノードの product_uuid。詳細は [こちら](#verify-the-mac-address-and-product-uuid-are-unique-for-every-node) をご覧ください
* マシン上で必要なポートが開いている。詳細は [こちら](#check-required-ports) をご覧ください。
* スワップを無効化。kubernetes を適切に動かすためには、 swap の無効化が **必須** です。

{{% /capture %}}

{{% capture steps %}}

<!--
## Verify the MAC address and product_uuid are unique for every node
-->
## MAC アドレスと全ノードの product_uuid がユニークなのを確認 {#verify-the-mac-address-and-product-uuid-are-unique-for-every-node}

<!--
* You can get the MAC address of the network interfaces using the command `ip link` or `ifconfig -a`
* The product_uuid can be checked by using the command `sudo cat /sys/class/dmi/id/product_uuid`
-->
* ネットワーク・インターフェースの MAC アドレスを取得するには、 `ip link` か `ifconfig -a` を使う
* product_uuid を確認するには、コマンド `sudo cat /sys/class/dmi/id/product_uuid` を使う

<!--
It is very likely that hardware devices will have unique addresses, although some virtual machines may have
identical values. Kubernetes uses these values to uniquely identify the nodes in the cluster.
If these values are not unique to each node, the installation process
may [fail](https://github.com/kubernetes/kubeadm/issues/31).
-->
ハードウェア・デバイスはユニークなアドレスを持つと思われますが、いくつかの仮想マシンは全く同じ値を持つ可能性があります。Kubernetes はこの値をクラスタ内のノードをユニークに識別するために使います。各ノードがユニークの値を持たなければ、インストール作業は [失敗](https://github.com/kubernetes/kubeadm/issues/31) するでしょう。

<!--
## Check network adapters
-->
## ネットワーク・アダプタの確認 {#check-network-adapters}

<!--
If you have more than one network adapter, and your Kubernetes components are not reachable on the default
route, we recommend you add IP route(s) so Kubernetes cluster addresses go via the appropriate adapter.
-->
１つ以上のネットワーク・アダプタがある場合、Kubernetes 構成要素がデフォルトの径路では到達できません。そのため、Kubernets クラスタ IP 径路が適切なアダプタを経由するよう IP route の追加を推奨します。

<!--
## Check required ports
-->
## 必要なポートの確認 {#check-required-ports}

<!--
### Master node(s)
--->
### マスタ・ノード {#master-nodes}

<!--
| Protocol | Direction | Port Range | Purpose                 |
|----------|-----------|------------|-------------------------|
| TCP      | Inbound   | 6443*      | Kubernetes API server   |
| TCP      | Inbound   | 2379-2380  | etcd server client API  |
| TCP      | Inbound   | 10250      | Kubelet API             |
| TCP      | Inbound   | 10251      | kube-scheduler          |
| TCP      | Inbound   | 10252      | kube-controller-manager |
| TCP      | Inbound   | 10255      | Read-only Kubelet API   |
-->
| プロトコル | 方向 | ポート範囲 | 用途                 |
|----------|-----------|------------|-------------------------|
| TCP      | Inbound   | 6443*      | Kubernetes API サーバ   |
| TCP      | Inbound   | 2379-2380  | etcd サーバ・クライアント API  |
| TCP      | Inbound   | 10250      | Kubelet API             |
| TCP      | Inbound   | 10251      | kube-scheduler          |
| TCP      | Inbound   | 10252      | kube-controller-manager |
| TCP      | Inbound   | 10255      | Read-only Kubelet API   |


<!--
### Worker node(s)
-->
### ワーカ・ノード {#worker-nodes}

<!--
| Protocol | Direction | Port Range  | Purpose               |
|----------|-----------|-------------|-----------------------|
| TCP      | Inbound   | 10250       | Kubelet API           |
| TCP      | Inbound   | 10255       | Read-only Kubelet API |
| TCP      | Inbound   | 30000-32767 | NodePort Services**   |
-->

| プロトコル | 方向 | ポート範囲  | 用途               |
|----------|-----------|-------------|-----------------------|
| TCP      | Inbound   | 10250       | Kubelet API           |
| TCP      | Inbound   | 10255       | 読み込み専用 Kubelet API |
| TCP      | Inbound   | 30000-32767 | NodePort サービス**   |

<!--
** Default port range for [NodePort Services](/docs/concepts/services-networking/service/).
-->
** [NodePort サービス](/jp/docs/concepts/services-networking/service/) のデフォルトのポート範囲

<!--
Any port numbers marked with * are overridable, so you will need to ensure any
custom ports you provide are also open.
-->
* 印のポート番号は上書き可能です。そのため、任意のポートを指定できますが、確実にオープンにする必要があります。

<!--
Although etcd ports are included in master nodes, you can also host your own
etcd cluster externally or on custom ports.
-->
マスタ・ノードには etcd ポートが含まれますが、任意のポート選択や、自分自身のホストで外部の etcd クラスタも準備できます。

<!--
The pod network plugin you use (see below) may also require certain ports to be
open. Since this differs with each pod network plugin, please see the
documentation for the plugins about what port(s) those need.
-->
使用するポッド・ネットワーク・プラグイン（詳細は以下）によっては、何らかのポートをオープンにする必要が出てくる場合があります。各ポッド・ネットワーク・プラグインによって異なりますので、どのポートが必要になるかについては、プラグインのドキュメントをご覧ください。

<!--
## Installing Docker
-->
## Docker のインストール {#installing-docker}

<!--
On each of your machines, install Docker.
Version 17.03 is recommended, but 1.11, 1.12 and 1.13 are known to work as well.
Versions 17.06+ _might work_, but have not yet been tested and verified by the Kubernetes node team.
Keep track of the latest verified Docker version in the Kubernetes release notes.
-->
各マシンに Docker をインストールします。バージョン 17.03 が推奨ですが、1.11 、1.12、1.13 も同様に動作します。バージョン 17.06 以上も _動作するかもしれません_ が、Kubernetes ノード・チームによるテストは終えていません。Kubernetes リリース・ノートに書かれている確認済みの最新 Docker バージョンをご確認ください。

<!--
Please proceed with executing the following commands based on your OS as root. You may become the root user by executing `sudo -i` after SSH-ing to each host.
-->
使用している OS に対応したコマンドを root として実行します。root ユーザになるには各ホストに SSH した後 `sudo -i` が必要になる場合があります。

<!--
If you already have the required versions of the Docker installed, you can move on to next section.
If not, you can use the following commands to install Docker on your system:
-->
既に必要なバージョンの Docker をインストールしている場合は、次のセクションに移動できます。そうでない場合は、各システムに対応した Docker インストール用のコマンドを実行します。


{{< tabs name="docker_install" >}}
{{% tab name="Ubuntu, Debian or HypriotOS" %}}
<!--
Install Docker from Ubuntu's repositories:
-->
Ubuntu のリポジトリから Docker をインストールします：

```bash
apt-get update
apt-get install -y docker.io
```

<!--
or install Docker CE 17.03 from Docker's repositories for Ubuntu or Debian:
-->
あるいは Ubuntu または Debian の Docker リポジトリから Docker CE 17.03 をインストールします：

```bash
apt-get update
apt-get install -y apt-transport-https ca-certificates curl software-properties-common
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | apt-key add -
add-apt-repository "deb https://download.docker.com/linux/$(. /etc/os-release; echo "$ID") $(lsb_release -cs) stable"
apt-get update && apt-get install -y docker-ce=$(apt-cache madison docker-ce | grep 17.03 | head -1 | awk '{print $3}')
```
{{% /tab %}}
{{% tab name="CentOS, RHEL or Fedora" %}}
<!--
Install Docker using your operating system's bundled package:
-->
オペレーティングシステムに同梱されているパッケージを使って Docker をインストールします：

```bash
yum install -y docker
systemctl enable docker && systemctl start docker
```
{{% /tab %}}
{{% tab name="Container Linux" %}}
<!--
Enable and start Docker:
-->
Docker を有効化して起動します：

```bash
systemctl enable docker && systemctl start docker
```
{{% /tab %}}
{{< /tabs >}}

<!--
Refer to the [official Docker installation guides](https://docs.docker.com/engine/installation/)
for more information.
-->
詳しい情報は [公式 Docker インストール・ガイド](https://docs.docker.com/engine/installation/)をご覧ください。

<!--
## Installing kubeadm, kubelet and kubectl
-->
## kubeadm、kubelet、kubectl のインストール {#installing-kubeadm-kubelet-kubectl}

<!--
You will install these packages on all of your machines:
-->
これらのパッケージをすべてのマシン上にインストールします：

<!--
* `kubeadm`: the command to bootstrap the cluster.

* `kubelet`: the component that runs on all of the machines in your cluster
    and does things like starting pods and containers.

* `kubectl`: the command line util to talk to your cluster.
-->
* `kubeadm`: クラスタを立ち上げるためのコマンド。

* `kubelet`: クラスタ上のマシンすべてで稼働する構成要素（コンポーネント）であり、ポッドとコンテナの起動などを処理。

* `kubectl`: クラスタと通信するためのコマンドライン・ユーティリティ。

<!--
kubeadm **will not** install or manage `kubelet` or `kubectl` for you, so you will 
need to ensure they match the version of the Kubernetes control panel you want 
kubeadm to install for you. If you do not, there is a risk of a version skew occurring that
can lead to unexpected, buggy behaviour. However, _one_ minor version skew between the 
kubelet and the control plane is supported, but the kubelet version may never exceed the API
server version. For example, kubelets running 1.7.0 should be fully compatible with a 1.8.0 API server,
but not vice versa.
-->
kubeadm は `kubelet` や `kubectl` のインストールや管理を **しません**。そのため kubeadm をインストールしたバージョンに対応する Kubernetes コントロール・プレーンが必要です。もしもバージョンが一致しなければ、バージョンのねじれが生じ、それにより予期しないバグ的な挙動を引き起こす可能性があります。しかしながら、kubelet とコントロール・プレーンは１つのマイナーバージョンのねじれをサポートしています。たとえば、kubelets 1.7.0 を実行する環境であれば、1.8.0 API サーバと完全に互換性があるべきです。これは逆の場合も同様です。
<!--
For more information on version skews, please read our 
[version skew policy](/docs/setup/independent/create-cluster-kubeadm/#version-skew-policy).
-->
バージョンのねじれに対する詳しい情報は [バージョンねじれポリシー](/jp/docs/setup/independent/create-cluster-kubeadm/#version-skew-policy) をご覧ください。

{{< tabs name="k8s_install" >}}
{{% tab name="Ubuntu, Debian or HypriotOS" %}}
```bash
apt-get update && apt-get install -y apt-transport-https curl
curl -s https://packages.cloud.google.com/apt/doc/apt-key.gpg | apt-key add -
cat <<EOF >/etc/apt/sources.list.d/kubernetes.list
deb http://apt.kubernetes.io/ kubernetes-xenial main
EOF
apt-get update
apt-get install -y kubelet kubeadm kubectl
```
{{% /tab %}}
{{% tab name="CentOS, RHEL or Fedora" %}}
```bash
cat <<EOF > /etc/yum.repos.d/kubernetes.repo
[kubernetes]
name=Kubernetes
baseurl=https://packages.cloud.google.com/yum/repos/kubernetes-el7-x86_64
enabled=1
gpgcheck=1
repo_gpgcheck=1
gpgkey=https://packages.cloud.google.com/yum/doc/yum-key.gpg https://packages.cloud.google.com/yum/doc/rpm-package-key.gpg
EOF
setenforce 0
yum install -y kubelet kubeadm kubectl
systemctl enable kubelet && systemctl start kubelet
```
<!--
  **Note:**

  - Disabling SELinux by running `setenforce 0` is required to allow containers to access the host filesystem, which is required by pod networks for example.
    You have to do this until SELinux support is improved in the kubelet.
  - Some users on RHEL/CentOS 7 have reported issues with traffic being routed incorrectly due to iptables being bypassed. You should ensure
    `net.bridge.bridge-nf-call-iptables` is set to 1 in your `sysctl` config, e.g.
-->
  **メモ：**
  
  - ホスト・ファイルシステムにコンテナがアクセスするためには、`setenforce 0` を実行して SELinux を無効化する必要があります。たとえば、 pod ネットワークで必要です。この作業は kubelet の SELinux サポートが改善されるまで必要です。
  - RHEL/CentOS 7 の一部ユーザは iptables がバイパスすることによるトラフィック音不適切な径路づけが報告されています。 `sysctl` 設定で `net.bridge.bridge-nf-call-iptables` が 1 になっているかどうかを確認して下さい。例：
  
    ```bash
    cat <<EOF >  /etc/sysctl.d/k8s.conf
    net.bridge.bridge-nf-call-ip6tables = 1
    net.bridge.bridge-nf-call-iptables = 1
    EOF
    sysctl --system
    ```
{{% /tab %}}
{{% tab name="Container Linux" %}}
<!--
Install CNI plugins (required for most pod network):
-->
CNI プラス員をインストールします（ほとんどのポッド・ネットワークに必要です）：

```bash
CNI_VERSION="v0.6.0"
mkdir -p /opt/cni/bin
curl -L "https://github.com/containernetworking/plugins/releases/download/${CNI_VERSION}/cni-plugins-amd64-${CNI_VERSION}.tgz" | tar -C /opt/cni/bin -xz
```

<!--
Install `kubeadm`, `kubelet`, `kubectl` and add a `kubelet` systemd service:
-->
`kubeadm`、`kubelet`、`kubectl`をインストールし、 `kubelet` systemd サービスを追加します。

```bash
RELEASE="$(curl -sSL https://dl.k8s.io/release/stable.txt)"

mkdir -p /opt/bin
cd /opt/bin
curl -L --remote-name-all https://storage.googleapis.com/kubernetes-release/release/${RELEASE}/bin/linux/amd64/{kubeadm,kubelet,kubectl}
chmod +x {kubeadm,kubelet,kubectl}

curl -sSL "https://raw.githubusercontent.com/kubernetes/kubernetes/${RELEASE}/build/debs/kubelet.service" | sed "s:/usr/bin:/opt/bin:g" > /etc/systemd/system/kubelet.service
mkdir -p /etc/systemd/system/kubelet.service.d
curl -sSL "https://raw.githubusercontent.com/kubernetes/kubernetes/${RELEASE}/build/debs/10-kubeadm.conf" | sed "s:/usr/bin:/opt/bin:g" > /etc/systemd/system/kubelet.service.d/10-kubeadm.conf
```

<!--
Enable and start `kubelet`:
-->
`kubelet` を有効化して実行します：

```bash
systemctl enable kubelet && systemctl start kubelet
```
{{% /tab %}}
{{< /tabs >}}

<!--
The kubelet is now restarting every few seconds, as it waits in a crashloop for
kubeadm to tell it what to do.
-->
kubelet は数秒毎に再起動します。これは kubeadm と通信できるようになるまで、クラッシュループのために待機するからです。

<!--
## Configure cgroup driver used by kubelet on Master Node
-->
## マスタ・ノード上で kubelet によって使われる cgroup ドライバの設定

<!--
Make sure that the cgroup driver used by kubelet is the same as the one used by Docker. Verify that your Docker cgroup driver matches the kubelet config:
-->
kubelet によって使われる cgroup ドライバが Docker が使うものと同じかどうか確認します。Docker の cgroup ドライバが kubelet 設定と一致するかどうかを確認するには：

```bash
docker info | grep -i cgroup
cat /etc/systemd/system/kubelet.service.d/10-kubeadm.conf
```

<!--
If the Docker cgroup driver and the kubelet config don't match, change the kubelet config to match the Docker cgroup driver. The
flag you need to change is `--cgroup-driver`. If it's already set, you can update like so:
-->
もしも Docker cgorup ドライバと kubelet 設定が一致しなければ、kubelet の設定を Docker cgroup ドライバと一致するように調整します。変更には `--cgroup-driver`が必要です。既に指定済みであれば、次のように更新する必要があります：

```bash
sed -i "s/cgroup-driver=systemd/cgroup-driver=cgroupfs/g" /etc/systemd/system/kubelet.service.d/10-kubeadm.conf
```

<!--
Otherwise, you will need to open the systemd file and add the flag to an existing environment line.
-->
さらに、systemd ファイルを開き、既存の環境を記述した行にフラグを追加する必要があります。

<!--
Then restart kubelet:
-->
それから kubelet を再起動します：

```bash
systemctl daemon-reload
systemctl restart kubelet
```

<!--
## Troubleshooting
-->
## トラブルシューティング

<!--
If you are running into difficulties with kubeadm, please consult our [troubleshooting docs](/docs/setup/independent/troubleshooting-kubeadm/).
-->
kubeadm の実行がうまくいかない場合は、 [トラブルシューティング・ドキュメント](/jp/docs/setup/independent/troubleshooting-kubeadm/) をご覧ください。

{{% capture whatsnext %}}

<!--
* [Using kubeadm to Create a Cluster](/docs/setup/independent/create-cluster-kubeadm/)
-->
* [kubeadm を使ってクラスタを作成](/jp/docs/setup/independent/create-cluster-kubeadm/)

{{% /capture %}}




