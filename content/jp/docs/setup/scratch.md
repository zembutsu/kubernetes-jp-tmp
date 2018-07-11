---
reviewers:
- erictune
- lavalamp
- thockin
title: カスタム・クラスタをゼロ（スクラッチ）から構築
---

<!--
This guide is for people who want to craft a custom Kubernetes cluster.  If you
can find an existing Getting Started Guide that meets your needs on [this
list](/docs/setup/), then we recommend using it, as you will be able to benefit
from the experience of others.  However, if you have specific IaaS, networking,
configuration management, or operating system requirements not met by any of
those guides, then this guide will provide an outline of the steps you need to
take.  Note that it requires considerably more effort than using one of the
pre-defined guides.
-->
このガイドが対象としているのは、カスタム Kubernetes クラスタを高度な技術でうまく作りたい方々です。[こちらの一覧](/jp/docs/setup/)から、既存の導入ガイドから必要に応じたガイドを見つけられれば、そちらをお読みいただくのを推奨します。他のガイドを試されるよりも役立つでしょう。しかしながら、特定の IaaS、ネットワーク機能、設定管理、オペレーティングシステムの必要条件に一致するものが先程のガイドになければ、こちらのガイドで皆さんが必要に応じて取るべき手順概要を提供します。予め準備されているガイドをご利用になるよりも、かなりの取り組みが必要になりますのでご注意ください。

<!--
This guide is also useful for those wanting to understand at a high level some of the
steps that existing cluster setup scripts are making.
-->
またこのガイドは、既存のクラスタをセットアップするスクリプトが何をしているか、高いレベルで手順を理解をしたい方にも役立つでしょう。

{{< toc >}}

<!--
## Designing and Preparing
-->
## 設計と準備

<!--
### Learning
-->
### 学習

<!--
  1. You should be familiar with using Kubernetes already.  We suggest you set
    up a temporary cluster by following one of the other Getting Started Guides.
    This will help you become familiar with the CLI ([kubectl](/docs/user-guide/kubectl/)) and concepts ([pods](/docs/user-guide/pods/), [services](/docs/concepts/services-networking/service/), etc.) first.
  1. You should have `kubectl` installed on your desktop.  This will happen as a side
    effect of completing one of the other Getting Started Guides.  If not, follow the instructions
    [here](/docs/tasks/kubectl/install/).
-->
  1. 既に Kubernetes の使用に慣れているべきでしょう。他の導入ガイドのどれか１つに従い、一時的なクラスタのセットアップを推奨します。これにより、まず CLI ([kubectl](/jp/docs/user-guide/kubectl/)) と概念（[ポッド](/jp/docs/user-guide/pods/)、 [サービス](/jp/docs/concepts/services-networking/service/)、等 ）に慣れるのが役立つでしょう。
  1. 皆さんの PC 上に `kubectl` をインストールすべきでしょう。この結果、他の導入ガイドを終わらせようとしても、思わぬ副作用を引き起こすかもしれません。問題なければ [こちら](/jp/docs/tasks/kubectl/install/) の手順に従ってください。

<!--
### Cloud Provider
-->
### クラウド・プロバイダ（Cloud Provider）{#cloud-provider}

<!--
Kubernetes has the concept of a Cloud Provider, which is a module which provides
an interface for managing TCP Load Balancers, Nodes (Instances) and Networking Routes.
The interface is defined in `pkg/cloudprovider/cloud.go`.  It is possible to
create a custom cluster without implementing a cloud provider (for example if using
bare-metal), and not all parts of the interface need to be implemented, depending
on how flags are set on various components.
-->
Kubernetes にはクラウド・プロバイダ（Cloud Provider）の概念を持ちます。クラウド・プロバイダとはインターフェースを提供するモジュールであり、TCP 負荷分散、ノード（インスタンス）、ネットワーク上の径路を管理します。インターフェースは `pkg/cloudprovider/cloud.go` で定義されています。これはクラウド・プロバイダの実装を変更せずに、カスタム・クラスタの構築を可能にします（たとえばベアメタルを使いたい場合）。また、インターフェースが必要な全ての部品を実装する必要はありません。依存するのは、様々な構成要素（コンポーネント）でどのようにフラグをセットするかに依存します。

<!--
### Nodes
-->
### ノード

<!--
- You can use virtual or physical machines.
- While you can build a cluster with 1 machine, in order to run all the examples and tests you
  need at least 4 nodes.
- Many Getting-started-guides make a distinction between the master node and regular nodes.  This
  is not strictly necessary.
- Nodes will need to run some version of Linux with the x86_64 architecture.  It may be possible
  to run on other OSes and Architectures, but this guide does not try to assist with that.
- Apiserver and etcd together are fine on a machine with 1 core and 1GB RAM for clusters with 10s of nodes.
  Larger or more active clusters may benefit from more cores.
- Other nodes can have any reasonable amount of memory and any number of cores.  They need not
  have identical configurations.
-->
- 物理または仮想マシンを使用できます。
- クラスタは１台のマシンでも構成可能ですが、全ての例やテストを行うには、少なくとも４つのノードが必要です。
- 多くの導入ガイドはマスタ・ノードと通常のノードを区別しています。厳密には不要です。
- ノードには x86_64 アーキテクチャに対応した何らかの Linux バージョンが必要です。他の OS やアーキテクチャも実行可能かもしれませんが、このガイドでは扱いません。
- apiserver と etcd は１台のマシンで一緒に動かして構いません。10台のノードで構成するクラスタごとに、マシンは１コア、1GB のメモリです。より大きなクラスタでは、さらに多くのコアを用意した方が良いでしょう。
- 他のノードはより多くのメモリと多くのコア数が適切です。設定は全てを同じにする必要はありません。

<!--
### Network
-->
### ネットワーク

<!--
#### Network Connectivity
-->
#### ネットワーク接続性

<!--
Kubernetes has a distinctive [networking model](/docs/concepts/cluster-administration/networking/).
-->
Kubernetes は独自の[ネットワーク構造（モデル）](/jp/docs/concepts/cluster-administration/networking/)です。

<!--
Kubernetes allocates an IP address to each pod.  When creating a cluster, you
need to allocate a block of IPs for Kubernetes to use as Pod IPs.  The simplest
approach is to allocate a different block of IPs to each node in the cluster as
the node is added.  A process in one pod should be able to communicate with
another pod using the IP of the second pod.  This connectivity can be
accomplished in two ways:
-->
Kubernetes はポッドごとに IP アドレスを割り当てます。クラスタの作成時、ポッド IP が利用するために、Kubernetes に対して IP アドレスのブロックを割り当てる必要があります。クラスタに追加するノードごとに、異なる IP のブロックを割り当てるのが一番簡単な取り組みです。

<!--
- **Using an overlay network**
  - An overlay network obscures the underlying network architecture from the
    pod network through traffic encapsulation (for example vxlan).
  - Encapsulation reduces performance, though exactly how much depends on your solution.
- **Without an overlay network**
  - Configure the underlying network fabric (switches, routers, etc.) to be aware of pod IP addresses.
  - This does not require the encapsulation provided by an overlay, and so can achieve
    better performance.
-->
- **オーバレイ・ネットワークを使用する**
  - オーバレイ・ネットワークは基礎となるネットワーク構造（アーキテクチャ）を覆い隠します。これは、ポッド・ネットワークを通した通信（トラフィック）をカプセル化（例えば vxlan）するためです。
  - どのような解決策（ソリューション）を用いても、カプセル化は性能を低下します。
- **オーバレイ・ネットワークを使用しない**
  - ポッドの IP アドレスを認識できるよう、基礎となるネットワーク構成要素（fabric）（スイッチ、ルータ、等）を設定します。
  - オーバレイによるカプセル化が不要なため、より良い性能を得られます。

<!--
Which method you choose depends on your environment and requirements.  There are various ways
to implement one of the above options:
-->
どの手法を選択するかは、環境と必要条件に依存します。上述のオプションを実装するには、様々な方法があります。

<!--
- **Use a network plugin which is called by Kubernetes**
  - Kubernetes supports the [CNI](https://github.com/containernetworking/cni) network plugin interface.
  - There are a number of solutions which provide plugins for Kubernetes (listed alphabetically):
    - [Calico](http://docs.projectcalico.org/)
    - [Flannel](https://github.com/coreos/flannel)
    - [Open vSwitch (OVS)](http://openvswitch.org/)
    - [Romana](http://romana.io/)
    - [Weave](http://weave.works/)
    - [More found here](/docs/admin/networking#how-to-achieve-this/)
  - You can also write your own.
- **Compile support directly into Kubernetes**
  - This can be done by implementing the "Routes" interface of a Cloud Provider module.
  - The Google Compute Engine ([GCE](/docs/getting-started-guides/gce/)) and [AWS](/docs/getting-started-guides/aws/) guides use this approach.
- **Configure the network external to Kubernetes**
  - This can be done by manually running commands, or through a set of externally maintained scripts.
  - You have to implement this yourself, but it can give you an extra degree of flexibility.
-->
- **Kubernetes が呼び出すネットワーク・プラグインを使う**
  - Kubernetes は [CNI](https://github.com/containernetworking/cni) ネットワーク・プラグイン・インターフェースをサポートします。
  - Kubernetes 用に提供されているプラグインによる、様々な解決策（一覧はアルファベット順）：
    - [Calico](http://docs.projectcalico.org/)
    - [Flannel](https://github.com/coreos/flannel)
    - [Open vSwitch (OVS)](http://openvswitch.org/)
    - [Romana](http://romana.io/)
    - [Weave](http://weave.works/)
    - [こちらに更にあります](/jp/docs/admin/networking#how-to-achieve-this/)
  - あるいは、自分自身でも書けます。
- **Kubernetes が直接コンパイルをサポート**
  - クラウド・プロバイダ・モジュールの「Routes」（径路付け）インターフェースとして実装可能できます。。
  - Google Compute Engine  ([GCE](/jp/docs/getting-started-guides/gce/)) と [AWS](/jp/docs/getting-started-guides/aws/) がこの方法です。
- **Kubernetes のネットワーク外で設定**
  - 手動でのコマンド実行や、外部で保守されているスクリプト群を通して行います。
  - 自分自身で実装する必要がありますが、高い自由度を得られます。

<!--
You will need to select an address range for the Pod IPs.
-->
Pod IP のアドレス範囲を自分で選択する必要があります。

<!--
- Various approaches:
  - GCE: each project has its own `10.0.0.0/8`.  Carve off a `/16` for each
    Kubernetes cluster from that space, which leaves room for several clusters.
    Each node gets a further subdivision of this space.
  - AWS: use one VPC for whole organization, carve off a chunk for each
    cluster, or use different VPC for different clusters.
- Allocate one CIDR subnet for each node's PodIPs, or a single large CIDR
  from which smaller CIDRs are automatically allocated to each node.
  - You need max-pods-per-node * max-number-of-nodes IPs in total. A `/24` per
    node supports 254 pods per machine and is a common choice.  If IPs are
    scarce, a `/26` (62 pods per machine) or even a `/27` (30 pods) may be sufficient.
  - For example, use `10.10.0.0/16` as the range for the cluster, with up to 256 nodes
    using `10.10.0.0/24` through `10.10.255.0/24`, respectively.
  - Need to make these routable or connect with overlay.
-->
- 様々な取り組み：
  - GCE: 各プロジェクトが自身で `10.0.0.0/8` を持ちます。各 Kubernetes クラスタの領域から `/16` が切り取られますので、複数のクラスタのために余地を残します。
  - AWS: 組織ごとに１つの VPC を使うため、クラスタが大きな塊となるか、異なるクラスタごとに異なる VPC を用います。
- 各ノードの Pod に１つの CIDR サブネットを割り当てるか、小さな CIDR よりも大きな CIDR が自動的に各ノードに割り当てられます。
  - ノードごとの最大ポッド数（max-pods-per-node） × 最大ノード数（max-number-of-nodes）の IP アドレスが合計で必要です。マシンで 254 のポッドをサポートできるよう、ノードごとに `/24`が一般的な選択です。IP が不足していれば、 `/26` （マシンごとに 62 ポッド）か、あるいは `/27` （30ポッド）でも十分でしょう。
  - たとえば、クラスタの範囲として `10.10.0.0/16` を用いると、最大で256 ノードが `10.10.0.0/24` から `10.10.255.0/24` の範囲で個々に使えます。
  - これらには到達可能（routable）かオーバレイで接続できるようにする必要があります。

<!--
Kubernetes also allocates an IP to each [service](/docs/concepts/services-networking/service/).  However,
service IPs do not necessarily need to be routable.  The kube-proxy takes care
of translating Service IPs to Pod IPs before traffic leaves the node.  You do
need to allocate a block of IPs for services.  Call this
`SERVICE_CLUSTER_IP_RANGE`.  For example, you could set
`SERVICE_CLUSTER_IP_RANGE="10.0.0.0/16"`, allowing 65534 distinct services to
be active at once.  Note that you can grow the end of this range, but you
cannot move it without disrupting the services and pods that already use it.
-->
また、Kubernetes は IP を各 [サービス](/jp/docs/concepts/services-networking/service/) にも割り当てます。しかしながら、サービス IP に到達可能（routable）である必要はありません。ノードから通信が離れる前に、ポッド IP に対し、kube-proxy はサービス IPの変換（translating Service IP）を管理します。サービスに対して IP のブロックを割り当てる必要があります。これを `SERVICE_CLUSTER_IP_RANGE` （サービス・クラスタ IP 範囲）と呼びます。たとえば `SERVICE_CLUSTER_IP_RANGE="10.0.0.0/16"` を指定したら、一度に 65534 の異なるサービスを動かせます。範囲の最後に到達してしまうと、既に利用しているサービスやノードを壊さず移動できませんので、ご注意ください。

<!--
Also, you need to pick a static IP for master node.
-->
また、マスタ・ノードに対しては固定 IP の選定が必要です。

<!--
- Call this `MASTER_IP`.
- Open any firewalls to allow access to the apiserver ports 80 and/or 443.
- Enable ipv4 forwarding sysctl, `net.ipv4.ip_forward = 1`
-->
- これを `MASTER_IP` （マスタ IP）と呼びます。
- apiserver のポート 80 と 443 の両方・あるいは片方に対してファイアウォールからの接続を開く（オープンにする）必要があります。
- ipv4 転送（フォワーディング）を sysctl で有効にするには、  `net.ipv4.ip_forward = 1` を実行します。

<!--
#### Network Policy
-->
#### ネットワーク・ポリシー

<!--
Kubernetes enables the definition of fine-grained network policy between Pods using the [NetworkPolicy](/docs/concepts/services-networking/network-policies/) resource.
-->
Kubernets は [ネットワーク・ポリシー](/jp/docs/concepts/services-networking/network-policies/) リソースを使って、きめ細かなポッド間におけるネットワーク・ポリシー定義を可能にします。

<!--
Not all networking providers support the Kubernetes NetworkPolicy API, see [Using Network Policy](/docs/tasks/configure-pod-container/declare-network-policy/) for more information.
-->
全てのネットワーク・プロバイダが Kubernetes ネットワーク・ポリシー API をサポートするわけではありません。詳しい情報は[ネットワーク・ポリシーの利用](/docs/tasks/configure-pod-container/declare-network-policy/)をご覧ください。

<!--
### Cluster Naming
-->
### クラスタの命名

<!--
You should pick a name for your cluster.  Pick a short name for each cluster
which is unique from future cluster names. This will be used in several ways:
-->
クラスタには名前を付けるべきでしょう。今後重複しないユニークで短い名前を各クラスタごとに選びます。いくつかの手段で使われます。

<!--
  - by kubectl to distinguish between various clusters you have access to.  You will probably want a
    second one sometime later, such as for testing new Kubernetes releases, running in a different
region of the world, etc.
  - Kubernetes clusters can create cloud provider resources (for example, AWS ELBs) and different clusters
    need to distinguish which resources each created.  Call this `CLUSTER_NAME`.
-->
  - kubectl でアクセスが必要な様々なクラスタ間を識別します。後々になり、新しい Kubernetes リリースのテストなど、２つめのクラスタを準備したいとき、世界の別のリージョンで実行したいとき等です。
  - Kubernetes クラスタはクラウド・プロバイダのリソースを作成できますので（たとえば、AWS の ELB）、作成するリソースを区別するためにクラスタの識別が必要です。これが `CLUSTER_NAME` （クラスタ名）です。

<!--
### Software Binaries
-->
### ソフトウェアのバイナリ {#software-binaries}

<!--
You will need binaries for:
-->
それぞれのバイナリが必要です：

<!--
  - etcd
  - A container runner, one of:
    - docker
    - rkt
  - Kubernetes
    - kubelet
    - kube-proxy
    - kube-apiserver
    - kube-controller-manager
    - kube-scheduler
-->

  - etcd
  - コンテナを動かすもの、どちらか１つ:
    - docker
    - rkt
  - Kubernetes
    - kubelet
    - kube-proxy
    - kube-apiserver
    - kube-controller-manager
    - kube-scheduler

<!--
#### Downloading and Extracting Kubernetes Binaries
-->
#### Kubernetes バイナリのダウンロードと展開

<!--
A Kubernetes binary release includes all the Kubernetes binaries as well as the supported release of etcd.
You can use a Kubernetes binary release (recommended) or build your Kubernetes binaries following the instructions in the
[Developer Documentation](https://git.k8s.io/community/contributors/devel/).  Only using a binary release is covered in this guide.
-->
Kubernetes バイナリ・リリースに含まれてるのは、Kubernetes の全てのバイナリだけでなく、サポート対象の etcd もです。Kubernetes バイナリ・リリースを使う（推奨）か、[開発ドキュメント](https://git.k8s.io/community/contributors/devel/) にある手順に従い Kubernetes バイナリの構築も可能です。このガイドではバイナリ・リリースを使う方法のみ扱います。

<!--
Download the [latest binary release](https://github.com/kubernetes/kubernetes/releases/latest) and unzip it.
Server binary tarballs are no longer included in the Kubernetes final tarball, so you will need to locate and run
`./kubernetes/cluster/get-kube-binaries.sh` to download the client and server binaries.
Then locate `./kubernetes/server/kubernetes-server-linux-amd64.tar.gz` and unzip *that*.
Then, within the second set of unzipped files, locate `./kubernetes/server/bin`, which contains
all the necessary binaries.
-->
[最新のバイナリ・リリース](https://github.com/kubernetes/kubernetes/releases/latest)をダウンロードし、展開（unzip）します。Kubernetes 最新の tar ボールには、サーバのバイナリは含まれていません。そのため、クライアントとサーバのバイナリをダウンロードするには、`./kubernetes/cluster/get-kube-binaries.sh` を探して実行する必要があります。それから `./kubernetes/server/kubernetes-server-linux-amd64.tar.gz` を探し、 **それを** 展開（unzip）します。それから、２回めに展開したファイル群の場所へ移動し、全ての必要なバイナリが含まれている `./kubernetes/server/bin` を探します。

<!--
#### Selecting Images
-->
#### イメージの選択 {#selecting-images}

<!--
You will run docker, kubelet, and kube-proxy outside of a container, the same way you would run any system daemon, so
you just need the bare binaries.  For etcd, kube-apiserver, kube-controller-manager, and kube-scheduler,
we recommend that you run these as containers, so you need an image to be built.
-->
コンテナのほかにも docker 、kubelet、kube-proxy を実行します。何らかのシステム・デーモンも同様に実行する必要があるため、それぞれのバイナリが必要です。etcd、kube-apiserver、kube-controller-maager、kube-scheduler については、コンテナとしての実行を推奨しますので、イメージを構築する必要があります。

<!--
You have several choices for Kubernetes images:
-->
Kubernetes イメージには複数の選択肢があります：

<!--
- Use images hosted on Google Container Registry (GCR):
  - For example `k8s.gcr.io/hyperkube:$TAG`, where `TAG` is the latest
    release tag, which can be found on the [latest releases page](https://github.com/kubernetes/kubernetes/releases/latest).
  - Ensure $TAG is the same tag as the release tag you are using for kubelet and kube-proxy.
  - The [hyperkube](https://releases.k8s.io/{{< param "githubbranch" >}}/cmd/hyperkube) binary is an all in one binary
    - `hyperkube kubelet ...` runs the kubelet, `hyperkube apiserver ...` runs an apiserver, etc.
- Build your own images.
  - Useful if you are using a private registry.
  - The release contains files such as `./kubernetes/server/bin/kube-apiserver.tar` which
    can be converted into docker images using a command like
    `docker load -i kube-apiserver.tar`
  - You can verify if the image is loaded successfully with the right repository and tag using
    command like `docker images`
-->
- Google Container Registry (GCR)にあるイメージを使う：
  - たとえば、 `k8s.gcr.io/hyperkube:$TAG` の場合、 `TAG` が最新（latest）リリースのタグのものです。これは [最新恩リリースページ](https://github.com/kubernetes/kubernetes/releases/latest)にあります。
  - kubelet や kube-proxy で使おうとしている release タグも $TAG と同じかどうか確認します。
  - [hyperkube](https://releases.k8s.io/{{< param "githubbranch" >}}/cmd/hyperkube) バイナリは、すべてが１つのバイナリに入っています。
    - `hyperkube kubelet ...` は kubelet を実行し、 `hyperkube apiserver ...` は apiserver を実行します。
- 自分でイメージを構築する：
  - プライベート・レジストリを使う場合に便利です。
  - リリースには `./kubernetes/server/bin/kube-apiserver.tar` のようなファイルを含みます。こちらを docker イメージに変換するには `docker load -i kube-apiserver.tar` コマンドで変換できます
  - イメージの読み込みが成功したかどうかの確認は、`docker images` のようなコマンドを使い、リポジトリとタグが正しいかどうかご確認ください。

<!--
For etcd, you can:
-->
etcd の場合は、次のようにできます：

<!--
- Use images hosted on Google Container Registry (GCR), such as `k8s.gcr.io/etcd:2.2.1`
- Use images hosted on [Docker Hub](https://hub.docker.com/search/?q=etcd) or [Quay.io](https://quay.io/repository/coreos/etcd), such as `quay.io/coreos/etcd:v2.2.1`
- Use etcd binary included in your OS distro.
- Build your own image
  - You can do: `cd kubernetes/cluster/images/etcd; make`
-->
- `k8s.gcr.io/etcd:2.2.1` のような Google Contaienr Registry (GCR) にあるイメージを使う。
- [Docker Hub](https://hub.docker.com/search/?q=etcd) や [Quay.io](https://quay.io/repository/coreos/etcd) にある `quay.io/coreos/etcd:v2.2.1` のようなイメージを使う。
- OS ディストリビューションに含まれている etcd 倍内を使う。
- 自分でイメージを構築する。
  - 自分でできます: `cd kubernetes/cluster/images/etcd; make`


<!--
We recommend that you use the etcd version which is provided in the Kubernetes binary distribution.   The Kubernetes binaries in the release
were tested extensively with this version of etcd and not with any other version.
The recommended version number can also be found as the value of `TAG` in `kubernetes/cluster/images/etcd/Makefile`.
-->
私たちが推奨する etcd のバージョンは、Kubernetes バイナリ配布物で提供されているものです。Kubernetes バイナリのリリースには、広範囲にテストされた etcd のバージョンが含まれていますが、他のバージョンではありません。また、推奨されているバージョン番号は `kubernetes/cluster/images/etcd/Makefile` にある `TAG` の値からもわかります。

<!--
The remainder of the document assumes that the image identifiers have been chosen and stored in corresponding env vars.  Examples (replace with latest tags and appropriate registry):
-->
以降のドキュメントでは、イメージ識別子を選択し、適切な環境変数（env var）に保存しているものとみなします。例（latest タグと適切なレジストリに書き換えてください）：

  - `HYPERKUBE_IMAGE=k8s.gcr.io/hyperkube:$TAG`
  - `ETCD_IMAGE=k8s.gcr.io/etcd:$ETCD_VERSION`

<!--
### Security Models
-->
### セキュリティ・モデル

<!--
There are two main options for security:
-->
セキュリティに対して２つの主なオプションがあります：

<!--
- Access the apiserver using HTTP.
  - Use a firewall for security.
  - This is easier to setup.
- Access the apiserver using HTTPS
  - Use https with certs, and credentials for user.
  - This is the recommended approach.
  - Configuring certs can be tricky.
-->
- apiserver に HTTP で接続
  - セキュリティのためにファイアウォールを使う
  - こちらはセットアップが簡単
- apiserver に HTTPS で接続
  -  https に証明書とユーザ用の信用証明書（credential）を使う
  - こちらが推奨する手法
  - 証明書の設定には注意が必要（トリッキー）

<!--
If following the HTTPS approach, you will need to prepare certs and credentials.
-->
HTTPS で取り組む場合は、証明書と信用証明書を自分で準備する必要があります。

<!--
#### Preparing Certs
-->
#### 信用証明書の準備

<!--
You need to prepare several certs:
-->
複数の証明書を準備する必要があります：

<!--
- The master needs a cert to act as an HTTPS server.
- The kubelets optionally need certs to identify themselves as clients of the master, and when
  serving its own API over HTTPS.
-->
- マスタを HTTPS サーバとして稼働するために必要な証明書
- kubelet がマスタのクライアントとして自身の認識と、HTTPS 経由で自身の API を提供するときは、オプションで証明書が必要

<!--
Unless you plan to have a real CA generate your certs, you will need
to generate a root cert and use that to sign the master, kubelet, and
kubectl certs. How to do this is described in the [authentication
documentation](/docs/admin/authentication/#creating-certificates/).
-->
実在する認証局（CA）で証明書を作るのを計画している場合を除き、ルート証明書（root cert）を作成し、これをマスタ、kubeletl、kubectl 証明書に対する署名で用いる必要があります。この方法については [認証ドキュメント](/docs/admin/authentication/#creating-certificates/)に記述しています。

<!--
You will end up with the following files (we will use these variables later on)
-->
最終的には以下のファイルを扱います（それぞれを後ほど使います）。

<!--
- `CA_CERT`
  - put in on node where apiserver runs, for example in `/srv/kubernetes/ca.crt`.
- `MASTER_CERT`
  - signed by CA_CERT
  - put in on node where apiserver runs, for example in `/srv/kubernetes/server.crt`
- `MASTER_KEY `
  - put in on node where apiserver runs, for example in `/srv/kubernetes/server.key`
- `KUBELET_CERT`
  - optional
- `KUBELET_KEY`
  - optional
-->
- `CA_CERT`
  - apiserver を動かすノードに置きます。設置例 `/srv/kubernetes/ca.crt`
- `MASTER_CERT`
  - CA_CERT で署名されたもの
  - apiserver を動かすノードに置きます。設置例 `/srv/kubernetes/server.crt`
- `MASTER_KEY `
  - apiserver を動かすノードに置きます。設置例 `/srv/kubernetes/server.key`
- `KUBELET_CERT`
  - オプション
- `KUBELET_KEY`
  - オプション

<!--
#### Preparing Credentials
-->
#### 証明書の準備

<!--
The admin user (and any users) need:
-->
管理者（と何らかのユーザ）が必要です：

<!--
  - a token or a password to identify them.
  - tokens are just long alphanumeric strings, 32 chars for example. See
    - `TOKEN=$(dd if=/dev/urandom bs=128 count=1 2>/dev/null | base64 | tr -d "=+/[:space:]" | dd bs=32 count=1 2>/dev/null)`
-->
  - トークンまたはパスワードで識別します
  - トークンは長いアルファベット文字列であり、例えば３２文字です。こちらをご覧ください。
    - `TOKEN=$(dd if=/dev/urandom bs=128 count=1 2>/dev/null | base64 | tr -d "=+/[:space:]" | dd bs=32 count=1 2>/dev/null)`

<!--
Your tokens and passwords need to be stored in a file for the apiserver
to read.  This guide uses `/var/lib/kube-apiserver/known_tokens.csv`.
The format for this file is described in the [authentication documentation](/docs/admin/authentication/).
-->
トークンとパスワードは apiserver が読み込めるファイルに格納する必要があります。このガイドでは  `/var/lib/kube-apiserver/known_tokens.csv` を使います。このファイルの書式については [認証ドキュメント](/docs/admin/authentication/) に詳細があります。

<!--
For distributing credentials to clients, the convention in Kubernetes is to put the credentials
into a [kubeconfig file](/docs/concepts/cluster-administration/authenticate-across-clusters-kubeconfig/).
-->
クライアントに証明書を配布するには、 Kubernetes では [kubeconfig ファイル](/docs/concepts/cluster-administration/authenticate-across-clusters-kubeconfig/) の中に証明書を入れるのが慣例です。

<!--
The kubeconfig file for the administrator can be created as follows:
-->
管理者は次の手順で kubeconfig ファイルを作成できます：

<!--
 - If you have already used Kubernetes with a non-custom cluster (for example, used a Getting Started
   Guide), you will already have a `$HOME/.kube/config` file.
 - You need to add certs, keys, and the master IP to the kubeconfig file:
    - If using the firewall-only security option, set the apiserver this way:
      - `kubectl config set-cluster $CLUSTER_NAME --server=http://$MASTER_IP --insecure-skip-tls-verify=true`
    - Otherwise, do this to set the apiserver ip, client certs, and user credentials.
      - `kubectl config set-cluster $CLUSTER_NAME --certificate-authority=$CA_CERT --embed-certs=true --server=https://$MASTER_IP`
      - `kubectl config set-credentials $USER --client-certificate=$CLI_CERT --client-key=$CLI_KEY --embed-certs=true --token=$TOKEN`
    - Set your cluster as the default cluster to use:
      - `kubectl config set-context $CONTEXT_NAME --cluster=$CLUSTER_NAME --user=$USER`
      - `kubectl config use-context $CONTEXT_NAME`
-->
 - 特に調整していない Kubernetes クラスタがあれば（たとえば、導入ガイドを使う場合）、既に `$HOME/.kube/config` ファイルがあります。
 - kubeconfig ファイルに対して証明書、鍵、マスタ IP を追加する必要があります：
    - ファイアウォールのみ（firewall-only）セキュリティ・オプションを使う場合、apiserver は、このように設定します：
      - `kubectl config set-cluster $CLUSTER_NAME --server=http://$MASTER_IP --insecure-skip-tls-verify=true`
    - あるいは、apiserver IP、クライアントの証明書とユーザの信用証明書（Credential）を設定するには、このようにします：
      - `kubectl config set-cluster $CLUSTER_NAME --certificate-authority=$CA_CERT --embed-certs=true --server=https://$MASTER_IP`
      - `kubectl config set-credentials $USER --client-certificate=$CLI_CERT --client-key=$CLI_KEY --embed-certs=true --token=$TOKEN`
    - クラスタをデフォルトで使用するクラスタに指定するには：
      - `kubectl config set-context $CONTEXT_NAME --cluster=$CLUSTER_NAME --user=$USER`
      - `kubectl config use-context $CONTEXT_NAME`

<!--
Next, make a kubeconfig file for the kubelets and kube-proxy.  There are a couple of options for how
many distinct files to make:
-->
次に、kubelet と kube-proxy に対応する設定ファイルを作ります。ファイルを作成するには様々な選択肢があります：

<!--
  1. Use the same credential as the admin
    - This is simplest to setup.
  1. One token and kubeconfig file for all kubelets, one for all kube-proxy, one for admin.
    - This mirrors what is done on GCE today
  1. Different credentials for every kubelet, etc.
    - We are working on this but all the pieces are not ready yet.
-->
  1. 管理者向けと同じ信用証明書（credential）を書く
    - 最もセットアップが簡単です。
  1. あるトークンと kubeconfig ファイルを kubelet 向けとし、あるものは kube-proxy 用、あるものは admin （管理用）とする。
    - 現在の GCE が行っている模倣です。
  1. kubelet ごとに異なる信用証明書（credential）を使う、等
    - 対応中ですが、まだ全ての部品が揃っていません。

<!--
You can make the files by copying the `$HOME/.kube/config` or by using the following template:
-->
ファイルは `$HOME/.kube/config` にコピーして作成できますが、あるいは以下のテンプレートを使っても作成できます：


```yaml
apiVersion: v1
kind: Config
users:
- name: kubelet
  user:
    token: ${KUBELET_TOKEN}
clusters:
- name: local
  cluster:
    certificate-authority: /srv/kubernetes/ca.crt
contexts:
- context:
    cluster: local
    user: kubelet
  name: service-account-context
current-context: service-account-context
```

<!--
Put the kubeconfig(s) on every node.  The examples later in this
guide assume that there are kubeconfigs in `/var/lib/kube-proxy/kubeconfig` and
`/var/lib/kubelet/kubeconfig`.
-->
kubeconfig を全てのノードに置きます。このガイドの後半では、`/var/lib/kube-proxy/kubeconfig` と `/var/lib/kubelet/kubeconfig` に kubeconfig ファイルがある想定です。

<!--
## Configuring and Installing Base Software on Nodes
-->
## ノードの設定と、土台となるソフトウェアのインストール

<!--
This section discusses how to configure machines to be Kubernetes nodes.
-->
このセクションではマシンを Kubernetes ノードに設定する方法を扱います。

<!--
You should run three daemons on every node:
--
各ノードで３つのデーモンを実行しなくてはいけません。
<!--
  - docker or rkt
  - kubelet
  - kube-proxy
-->
  - docker または rkt
  - kubelet
  - kube-proxy

<!--
You will also need to do assorted other configuration on top of a
base OS install.
-->

また、土台となる OS インストールに加えて、その上に様々な設定が必要になる場合があります。

<!--
Tip: One possible starting point is to setup a cluster using an existing Getting
Started Guide.   After getting a cluster running, you can then copy the init.d scripts or systemd unit files from that
cluster, and then modify them for use on your custom cluster.
-->
Tip：開始地点（スタート地点）としては既存の導入ガイドを使ってクラスタを作る方が望ましいかもしれません。クラスタが稼働後、クラスタから init.d スクリプトや systemd ユニットファイルをコピーし、カスタム・クラスタのために変更して利用できます。


### Docker
<!--
The minimum required Docker version will vary as the kubelet version changes.  The newest stable release is a good choice.  Kubelet will log a warning and refuse to start pods if the version is too old, so pick a version and try it.
-->
必要な Docker の最小バージョンは、Kubelet のバージョン変更に伴い変わります。最も新しい安定版（stable）リリースが良い選択です。もしも kubelet のバージョン番号が古く、警告が出てポッドの開始が拒否されるのであれば、新しいバージョンを導入してからお試しください。

<!--
If you previously had Docker installed on a node without setting Kubernetes-specific
options, you may have a Docker-created bridge and iptables rules.  You may want to remove these
as follows before proceeding to configure Docker for Kubernetes.
-->
もしも、以前に Docker をインストールしたものの、Kubernetes 対応オプションの設定が無かった場合は、Docker が作成したブリッジと iptables ルールがあるかもしれません。Kubernetes 対応の Docker 設定を進める前に、削除したい場合は、次のようにします。


```shell
iptables -t nat -F
ip link set docker0 down
ip link delete docker0
```
<!--
The way you configure docker will depend in whether you have chosen the routable-vip or overlay-network approaches for your network.
Some suggested docker options:
-->
Docker の設定にあたっては、ネットワーク上で到達可能な vip（routable-vip）やオーバレイ・ネットワークの取り組みを、どう選ぶかに依存します。推奨される Docker のオプションは：

<!--
  - create your own bridge for the per-node CIDR ranges, call it cbr0, and set `--bridge=cbr0` option on docker.
  - set `--iptables=false` so docker will not manipulate iptables for host-ports (too coarse on older docker versions, may be fixed in newer versions)
so that kube-proxy can manage iptables instead of docker.
  - `--ip-masq=false`
    - if you have setup PodIPs to be routable, then you want this false, otherwise, docker will
      rewrite the PodIP source-address to a NodeIP.
    - some environments (for example GCE) still need you to masquerade out-bound traffic when it leaves the cloud environment. This is very environment specific.
    - if you are using an overlay network, consult those instructions.
  - `--mtu=`
    - may be required when using Flannel, because of the extra packet size due to udp encapsulation
  - `--insecure-registry $CLUSTER_SUBNET`
    - to connect to a private registry, if you set one up, without using SSL.
-->
  - ノードごとの CIDR 範囲にブリッジを自分で作成し、その名称が cbr0 であれば、 docker で `--bridge=cbr0`を指定します。
  - `--iptables=false` を設定すると、docker はホスト・ポート（host-ports）の iptables を操作しません（docker の古いバージョンでは雑な設定したが、新しいバージョンでは修正されているかもしれません）。そのため、docker ではなく kube-proxy が iptables を管理可能になります。
  - `--ip-masq=false`
    - ポッド IP を到達可能（routable）にセットアップしたい場合は、こちらを false にします。そうしなければ、docker はポッド IP ソースアドレスをノード IP に｀書き換えるでしょう。
    - 環境によっては（GCE など）、クラウド環境から離れるには、アウトバウンドにトラフィックをマスカレードする必要があります。これは環境にとても依存します。
    - オーバレイ・ネットワークを使うには、これらの手順を調べます。
  - `--mtu=`
    - Flannel 使用時は、UDP はカプセル化によるパケットサイズが大きくなるため、必要となるかもしれません。
  - `--insecure-registry $CLUSTER_SUBNET`
    - SSL を使用しないプライベート・レジストリ に対して接続をしたい場合。

<!--
You may want to increase the number of open files for docker:
-->
docker が開けるファイル数を増加したい場合：

   - `DOCKER_NOFILE=1000000`

<!--
Where this config goes depends on your node OS.  For example, GCE's Debian-based distro uses `/etc/default/docker`.
-->
この設定場所はノードの OS に依存します。たとえば、GCE の Debian を土台とするディストリビューションは `/etc/default/docker` を使います。

<!--
Ensure docker is working correctly on your system before proceeding with the rest of the
installation, by following examples given in the Docker documentation.
-->
確実にシステム上で docker を正しく動かすには、この手順の後半に進む前に、Docker ドキュメントの記述を理解ください。

### rkt

<!--
[rkt](https://github.com/coreos/rkt) is an alternative to Docker.  You only need to install one of Docker or rkt.
The minimum version required is [v0.5.6](https://github.com/coreos/rkt/releases/tag/v0.5.6).
-->
[rkt](https://github.com/coreos/rkt) は Docker の代わりとなるものです。Docker か rkt のどちらか一方のみインストールが必要です。必要な最小バージョンは [v0.5.6](https://github.com/coreos/rkt/releases/tag/v0.5.6) です。

<!--
[systemd](http://www.freedesktop.org/wiki/Software/systemd/) is required on your node to run rkt.  The
minimum version required to match rkt v0.5.6 is
[systemd 215](http://lists.freedesktop.org/archives/systemd-devel/2014-July/020903.html).
-->
ノード上で rkt を実行するには [systemd](http://www.freedesktop.org/wiki/Software/systemd/) が必要です。rkt v0.5.6 に対応する必要バージョンは [systemd 215](http://lists.freedesktop.org/archives/systemd-devel/2014-July/020903.html) です。

<!--
[rkt metadata service](https://github.com/coreos/rkt/blob/master/Documentation/networking/overview.md) is also required
for rkt networking support.  You can start rkt metadata service by using command like
`sudo systemd-run rkt metadata-service`
-->
また、rkt ネットワーク機能のサポートには [rkt メタデータ・サービス（metadata service）](https://github.com/coreos/rkt/blob/master/Documentation/networking/overview.md) も必要です。rkt メタデータ・サービスを開始するには、コマンドラインで `sudo systemd-run rkt metadata-service` を使います。

<!--
Then you need to configure your kubelet with flag:
-->
それから、kubelet にフラグを設定する必要があります：

  - `--container-runtime=rkt`

### kubelet
<!--
All nodes should run kubelet.  See [Software Binaries](#software-binaries).
-->
全てのノードで kubelet を実行すべきです。詳細は[ソフトウェアのバイナリ](#software-binaries)をご覧ください。

<!--
Arguments to consider:
-->
検討する引数：
<!--
  - If following the HTTPS security approach:
    - `--kubeconfig=/var/lib/kubelet/kubeconfig`
  - Otherwise, if taking the firewall-based security approach
  - `--config=/etc/kubernetes/manifests`
  - `--cluster-dns=` to the address of the DNS server you will setup (see [Starting Cluster Services](#starting-cluster-services).)
  - `--cluster-domain=` to the dns domain prefix to use for cluster DNS addresses.
  - `--docker-root=`
  - `--root-dir=`
  - `--pod-cidr=` The CIDR to use for pod IP addresses, only used in standalone mode.  In cluster mode, this is obtained from the master.
  - `--register-node` (described in [Node](/docs/admin/node/) documentation.)
-->
  - HTTPS セキュリティ方式に対応するには:
    - `--kubeconfig=/var/lib/kubelet/kubeconfig`
  - あるいは、ファイアウォールを土台とするセキュリティ方式であれば
  - `--config=/etc/kubernetes/manifests`
  - `--cluster-dns=` セットアップする DNS サーバのアドレス([クラスタ・サービスの開始 Cluster Services](#starting-cluster-services)をご覧ください。)
  - `--cluster-domain=` クラスタ DNS アドレスが使う DNS ドメイン・プレフィックス（訳者注；先頭に付ける文字）
  - `--docker-root=`
  - `--root-dir=`
  - `--pod-cidr=` ポッド IP アドレスが使う CIDR であり、スタンドアロン・モードでのみ使います。クラスタ・モードでは、CIDR はマスタから取得します。
  - `--register-node` ([ノード](/jp/docs/admin/node/) ドキュメントに記述があります)

### kube-proxy

<!--
All nodes should run kube-proxy.  (Running kube-proxy on a "master" node is not
strictly required, but being consistent is easier.)   Obtain a binary as described for
kubelet.
-->
全てのノードで kube-proxy を実行すべきです（厳密には "master" ノード上で実行する必要はありませんが、整合性を取るのが簡単です）。バイナリの取得は kubelet に記述してあります。

<!--
Arguments to consider:
-->
検討する引数：
<!--
  - If following the HTTPS security approach:
    - `--master=https://$MASTER_IP`
    - `--kubeconfig=/var/lib/kube-proxy/kubeconfig`
  - Otherwise, if taking the firewall-based security approach
    - `--master=http://$MASTER_IP`
ｰｰ>
  - HTTPS セキュリティ方式に対応するには：
    - `--master=https://$MASTER_IP`
    - `--kubeconfig=/var/lib/kube-proxy/kubeconfig`
  - あるいは、ファイアウォールを土台とするセキュリティ方式であれば
    - `--master=http://$MASTER_IP`

<!--
Note that on some Linux platforms, you may need to manually install the
`conntrack` package which is a dependency of kube-proxy, or else kube-proxy
cannot be started successfully.
-->
Linux プラットフォームによっては、 `kube-proxy` が依存する `conntrack` パッケージを手動でインストールする必要があるでしょう。もしそうでなければ、起動できません。

<!--
For more details on debugging kube-proxy problems, please refer to
[Debug Services](/docs/tasks/debug-application-cluster/debug-service/)
-->
kube-proxy 問題のデバッグに関する詳細は [サービスのデバッグ](/docs/tasks/debug-application-cluster/debug-service/) を参照ください。

<!--
### Networking
-->
### ネットワーク機能 {#Networking}

<!--
Each node needs to be allocated its own CIDR range for pod networking.
Call this `NODE_X_POD_CIDR`.
-->
各ノードはポッド・ネットワーク用に自身の CIDR 範囲を割り当てる必要があります。これを `NODE_X_POD_CIDR` と呼びます。

<!--
A bridge called `cbr0` needs to be created on each node.  The bridge is explained
further in the [networking documentation](/docs/concepts/cluster-administration/networking/).  The bridge itself
needs an address from `$NODE_X_POD_CIDR` - by convention the first IP.  Call
this `NODE_X_BRIDGE_ADDR`.  For example, if `NODE_X_POD_CIDR` is `10.0.0.0/16`,
then `NODE_X_BRIDGE_ADDR` is `10.0.0.1/16`.  NOTE: this retains the `/16` suffix
because of how this is used later.
-->
各ノードでは `cbr0` と呼ぶブリッジの作成が必要です。このブリッジの詳細は [ネットワーク機能ドキュメント](/jp/docs/concepts/cluster-administration/networking/) で説明しています。ブリッジ自身が `NODE_X_POD_CIDR` からの IP アドレスを必要とします。これは慣例として１つめの IP であり、 `NODE_X_BRIDGE_ADDR` と呼びます。たとえば、 `NODE_X_POD_CIDR` が `10.0.0.0/16` であれば、`NODE_X_BRIDGE_ADDR` は `10.0.0.1/16` です。メモ：保持している`/16` サフィックスをどう扱うかは、後から使います。

<!--
If you have turned off Docker's IP masquerading to allow pods to talk to each
other, then you may need to do masquerading just for destination IPs outside
the cluster network.  For example:
-->
もしもポッドがお互いに通信できるよう、Docker の IP マスカレード機能を無効にしている場合は、送信先 IP がクラスタ・ネットワーク外になるようマスカレーディングが必要になる場合があります。例：


```shell
iptables -t nat -A POSTROUTING ! -d ${CLUSTER_SUBNET} -m addrtype ! --dst-type LOCAL -j MASQUERADE
```

<!--
This will rewrite the source address from
the PodIP to the Node IP for traffic bound outside the cluster, and kernel
[connection tracking](http://www.iptables.info/en/connection-state.html)
will ensure that responses destined to the node still reach
the pod.
-->
これはポッド IP をソースとするアドレスを、ノード IP にに書き換えるもので、通信がクラスタの外に流れるようにします。また、カーネルの[connection tracking](http://www.iptables.info/en/connection-state.html) によって、到達したポッドからもノードが確実に応答できるようにします。

<!--
NOTE: This is environment specific.  Some environments will not need
any masquerading at all.  Others, such as GCE, will not allow pod IPs to send
traffic to the internet, but have no problem with them inside your GCE Project.
-->
メモ：これは環境により異なります。ある環境ではマスカレードは不要でしょう。一方で、GCE のような所では、ポッドの IP がインターネットにトラフィックを遅れませんが、GCE プロジェクト内では何ら問題はありません。

<!--
### Other
-->

### その他 {#Other}

<!--
- Enable auto-upgrades for your OS package manager, if desired.
- Configure log rotation for all node components (for example using [logrotate](http://linux.die.net/man/8/logrotate)).
- Setup liveness-monitoring (for example using [supervisord](http://supervisord.org/)).
- Setup volume plugin support (optional)
  - Install any client binaries for optional volume types, such as `glusterfs-client` for GlusterFS
    volumes.
-->
- 必要に応じて OS パッケージ・マネージャの自動更新を有効にします。
- 全て音ノード構成要素のログ・ローテーションを設定します（たとえば [logrotate](http://linux.die.net/man/8/logrotate) を使います）。
- 死活監視をセットアップします（たとえば [supervisord](http://supervisord.org/) を使います）.
- ボリューム・プラグイン・サポートをセットアップします（オプション）。
  - オプションでボリューム・タイプのクライアンド・バイナリをインストールします。GlusterFS ボリュームであれば `glusterfs-client` です。

<!--
### Using Configuration Management
-->

### 構成管理を使う {#using-configuration-management}
<!--
The previous steps all involved "conventional" system administration techniques for setting up
machines.  You may want to use a Configuration Management system to automate the node configuration
process.  There are examples of [Saltstack](/docs/admin/salt/), Ansible, Juju, and CoreOS Cloud Config in the
various Getting Started Guides.
-->
これまでの手順において、マシンのセットアップに必要なのは、すべて「旧来の」システム管理技術でした。ノード設定処理を自動化するために、設定管理システムを使いたいかもしれません。様々な導入ガイドに、 [Saltstack](/docs/admin/salt/)、Ansible、Juju、CoreOS Cloud Config の例があります。

<!--
## Bootstrapping the Cluster
-->
## クラスタの初回起動（ブートストラップ）{#bootstrapping-the-cluster}

<!--
While the basic node services (kubelet, kube-proxy, docker) are typically started and managed using
traditional system administration/automation approaches, the remaining *master* components of Kubernetes are
all configured and managed *by Kubernetes*:
-->
基本ノード・サービス（kubelet、kube-proxy、docker）は、伝統的なシステム管理手法や自動化手法を用いて、一般的に起動および管理できます。そして Kubernetes の *マスタ* 構成要素は、全て *Kubernetes* によって設定・管理されます。

<!--
  - Their options are specified in a Pod spec (yaml or json) rather than an /etc/init.d file or
    systemd unit.
  - They are kept running by Kubernetes rather than by init.
-->
  - これらのオプションは /etc/init.d ファイルや systemd unit ではなく、ポッドの spec（yaml か json）で指定します。
  - これらは init ではなく、Kubernetes によって実行を保たれます。

### etcd

<!--
You will need to run one or more instances of etcd.
-->
１つまたは複数の etcd インスタンスを実行する必要があります。

<!--
  - Highly available and easy to restore - Run 3 or 5 etcd instances with, their logs written to a directory backed
    by durable storage (RAID, GCE PD)
  - Not highly available, but easy to restore - Run one etcd instance, with its log written to a directory backed
    by durable storage (RAID, GCE PD).
-->
  - 可用性を高めて、簡単に復旧する - etcd インスタンスを３つまたは５つ起動し、持続的なストレージ（RAID、GCE PD）が背後にあるディレクトリにログを書き出します。
  - 可用性が高くなくても、簡単に復旧する - etcd インスタンスを１つ起動し、持続的なストレージ（RAID、GCE PD）が背後にあるディレクトリにログを書き出します。
<!--
    {{< note >}}**Note:** May result in operations outages in case of
    instance outage. {{< /note >}}
  - Highly available - Run 3 or 5 etcd instances with non durable storage.
  
    {{< note >}}**Note:** Log can be written to non-durable storage
    because storage is replicated.{{< /note >}}
-->
    {{< note >}}**メモ:**インスタンスの障害によって、結果的に機能障害に至る場合があります。 {{< /note >}}
  - 可用性を高める - 持続的ではないストレージに、etcd を３または５つ起動する。
  
    {{< note >}}**メモ:** ストレージは複製されるため、持続的ではないストレージにもログを書き出します。{{< /note >}}

<!--
See [cluster-troubleshooting](/docs/admin/cluster-troubleshooting/) for more discussion on factors affecting cluster
availability.
-->
クラスタ可能性に影響する要素の議論は、[クラスタ・トラブルシューティング](/jp/docs/admin/cluster-troubleshooting/) をご覧ください。

<!--
To run an etcd instance:
-->
etcd インスタンスを実行するには：

<!--
1. Copy [`cluster/gce/manifests/etcd.manifest`](https://github.com/kubernetes/kubernetes/blob/master/cluster/gce/manifests/etcd.manifest)
1. Make any modifications needed
1. Start the pod by putting it into the kubelet manifest directory
-->
1. [`cluster/gce/manifests/etcd.manifest`](https://github.com/kubernetes/kubernetes/blob/master/cluster/gce/manifests/etcd.manifest) をコピーします。
1. 必要に応じて変更を加えます。
1. kubelet マニフェスト・ディレクトリに置き、ポッドを開始します。

<!--
### Apiserver, Controller Manager, and Scheduler
-->
### api サーバ、コントローラ・マネージャ、スケジューラ {#apiserver-controller-manager-and-scheduler}

<!--
The apiserver, controller manager, and scheduler will each run as a pod on the master node.
-->
api サーバ、コントロール・プレーン、スケジューラは、それぞれマスタ・ノード上のポッドとして実行されます。

<!--
For each of these components, the steps to start them running are similar:
-->
これらの構成要素ごとの、起動する手順は似ています：

<!--
1. Start with a provided template for a pod.
1. Set the `HYPERKUBE_IMAGE` to the values chosen in [Selecting Images](#selecting-images).
1. Determine which flags are needed for your cluster, using the advice below each template.
1. Set the flags to be individual strings in the command array (for example $ARGN below)
1. Start the pod by putting the completed template into the kubelet manifest directory.
1. Verify that the pod is started.
-->
1. ポッド用のテンプレートを与えて起動します。
1. [イメージの選択](#selecting-images) で選択したイメージ名を `HYPERKUBE_IMAGE に入れます。
1. 後述するテンプレートに関するアドバイスを用い、クラスタに応じて必要なフラグを決めます。
1. コマンド配列（以降の例では $ARGN）でフラグを個々の文字列として設定します。
1. 完成したテンプレートを kubelet マニフェスト・ディレクトリに置き、ポッドを起動します。
1. ポッドの輝度を確認します。

<!--
#### Apiserver pod template
-->
#### api サーバのポッド・テンプレート {#apiserver-pod-template}

```json
{
  "kind": "Pod",
  "apiVersion": "v1",
  "metadata": {
    "name": "kube-apiserver"
  },
  "spec": {
    "hostNetwork": true,
    "containers": [
      {
        "name": "kube-apiserver",
        "image": "${HYPERKUBE_IMAGE}",
        "command": [
          "/hyperkube",
          "apiserver",
          "$ARG1",
          "$ARG2",
          ...
          "$ARGN"
        ],
        "ports": [
          {
            "name": "https",
            "hostPort": 443,
            "containerPort": 443
          },
          {
            "name": "local",
            "hostPort": 8080,
            "containerPort": 8080
          }
        ],
        "volumeMounts": [
          {
            "name": "srvkube",
            "mountPath": "/srv/kubernetes",
            "readOnly": true
          },
          {
            "name": "etcssl",
            "mountPath": "/etc/ssl",
            "readOnly": true
          }
        ],
        "livenessProbe": {
          "httpGet": {
            "scheme": "HTTP",
            "host": "127.0.0.1",
            "port": 8080,
            "path": "/healthz"
          },
          "initialDelaySeconds": 15,
          "timeoutSeconds": 15
        }
      }
    ],
    "volumes": [
      {
        "name": "srvkube",
        "hostPath": {
          "path": "/srv/kubernetes"
        }
      },
      {
        "name": "etcssl",
        "hostPath": {
          "path": "/etc/ssl"
        }
      }
    ]
  }
}
```
<!--
Here are some apiserver flags you may need to set:
-->
必要に応じて api サーバのフラグを指定します：

<!--
- `--cloud-provider=` see [cloud providers](#cloud-providers)
- `--cloud-config=` see [cloud providers](#cloud-providers)
- `--address=${MASTER_IP}` *or* `--bind-address=127.0.0.1` and `--address=127.0.0.1` if you want to run a proxy on the master node.
- `--service-cluster-ip-range=$SERVICE_CLUSTER_IP_RANGE`
- `--etcd-servers=http://127.0.0.1:4001`
- `--tls-cert-file=/srv/kubernetes/server.cert`
- `--tls-private-key-file=/srv/kubernetes/server.key`
- `--enable-admission-plugins=$RECOMMENDED_LIST`
  - See [admission controllers](/docs/admin/admission-controllers/) for recommended arguments.
- `--allow-privileged=true`, only if you trust your cluster user to run pods as root.
-->
- `--cloud-provider=` [クラウド・プロバイダ](#cloud-providers) をご覧ください
- `--cloud-config=` [クラウド・プロバイダ](#cloud-providers) をご覧ください
- `--address=${MASTER_IP}` *または* `--bind-address=127.0.0.1` と `--address=127.0.0.1` を、マスタ・ノード上でプロキシを実行したい場合
- `--service-cluster-ip-range=$SERVICE_CLUSTER_IP_RANGE`
- `--etcd-servers=http://127.0.0.1:4001`
- `--tls-cert-file=/srv/kubernetes/server.cert`
- `--tls-private-key-file=/srv/kubernetes/server.key`
- `--enable-admission-plugins=$RECOMMENDED_LIST`
  - 推奨する引数は [承認制御（admission controllers）](/jp/docs/admin/admission-controllers/) をご覧ください
- `--allow-privileged=true` は、クラスタの利用者がポッドを root として実行するのが信頼できる時のみ

<!--
If you are following the firewall-only security approach, then use these arguments:
-->
ファイアウォールのみのセキュリティ方式であれば、これらの引数を使います：

- `--token-auth-file=/dev/null`
- `--insecure-bind-address=$MASTER_IP`
- `--advertise-address=$MASTER_IP`

<!--
If you are using the HTTPS approach, then set:
-->
HTTPS 方式を使う場合、こちらを指定します：

- `--client-ca-file=/srv/kubernetes/ca.crt`
- `--token-auth-file=/srv/kubernetes/known_tokens.csv`
- `--basic-auth-file=/srv/kubernetes/basic_auth.csv`

<!--
This pod mounts several node file system directories using the  `hostPath` volumes.  Their purposes are:
-->
このポッドは `hostPath` ボリュームを用い、複数のノード・ファイルシステム・ディレクトリをマウントます。 それらの目的は：

```
- The `/etc/ssl` mount allows the apiserver to find the SSL root certs so it can
  authenticate external services, such as a cloud provider.
  - This is not required if you do not use a cloud provider (bare-metal for example).
- The `/srv/kubernetes` mount allows the apiserver to read certs and credentials stored on the
  node disk.  These could instead be stored on a persistent disk, such as a GCE PD, or baked into the image.
- Optionally, you may want to mount `/var/log` as well and redirect output there (not shown in template).
  - Do this if you prefer your logs to be accessible from the root filesystem with tools like journalctl.
```
- `/etc/ssl` のマウントで api サーバが SSL ルート証明書を見つけられるため、クラウド・プロバイダのような外部サービスを認証できます。
  - クラウド・プロバイダを使わなければ（たとえばベアメタル）、こちらは不要です。
- `/srv/kubernetes` のマウントで api サーバがノード・ディスク上に保管された証明書と信用証明書（credential）を読めるようになります。これにより、GCE PD のような永続的なディスクやイメージ内に保管する代わりとなります。
- オプションとして、 `/var/log` も同様にマウントし、そこに出力先を変えます（リダイレクトします）。（テンプレートにはありません）
  - ログはルート・ファイルシステムから journalctl のようなツールを使い、アクセスできるほうが好ましいでしょう。

<!--
*TODO* document proxy-ssh setup.
-->
*TODO* proxy-ssh セットアップ・ドキュメント

<!--
##### Cloud Providers
-->
##### クラウド・プロバイダ {#cloud-providers}
<!--
Apiserver supports several cloud providers.
-->
apiサーバは複数のクラウド・プロバイダをサポートします。

<!--
- options for `--cloud-provider` flag are `aws`, `azure`, `cloudstack`, `fake`, `gce`, `mesos`, `openstack`, `ovirt`, `rackspace`, `vsphere`, or unset.
- unset used for bare metal setups.
- support for new IaaS is added by contributing code [here](https://releases.k8s.io/{{< param "githubbranch" >}}/pkg/cloudprovider/providers)
-->
- `--cloud-provider` フラグ用のオプションは `aws`、 `azure`、 `cloudstack`、 `fake`、 `gce`、 `mesos`、 `openstack`、 `ovirt`、 `rackspace`、 `vsphere`、あるいは未指定です。
- 未指定はベアメタルのセットアップに使います。
- [こちらから](https://releases.k8s.io/{{< param "githubbranch" >}}/pkg/cloudprovider/providers) コードによる貢献が追加されると、新しい IaaS がサポートされます。

<!--
Some cloud providers require a config file. If so, you need to put config file into apiserver image or mount through hostPath.
-->
いくつかのプロバイダは設定ファイルが必要です。そのため、設定ファイルを api サーバのイメージに入れるか、hostPath のマウントが必要です。

<!--
- `--cloud-config=` set if cloud provider requires a config file.
- Used by `aws`, `gce`, `mesos`, `openstack`, `ovirt` and `rackspace`.
- You must put config file into apiserver image or mount through hostPath.
- Cloud config file syntax is [Gcfg](https://code.google.com/p/gcfg/).
- AWS format defined by type [AWSCloudConfig](https://releases.k8s.io/{{< param "githubbranch" >}}/pkg/cloudprovider/providers/aws/aws.go)
- There is a similar type in the corresponding file for other cloud providers.
-->

- クラウド・プロバイダで設定ファイルが必要な場合は `--cloud-config=` で指定します。
  - `aws`、 `gce`、 `mesos`、 `openstack`、 `ovirt`、 `rackspace` で使います。
  - 設定ファイルは api サーバ・イメージに置くか hotPath を通してマウントする必要があります。
- Cloud 設定ファイルの構文は [Gcfg](https://code.google.com/p/gcfg/) です。
  - AWS のフォーマットはタイプ [AWSCloudConfig](https://releases.k8s.io/{{< param "githubbranch" >}}/pkg/cloudprovider/providers/aws/aws.go) で定義されています。
  - 他のクラウド・プロバイダも類似のファイル・タイプです。

<!--
#### Scheduler pod template
-->
#### スケジューラ・ポッドのテンプレート {#scheduler-pod-template}

<!--
Complete this template for the scheduler pod:
-->
スケジューラ・ポッド用向けに完成したテンプレートです：

```json
{
  "kind": "Pod",
  "apiVersion": "v1",
  "metadata": {
    "name": "kube-scheduler"
  },
  "spec": {
    "hostNetwork": true,
    "containers": [
      {
        "name": "kube-scheduler",
        "image": "$HYPERKUBE_IMAGE",
        "command": [
          "/hyperkube",
          "scheduler",
          "--master=127.0.0.1:8080",
          "$SCHEDULER_FLAG1",
          ...
          "$SCHEDULER_FLAGN"
        ],
        "livenessProbe": {
          "httpGet": {
            "scheme": "HTTP",
            "host": "127.0.0.1",
            "port": 10251,
            "path": "/healthz"
          },
          "initialDelaySeconds": 15,
          "timeoutSeconds": 15
        }
      }
    ]
  }
}
```
<!--
Typically, no additional flags are required for the scheduler.
-->
通常、スケジューラ向けには追加のフラグは不要です。

<!--
Optionally, you may want to mount `/var/log` as well and redirect output there.
-->
オプションで、 `/var/log` をマウントし、出力をそちらに変えてもよいでしょう。

<!--
#### Controller Manager Template
-->
#### コントローラ・マネージャ・テンプレート {#controller-manager-template}

<!--
Template for controller manager pod:
-->
コントローラ・マネージャ・ポッド用のテンプレートです：

```json
{
  "kind": "Pod",
  "apiVersion": "v1",
  "metadata": {
    "name": "kube-controller-manager"
  },
  "spec": {
    "hostNetwork": true,
    "containers": [
      {
        "name": "kube-controller-manager",
        "image": "$HYPERKUBE_IMAGE",
        "command": [
          "/hyperkube",
          "controller-manager",
          "$CNTRLMNGR_FLAG1",
          ...
          "$CNTRLMNGR_FLAGN"
        ],
        "volumeMounts": [
          {
            "name": "srvkube",
            "mountPath": "/srv/kubernetes",
            "readOnly": true
          },
          {
            "name": "etcssl",
            "mountPath": "/etc/ssl",
            "readOnly": true
          }
        ],
        "livenessProbe": {
          "httpGet": {
            "scheme": "HTTP",
            "host": "127.0.0.1",
            "port": 10252,
            "path": "/healthz"
          },
          "initialDelaySeconds": 15,
          "timeoutSeconds": 15
        }
      }
    ],
    "volumes": [
      {
        "name": "srvkube",
        "hostPath": {
          "path": "/srv/kubernetes"
        }
      },
      {
        "name": "etcssl",
        "hostPath": {
          "path": "/etc/ssl"
        }
      }
    ]
  }
}
```

<!--
Flags to consider using with controller manager:
-->
コントローラ・マネージャとして使用を検討するフラグ：
<!--
 - `--cluster-cidr=`, the CIDR range for pods in cluster.
 - `--allocate-node-cidrs=`, if you are using `--cloud-provider=`, allocate and set the CIDRs for pods on the cloud provider.
 - `--cloud-provider=` and `--cloud-config` as described in apiserver section.
 - `--service-account-private-key-file=/srv/kubernetes/server.key`, used by the [service account](/docs/user-guide/service-accounts) feature.
 - `--master=127.0.0.1:8080`
-->
 - `--cluster-cidr=` はクラスタ内のポッド向けの CIDR 範囲です。
 - `--allocate-node-cidrs=` と、もしも `--cloud-provider=` を使うのであれば、クラウド・プロバイダ上のポッドに割り当てて使う CIDR を指定します。
 - `--cloud-provider=` と `--cloud-config` を api サーバの箇所と同じように記述します。
 - `--service-account-private-key-file=/srv/kubernetes/server.key` を [サービスアカウントservice account](/jp/docs/user-guide/service-accounts) 機能のために使います。
 - `--master=127.0.0.1:8080`

<!--
#### Starting and Verifying Apiserver, Scheduler, and Controller Manager
-->
#### api サーバ、スケジューラ、コントローラ・マネージャの起動と確認{#starting-and-verifying-ppiserver-scheduler-and-controller-manager}

<!--
Place each completed pod template into the kubelet config dir
(whatever `--config=` argument of kubelet is set to, typically
`/etc/kubernetes/manifests`).  The order does not matter: scheduler and
controller manager will retry reaching the apiserver until it is up.
-->
それぞれの完全なポッド・テンプレートを kubelet 設定ファイルのディレクトリに置きます（ kubelet の引数を `--config=` で設定します。通常は `/etc/kubernetes/manifests` です）。順番はどちらでも構いません。つまり、スケジューラとコントローラ・マネージャは api サーバが稼働するまでリトライし続けます。

<!--
Use `ps` or `docker ps` to verify that each process has started.  For example, verify that kubelet has started a container for the apiserver like this:
-->
`ps` か `docker ps` で各プロセスが起動しているのを確認します。たとえば、kubelet で apiserver 用のコンテナが起動しているかどうかの確認は、次のようにします。

```shell
$ sudo docker ps | grep apiserver
5783290746d5        k8s.gcr.io/kube-apiserver:e36bf367342b5a80d7467fd7611ad873            "/bin/sh -c '/usr/lo'"    10 seconds ago      Up 9 seconds                              k8s_kube-apiserver.feb145e7_kube-apiserver-kubernetes-master_default_eaebc600cf80dae59902b44225f2fc0a_225a4695
```

<!--
Then try to connect to the apiserver:
-->
それから、api サーバに接続できるかどうか試します：


```shell
$ echo $(curl -s http://localhost:8080/healthz)
ok
$ curl -s http://localhost:8080/api
{
  "versions": [
    "v1"
  ]
}
```
<!--
If you have selected the `--register-node=true` option for kubelets, they will now begin self-registering with the apiserver.
You should soon be able to see all your nodes by running the `kubectl get nodes` command.
Otherwise, you will need to manually create node objects.
-->
もしも kubelet に `--register-node=true`  オプションを選択した場合、api サーバに対して自己登録（self-registering）を開始します。`kubectl get nodes` コマンドの実行により、まもなく全てのノードが見えるようになります。そうでなければ、ノード・オブジェクトを手動で追加する必要があります。

<!--
### Starting Cluster Services
-->
### クラスタ・サービスの開始 {#staring-cluster-services}

<!--
You will want to complete your Kubernetes clusters by adding cluster-wide
services.  These are sometimes called *addons*, and [an overview
of their purpose is in the admin guide](/docs/admin/cluster-components/#addons).
-->
クラスタ全体のサービス（cluster-wide services）を Kubernetes クラスタ全体で揃えられます。これは「アドオン」（addons）と呼ばれており、[役割は管理者ガイドの概要](/jp/docs/admin/cluster-components/#addons).をご覧ください。

<!--
Notes for setting up each cluster service are given below:
-->
各クラスタ・サービスのセットアップ時に気を付けるのは、以下の通りです：

<!--
* Cluster DNS:
  * Required for many Kubernetes examples
  * [Setup instructions](http://releases.k8s.io/{{< param "githubbranch" >}}/cluster/addons/dns/)
  * [Admin Guide](/docs/concepts/services-networking/dns-pod-service/)
* Cluster-level Logging
  * [Cluster-level Logging Overview](/docs/user-guide/logging/overview/)
  * [Cluster-level Logging with Elasticsearch](/docs/user-guide/logging/elasticsearch/)
  * [Cluster-level Logging with Stackdriver Logging](/docs/user-guide/logging/stackdriver/)
* Container Resource Monitoring
  * [Setup instructions](http://releases.k8s.io/{{< param "githubbranch" >}}/cluster/addons/cluster-monitoring/)
* GUI
  * [Setup instructions](https://github.com/kubernetes/dashboard)
-->
* クラスタ DNS:
  * 多くの Kubernetes 手本（examples）が必要です。
  * [セットアップ手順](http://releases.k8s.io/{{< param "githubbranch" >}}/cluster/addons/dns/)
  * [管理ガイド](/jp/docs/concepts/services-networking/dns-pod-service/)
* クラスタ段階のログ記録（Cluster-level Logging）
  * [クラスタ段階のログ記録概要](/jp/docs/user-guide/logging/overview/)
  * [Elasticserch によるクラスタ段階のログ記録](/jp/docs/user-guide/logging/elasticsearch/)
  * [Stackdriver Logging によるクラスタ段階のログ記録](/docs/user-guide/logging/stackdriver/)
* コンテナ資源の監視
  * [セットアップ手順](http://releases.k8s.io/{{< param "githubbranch" >}}/cluster/addons/cluster-monitoring/)
* GUI
  * [セットアップ手順](https://github.com/kubernetes/dashboard)

<!--
## Troubleshooting
-->
## トラブルシューティング {#troubleshooting}

<!--
### Running validate-cluster
-->
### 正しいクラスタの稼働 {#running-validate-cluster}

<!--
`cluster/validate-cluster.sh` is used by `cluster/kube-up.sh` to determine if
the cluster start succeeded.
-->
クラスタの開始が成功しているかどうかを確かめるために、`cluster/validate-cluster.sh` が  `cluster/kube-up.sh` によって使われます。

<!--
Example usage and output:
-->
使い方例と結果：

```shell
KUBECTL_PATH=$(which kubectl) NUM_NODES=3 KUBERNETES_PROVIDER=local cluster/validate-cluster.sh
Found 3 node(s).
NAME                    STATUS    AGE     VERSION
node1.local             Ready     1h      v1.6.9+a3d1dfa6f4335
node2.local             Ready     1h      v1.6.9+a3d1dfa6f4335
node3.local             Ready     1h      v1.6.9+a3d1dfa6f4335
Validate output:
NAME                 STATUS    MESSAGE              ERROR
controller-manager   Healthy   ok
scheduler            Healthy   ok
etcd-1               Healthy   {"health": "true"}
etcd-2               Healthy   {"health": "true"}
etcd-0               Healthy   {"health": "true"}
Cluster validation succeeded
```
<!--
### Inspect pods and services
-->
### ポッドとサービスの調査 {#inspect-pods-and-services}

<!--
Try to run through the "Inspect your cluster" section in one of the other Getting Started Guides, such as [GCE](/docs/getting-started-guides/gce/#inspect-your-cluster).
You should see some services.  You should also see "mirror pods" for the apiserver, scheduler and controller-manager, plus any add-ons you started.
-->
他の導入ガイドにある "クラスタの調査" セクションを通しての実行をお試しください。[GCE](/jp/docs/getting-started-guides/gce/#inspect-your-cluster) であればこちらです。

<!--
### Try Examples
-->
### 例を試す {#try-examples}

<!--
At this point you should be able to run through one of the basic examples, such as the [nginx example](/examples/application/deployment.yaml).
-->
この時点では、 [nginx 例](/examples/application/deployment.yaml) のように、基本的な例を通して実行すべきでしょう。

<!--
### Running the Conformance Test
-->
### 適合試験の実行 {#running-the-conformance-test}

<!--
You may want to try to run the [Conformance test](http://releases.k8s.io/{{< param "githubbranch" >}}/test/e2e_node/conformance/run_test.sh).  Any failures may give a hint as to areas that need more attention.
-->
[適合試験（Conformance test）](http://releases.k8s.io/{{< param "githubbranch" >}}/test/e2e_node/conformance/run_test.sh). の実行を試みましょう。何らかの問題があれば、より注意を払うべき場所のヒントが得られます。

<!--
### Networking
-->
### ネットワーク機能 {#networking}

<!--
The nodes must be able to connect to each other using their private IP. Verify this by
pinging or SSH-ing from one node to another.
-->
ノードはお互いにプライベート IP を通して接続できる必要があります。あるノードから別のノードに ping や SSH できるかどうかで確認ください。


<!--
### Getting Help
-->
### ヘルプを得るには {#getting-help}

<!--
If you run into trouble, please see the section on [troubleshooting](/docs/getting-started-guides/gce/#troubleshooting), post to the
[kubernetes-users group](https://groups.google.com/forum/#!forum/kubernetes-users), or come ask questions on [Slack](/docs/troubleshooting#slack).
-->
トラブルに遭遇した場合は、 [トラブルシューティング](/jp/docs/getting-started-guides/gce/#troubleshooting)、[kubernetes-users group](https://groups.google.com/forum/#!forum/kubernetes-users)への投稿、あるいは [Slack](/docs/troubleshooting#slack) でお訊ねください。

<!--
## Support Level
-->
## サポート・レベル {#support-level}

<!--
IaaS Provider        | Config. Mgmt | OS     | Networking  | Docs                                              | Conforms | Support Level
-------------------- | ------------ | ------ | ----------  | ---------------------------------------------     | ---------| ----------------------------
any                  | any          | any    | any         | [docs](/docs/getting-started-guides/scratch/)                                |          | Community ([@erictune](https://github.com/erictune))
-->
IaaS プロバイダ        | 設定管理 | OS     | ネットワーク機能  | ドキュメント                                              | 確認 | サポート・レベル
-------------------- | ------------ | ------ | ----------  | ---------------------------------------------     | ---------| ----------------------------
any                  | any          | any    | any         | [docs](/jp/docs/getting-started-guides/scratch/)                                |          | コミュニティ ([@erictune](https://github.com/erictune))


<!--
For support level information on all solutions, see the [Table of solutions](/docs/getting-started-guides/#table-of-solutions/) chart.
-->
ソリューションすべてに対するサポート・レベル情報は [Table of solutions](/docs/getting-started-guides/#table-of-solutions/) にある表をご覧ください。

