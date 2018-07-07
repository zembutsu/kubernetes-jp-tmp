---
reviewers:
- sig-cluster-lifecycle
title: kubeadm でマスタ１台のクラスタを構築
content_template: templates/task
weight: 70
---

{{% capture overview %}}

<!--
<img src="https://raw.githubusercontent.com/cncf/artwork/master/kubernetes/certified-kubernetes/versionless/color/certified-kubernetes-color.png" align="right" width="150px">**kubeadm** helps you bootstrap a minimum viable Kubernetes cluster that conforms to best practices.  With kubeadm, your cluster should pass [Kubernetes Conformance tests](https://kubernetes.io/blog/2017/10/software-conformance-certification). 
-->

<img src="https://raw.githubusercontent.com/cncf/artwork/master/kubernetes/certified-kubernetes/versionless/color/certified-kubernetes-color.png" align="right" width="150px">**kubeadm** はベスト・プラクティスに従う最小環境の Kubernetes クラスタ構築に役立ちます。kubeadm とあわせ、クラスタの環境は [kubernetes適合試験](https://kubernetes.io/blog/2017/10/software-conformance-certification) に合格しているべきです。また、kubeadm は他にもクラスタのライフサイクル機能も支援します。たとえばクラスタのアップグレードやダウングレード、 [ブートストラップ・トークン(Bootstrap Tokens)](/jp/docs/admin/bootstrap-tokens/) の管理です。

<!--
Because you can install kubeadm on various types of machine (e.g. laptop, server, 
Raspberry Pi, etc.), it's well suited for integration with provisioning systems 
such as Terraform or Ansible.
-->

kubeadm を様々なマシン（例：ノートPC、サーバ、Raspberry Pi、等）にインストールできますので、Terraform や Ansible のようなプロビジョニング・システムとの同時利用も適います。

<!--
kubeadm's simplicity means it can serve a wide range of use cases:
-->

kubeadm は広範囲にわたる利用場面を簡単にします。

<!--
- New users can start with kubeadm to try Kubernetes out for the first time.
- Users familiar with Kubernetes can spin up clusters with kubeadm and test their applications.
- Larger projects can include kubeadm as a building block in a more complex system that can also include other installer tools.
-->

- Kubernetes を始めて使おうとする新規利用者は、kubeadm で始められる。
- Kubernetes に慣れている利用者は、kubeadm で素早くクラスタを準備し、アプリケーションをテストできる。
- より複雑なシステムの構築範囲で、他のインストーラ・ツールを含むプロジェクトでも、kubeadm を導入できる。

<!--
kubeadm is designed to be a simple way for new users to start trying
Kubernetes out, possibly for the first time, a way for existing users to
test their application on and stitch together a cluster easily, and also to be
a building block in other ecosystem and/or installer tool with a larger
scope.
-->

新規利用者が Kubernetes を試すにあたり、初めから可能な限り最も簡単な手法となるように Kubeadm は設計されています。それだけでなく、既存利用者はアプリケーションをテストし、クラスタを簡単に組み上げ、広範囲に亘るインストール用ツールや他とのエコシステムを築き上げるためです。

<!--
You can install _kubeadm_ very easily on operating systems that support
installing deb or rpm packages. The responsible SIG for kubeadm,
[SIG Cluster Lifecycle](https://github.com/kubernetes/community/tree/master/sig-cluster-lifecycle), provides these packages pre-built for you,
but you may also on other OSes.
-->

 _kubeadm_ をオペレーティングシステムにインストールするのは非常に簡単です。そのために deb あるいは rpm パッケージのインストール緒wサポートしています。kubeadm 用 SIG の対応としては、[SIG クラスタ・ライフサイクル](https://github.com/kubernetes/community/tree/master/sig-cluster-lifecycle) では構築済みのパッケージが提供されているだけでなく、その他の OS 向けにも提供されています。

### kubeadm 完成度

| 領域                      | 成熟度 |
|---------------------------|--------------- |
| コマンドライン UX           | beta           |
| 実装            | beta           |
| 設定ファイル API           | alpha          |
| セルフ・ホスティング              | alpha          |
| kubeadm alpha コマンド | alpha          |
| CoreDNS                   | GA          |
| DynamicKubeletConfig      | alpha          |

<!--
kubeadm's overall feature state is **Beta** and will soon be graduated to
**General Availability (GA)** during 2018. Some sub-features, like self-hosting
or the configuration file API are still under active development. The
implementation of creating the cluster may change slightly as the tool evolves,
but the overall implementation should be pretty stable. Any commands under
`kubeadm alpha` are by definition, supported on an alpha level.
-->

kubeadm 機能の大半は **Beta** ですが、間もなく2018 年内には **一般利用可能 (GA)** に到達予定です。機能にはセルフ・ホスティングや設定ファイル API といった複数のサブ機能を含んでおり、活発に開発が進められています。クラスタ構築の実装は、ツールの進化によって若干の変化があるかもしれません。しかし、大部分の実装はおおよそ安定しています。`kubeadm alpha` 配下で定義されているコマンドは、アルファ・レベルのサポートです。

<!--
### Support timeframes
-->
### サポート期間

<!--
Kubernetes releases are generally supported for nine months, and during that
period a patch release may be issued from the release branch if a severe bug or
security issue is found. Here are the latest Kubernetes releases and the support
timeframe; which also applies to `kubeadm`.
-->

Kubernetes のリリースは通常９ヶ月間サポートされており、何らかのバグやセキュリティ問題が発見されれば、リリース・ブランチから分岐して、定期的にパッチを提供します。以下は Kubernetes の最新リリースとサポート期間であり、 `kubeadm` にも適用されます。

<!--
| Kubernetes version | Release month  | End-of-life-month |
|--------------------|----------------|-------------------|
| v1.6.x             | March 2017     | December 2017     |
| v1.7.x             | June 2017      | March 2018        |
| v1.8.x             | September 2017 | June 2018         |
| v1.9.x             | December 2017  | September 2018    |
| v1.10.x            | March 2018     | December 2018     |
| v1.11.x            | June 2018      | March 2019        |
-->

| Kubernetes バージョン | リリース月  | 提供終了(End-of-life-month) |
|--------------------|----------------|-------------------|
| v1.6.x             | 2017年3月     | 2017年12月     |
| v1.7.x             | 2017年6月      | 2018年3月        |
| v1.8.x             | 2017年9月 | 2018年6月         |
| v1.9.x             | 2017年12月  | 2018年9月    |
| v1.10.x            | 2018年3月     | 2018年12月     |
| v1.11.x            | 2018年6月      | 2019年3月        |

{{% /capture %}}

{{% capture prerequisites %}}

<!--
- One or more machines running a deb/rpm-compatible OS, for example Ubuntu or CentOS
- 2 GB or more of RAM per machine. Any less leaves little room for your
   apps.
- 2 CPUs or more on the master
- Full network connectivity among all machines in the cluster. A public or
   private network is fine.
 -->
 
- dev/rpm 互換 OS を実行可能な１つまたは複数のマシン。例： Ubuntu または CentOS 。
- マシンごとに 2 GB 以上のメモリ（動作アプリケーションが小さければ少なくても可能）
- マスタは 2 CPU 以上
- クラスタ内の全てのマシン間で完全なネットワーク接続性がある（パブリックまたはプライベート・ネットワークでも）
 
{{% /capture %}}

{{% capture steps %}}

<!--
## Objectives
-->
## 目標

<!--
* Install a single master Kubernetes cluster or [high availability cluster](https://kubernetes.io/docs/setup/independent/high-availability/)
* Install a Pod network on the cluster so that your Pods can
  talk to each other
-->

* 単一マスタ Kubernetes クラスタか、 [高可用性クラスタ](/jp/docs/setup/independent/high-availability/) をインストール
* クラスタ上に Pod ネットワークをインストールするので、Pod 間で相互通信できる

<!--
## Instructions
-->

## 手順

<!--
### Installing kubeadm on your hosts
-->

### ホスト上に kubeadm をインストール

<!--
See ["Installing kubeadm"](/docs/setup/independent/install-kubeadm/).
-->

[kubeadm のインストール](/jp/docs/tasks/tools/install-kubeadm/) をご覧ください。

<!--
{{< note >}}
**Note:** If you have already installed kubeadm, run `apt-get update &&
apt-get upgrade` or `yum update` to get the latest version of kubeadm.

When you upgrade, the kubelet restarts every few seconds as it waits in a crashloop for
kubeadm to tell it what to do. This crashloop is expected and normal. 
After you initialize your master, the kubelet runs normally.
{{< /note >}}
-->

{{< note >}}
**メモ:** 既に kubeadm をインストールしている場合は、`apt-get update && apt-get upgrade` か `yum update` を実行し、kubeadm の最新版を入手します。

kubelet は数秒ごとに再起動するため、kubeadm が次に何をすべきか伝えるためクラッシュループ (crashloop) で待機します。クラッシュループは予測可能であり、正常ですから、次のステップに進めば、kubelet は通常通り動作します。
{{< /note >}}

<!--
### Initializing your master
-->
### マスタの初期化

<!--
The master is the machine where the control plane components run, including
etcd (the cluster database) and the API server (which the kubectl CLI
communicates with).
-->

マスタとはコントロール・プレーン・コンポーネントを実行するためのマシンであり、 etcd（クラスタのデータベース）と API サーバ（kubectl CLI との通信）を含みます。

<!--
1. Choose a pod network add-on, and verify whether it requires any arguments to 
be passed to kubeadm initialization. Depending on which
third-party provider you choose, you might need to set the `--pod-network-cidr` to
a provider-specific value. See [Installing a pod network add-on](#pod-network).
1. (Optional) Unless otherwise specified, kubeadm uses the network interface associated 
with the default gateway to advertise the master's IP. To use a different 
network interface, specify the `--apiserver-advertise-address=<ip-address>` argument 
to `kubeadm init`. To deploy an IPv6 Kubernetes cluster using IPv6 addressing, you 
must specify an IPv6 address, for example `--apiserver-advertise-address=fd00::101`
1. (Optional) Run `kubeadm config images pull` prior to `kubeadm init` to verify 
connectivity to gcr.io registries.   
-->

1. ポッド・ネットワーク・アドオンを選択し、kubeadm 初期化に必要な引数があるかどうかを確認します。選択したサードパーティ・プロバイダに依存しますが、プロバイダ特有の値を `--pod-network-cidr` での指定が必要な場合があります。詳細は [ポッド・ネットワーク・アドオンのインストール](#pod-network)をご覧ください。
1. （オプション）特に指定がなければ、kubeadm はマスタの IP を広報（advertise）するために、デフォルト・ゲートウェイに関連付けられたネットワーク・インターフェースを使います。別のネットワーク・インターフェースを使うには、 `kubeadm init` の引数で `--apiserver-advertise-address=<ip-address>` を指定します。IPv6 アドレスを割り当てて IPv6 Kubernetes クラスタを展開するには、  `--apiserver-advertise-address=fd00::101` のように、必ず IPv6 アドレスを指定します。
1. （オプション）gcr.io レジストリに対する接続性を確認するには、 `kubeadm init` よりも前に `kubeadm config images pull`を実行します。

<!--
Now run:
-->
それから、実行します。


```bash
kubeadm init <引数> 
```

<!--
### More information
-->
### 詳細情報

<!--
For more information about `kubeadm init` arguments, see the [kubeadm reference guide](/docs/reference/setup-tools/kubeadm/kubeadm/).
-->
`kubeadm init` で指定可能なフラグの詳細を知りたい場合は、 [kubeadm 参照ガイド] (/jp/docs/reference/setup-tools/kubeadm/kubeadm/) をご覧ください。

<!--
For a complete list of configuration options, see the [configuration file documentation](/docs/reference/setup-tools/kubeadm/kubeadm-init/#config-file).
-->
設定オプションの完全な一覧は、 [設定ファイル資料](/jp/docs/reference/setup-tools/kubeadm/kubeadm-init/#config-file) をご覧ください。

<!--
To customize control plane components, including optional IPv6 assignment to liveness probe for control plane components and etcd server, provide extra arguments to each component as documented in [custom arguments](/docs/admin/kubeadm#custom-args).
-->
コントロール・プレーンに対してカスタマイズをする場合、たとえばオプションの IPv6 アドレス割り当て、コントロール・プレーン構成要素や ctcd サーバに対する生存確認に対しては、[カスタム引数](/jp/docs/admin/kubeadm#custom-args) に記載されている追加引数の指定が必要です。

<!--
To run `kubeadm init` again, you must first [tear down the cluster](#tear-down).
-->
`kubeadm init`を実行するには、最初に [クラスタをディア・ダウン（tear down）]($tear-down) します。

<!--
If you join a node with a different architecture to your cluster, create a separate
Deployment or DaemonSet for `kube-proxy` and `kube-dns` on the node. This is because the Docker images for these
components do not currently support multi-architecture.
-->
クラスタに対して異なったアーキテクチャを持つノードを追加する場合は、分けて展開（Deployment）するか、ノード上に `kube-proxy` と `kube-dns` に対する別のデーモンセット（DaemonSet）が必要です。これは、各構成要素用の Docker イメージが、マルチ・アーキテクチャに現時点では対応していないからです。

<!--
`kubeadm init` first runs a series of prechecks to ensure that the machine
is ready to run Kubernetes. These prechecks expose warnings and exit on errors. `kubeadm init`
then downloads and installs the cluster control plane components. This may take several minutes. 
The output should look like:
-->
`kubeadm init` の初回実行時、マシンで Kubernetes を稼働する準備が整っているかどうか、一連の事前確認を行います。この事前確認により、警告の表示や、 エラーによる停止が起こるかもしれません。 `kubeadm init` の後、クラスタのコントロール・プレーン構成要素のダウンロードとインストールをします。これには数分かかるでしょう。出力は次のようになります。


```none
[init] Using Kubernetes version: vX.Y.Z
[preflight] Running pre-flight checks
[kubeadm] WARNING: starting in 1.8, tokens expire after 24 hours by default (if you require a non-expiring token use --token-ttl 0)
[certificates] Generated ca certificate and key.
[certificates] Generated apiserver certificate and key.
[certificates] apiserver serving cert is signed for DNS names [kubeadm-master kubernetes kubernetes.default kubernetes.default.svc kubernetes.default.svc.cluster.local] and IPs [10.96.0.1 10.138.0.4]
[certificates] Generated apiserver-kubelet-client certificate and key.
[certificates] Generated sa key and public key.
[certificates] Generated front-proxy-ca certificate and key.
[certificates] Generated front-proxy-client certificate and key.
[certificates] Valid certificates and keys now exist in "/etc/kubernetes/pki"
[kubeconfig] Wrote KubeConfig file to disk: "admin.conf"
[kubeconfig] Wrote KubeConfig file to disk: "kubelet.conf"
[kubeconfig] Wrote KubeConfig file to disk: "controller-manager.conf"
[kubeconfig] Wrote KubeConfig file to disk: "scheduler.conf"
[controlplane] Wrote Static Pod manifest for component kube-apiserver to "/etc/kubernetes/manifests/kube-apiserver.yaml"
[controlplane] Wrote Static Pod manifest for component kube-controller-manager to "/etc/kubernetes/manifests/kube-controller-manager.yaml"
[controlplane] Wrote Static Pod manifest for component kube-scheduler to "/etc/kubernetes/manifests/kube-scheduler.yaml"
[etcd] Wrote Static Pod manifest for a local etcd instance to "/etc/kubernetes/manifests/etcd.yaml"
[init] Waiting for the kubelet to boot up the control plane as Static Pods from directory "/etc/kubernetes/manifests"
[init] This often takes around a minute; or longer if the control plane images have to be pulled.
[apiclient] All control plane components are healthy after 39.511972 seconds
[uploadconfig] Storing the configuration used in ConfigMap "kubeadm-config" in the "kube-system" Namespace
[markmaster] Will mark node master as master by adding a label and a taint
[markmaster] Master master tainted and labelled with key/value: node-role.kubernetes.io/master=""
[bootstraptoken] Using token: <token>
[bootstraptoken] Configured RBAC rules to allow Node Bootstrap tokens to post CSRs in order for nodes to get long term certificate credentials
[bootstraptoken] Configured RBAC rules to allow the csrapprover controller automatically approve CSRs from a Node Bootstrap Token
[bootstraptoken] Creating the "cluster-info" ConfigMap in the "kube-public" namespace
[addons] Applied essential addon: CoreDNS
[addons] Applied essential addon: kube-proxy

Your Kubernetes master has initialized successfully!

To start using your cluster, you need to run (as a regular user):

  mkdir -p $HOME/.kube
  sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
  sudo chown $(id -u):$(id -g) $HOME/.kube/config

You should now deploy a pod network to the cluster.
Run "kubectl apply -f [podnetwork].yaml" with one of the addon options listed at:
  http://kubernetes.io/docs/admin/addons/

You can now join any number of machines by running the following on each node
as root:

  kubeadm join --token <token> <master-ip>:<master-port> --discovery-token-ca-cert-hash sha256:<hash>
```

<!--
To make kubectl work for your non-root user, run these commands, which are
also part of the `kubeadm init` output:
-->
root ではないユーザでも kubectl を動作するには、以下のコマンドを実行します。こちらは `kubeadm init` の出力結果にも表示されます。


```bash
mkdir -p $HOME/.kube
sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config
```

```
Alternatively, if you are the `root` user, you can run:
```
あるいは、 `root` ユーザの場合は次のように実行できます。

```bash
export KUBECONFIG=/etc/kubernetes/admin.conf
```

<!--
Make a record of the `kubeadm join` command that `kubeadm init` outputs. You
need this command to [join nodes to your cluster](#join-nodes).
-->
`kubeadm init` で出力される `kubeadm join`コマンド`を記録します。[ノードをクラスタに参加](#join-nodes) するために必要です。

<!--
The token is used for mutual authentication between the master and the joining
nodes.  The token included here is secret. Keep it safe, because anyone with this
token can add authenticated nodes to your cluster. These tokens can be listed,
created, and deleted with the `kubeadm token` command. See the
[kubeadm reference guide](/docs/reference/setup-tools/kubeadm/kubeadm-token/).
-->
トークンはマスタと参加ノード間が相互認証のために使います。トークンにはシークレットも含みます。このトークンがあれば、誰でもクラスタに認証されたノードとしてつかが可能となります。そのため、トークンは安全な場所に保管してください。トークンの一覧表示、作成、削除は `kubeadm token` コマンドで行います。詳しい情報は [リファレンス・ガイド](/jp/docs/reference/setup-tools/kubeadm/kubeadm-token/) をご覧ください。

<!--
### Installing a pod network add-on {#pod-network}
-->

### ポッド・ネットワーク・アドオンのインストール {#pod-network}

<!--
{{< caution >}}
**Caution:** This section contains important information about installation and deployment order. Read it carefully before proceeding.
{{< /caution >}}
-->
{{< caution >}}
**注意:** インストールと展開に関する重要な情報がこのセクションにあります。作業前に注意してお読みください。
{{< /caution >}}

<!--
You must install a pod network add-on so that your pods can communicate with
each other.
-->
ポットがお互いに通信できるようにするためには ポッド・ネットワーク・アドオンのインストールが必須です。

<!--
**The network must be deployed before any applications. Also, CoreDNS will not start up before a network is installed.
kubeadm only supports Container Network Interface (CNI) based networks (and does not support kubenet).**
-->
**アプリケーションより先に、ネットワークの展開が必要です。また、ネットワークのインストール前に CoreDNS をセットアップできません。kubeadm がサポートしているのは、コンテナ・ネットワーク・インターフェース（CNI）をベースとしたネットワークのみです（そして、kubenet はサポートしません）。**

<!--
Several projects provide Kubernetes pod networks using CNI, some of which also
support [Network Policy](/docs/concepts/services-networking/networkpolicies/). See the [add-ons page](/docs/concepts/cluster-administration/addons/) for a complete list of available network add-ons. 
- IPv6 support was added in [CNI v0.6.0](https://github.com/containernetworking/cni/releases/tag/v0.6.0). 
- [CNI bridge](https://github.com/containernetworking/plugins/blob/master/plugins/main/bridge/README.md) and [local-ipam](https://github.com/containernetworking/plugins/blob/master/plugins/ipam/host-local/README.md) are the only supported IPv6 network plugins in Kubernetes version 1.9.
-->
いくつかのプロジェクトでは Kubernetes ポッド・ネットワークに CNI が使われたものを提供しますが、またいくつかは [ネットワーク・ポリシー](/jp/docs/concepts/services-networking/networkpolicies/) もサポートします。[アドオン・ページ](/jp/docs/concepts/cluster-administration/addons/) をご覧いただくと、利用可能なネットワーク・アドオンの完全な一覧をご覧いただけます。IPv6 サポートは [CNI v0.6.0](https://github.com/containernetworking/cni/releases/tag/v0.6.0)　で追加されました。 [CNI bridge](https://github.com/containernetworking/plugins/blob/master/plugins/main/bridge/README.md) と [local-ipam](https://github.com/containernetworking/plugins/blob/master/plugins/ipam/host-local/README.md) がサポートされているのは、1.9 における IPv6 ネットワーク・プラグインです。

<!--
Note that kubeadm sets up a more secure cluster by default and enforces use of [RBAC](/docs/reference/access-authn-authz/rbac/).
Make sure that your network manifest supports RBAC.
-->
kubeadm はより安全なクラスタをセットアップするため、標準で [RBAC](/docs/reference/access-authn-authz/rbac/) の利用を強制します。ネットワーク・マニフェストでは、 RBAC のサポートを選択しているかどうかを確認してください。

<!--
You can install a pod network add-on with the following command:
-->
ポッド・ネットワークのアドオンは、以下のコマンドでインストールできます:

```bash
kubectl apply -f <add-on.yaml>
```

<!--
You can install only one pod network per cluster.
-->
クラスタごとに１つのポッド・ネットワークをインストールできます。

{{< tabs name="tabs-pod-install" >}}
{{% tab name="１つを選択..." %}}
対象となるサードパーティ製ポッド・ネットワークのインストール手順を見るには、タブをクリックします。
{{% /tab %}}

{{% tab name="Calico" %}}
<!--
For more information about using Calico, see [Quickstart for Calico on Kubernetes](https://docs.projectcalico.org/latest/getting-started/kubernetes/), [Installing Calico for policy and networking](https://docs.projectcalico.org/latest/getting-started/kubernetes/installation/calico), and other related resources.
-->
Calico の詳細は、 [Quickstart for Calico on Kubernetes](https://docs.projectcalico.org/latest/getting-started/kubernetes/)、 [Installing Calico for policy and networking](https://docs.projectcalico.org/latest/getting-started/kubernetes/installation/calico)、そして関連リソースをご覧ください。

<!--
In order for Network Policy to work correctly, you need to pass `--pod-network-cidr=192.168.0.0/16` to `kubeadm init`. Note that Calico works on `amd64` only.
-->
ネットワーク・ポリシーを正常に動かすためには、`kubeadm init` で `--pod-network-cidr=192.168.0.0/16` を 指定する必要があります。Calico が動作するのは `amd64` のみです。

```shell
kubectl apply -f https://docs.projectcalico.org/v3.1/getting-started/kubernetes/installation/hosted/rbac-kdd.yaml
kubectl apply -f https://docs.projectcalico.org/v3.1/getting-started/kubernetes/installation/hosted/kubernetes-datastore/calico-networking/1.7/calico.yaml
```

{{% /tab %}}
{{% tab name="Canal" %}}
<!--
Canal uses Calico for policy and Flannel for networking. Refer to the Calico documentation for the [official getting started guide](https://docs.projectcalico.org/latest/getting-started/kubernetes/installation/flannel).
-->
Canal はポリシーのために Calico を、ネットワーク機能のために Flannel を使います。Calico ドキュメントを参照するには、 [official getting started guide](https://docs.projectcalico.org/latest/getting-started/kubernetes/installation/flannel) を後rナンください。

<!--
For Canal to work correctly, `--pod-network-cidr=10.244.0.0/16` has to be passed to `kubeadm init`. Note that Canal works on `amd64` only.
-->
Canal を正常に動かすためには、`kubeadm init` で `--pod-network-cidr=10.244.0.0/16` を指定する必要があります。Canal が動作するのは `amd64` のみです。


```shell
kubectl apply -f https://docs.projectcalico.org/v3.1/getting-started/kubernetes/installation/hosted/canal/rbac.yaml
kubectl apply -f https://docs.projectcalico.org/v3.1/getting-started/kubernetes/installation/hosted/canal/canal.yaml
```

{{% /tab %}}
{{% tab name="Flannel" %}}
<!--
For `flannel` to work correctly, `--pod-network-cidr=10.244.0.0/16` has to be passed to `kubeadm init`. Note that `flannel` works on `amd64`, `arm`, `arm64` and `ppc64le`. For it to work on a platform other than
`amd64`, you must manually download the manifest and replace `amd64` occurrences with your chosen platform.
-->
`flannel` を正常に動かすためには `kubeadm init` で `--pod-network-cidr=10.244.0.0/16` を指定する必要があります。`flannel` が動作するのは `amd64`、 `arm`、 `arm64`、`ppc64le` ですが、`amd64` 以外のプラットフォームで動作する場合は、マニフェストを手動でダウンロードし、 `amd64` の部分を適切なプラットフォームに書き換える必要があります。

<!--
Set `/proc/sys/net/bridge/bridge-nf-call-iptables` to `1` by running `sysctl net.bridge.bridge-nf-call-iptables=1`
to pass bridged IPv4 traffic to iptables' chains. This is a requirement for some CNI plugins to work, for more information
please see [here](https://kubernetes.io/docs/concepts/cluster-administration/network-plugins/#network-plugin-requirements).
-->
実行時に `/proc/sys/net/bridge/bridge-nf-call-iptables` を `1` にセットすると、 `sysctl net.bridge.bridge-nf-call-iptables=1` によって IPv4 トラフィックを iptables のチェーンにブリッジします。これはいくつかの CNI プラグインを実行するために必要であり、より詳しい情報は [こちら](/jp/docs/concepts/cluster-administration/network-plugins/#network-plugin-requirements) をご覧ください。


```shell
kubectl apply -f https://raw.githubusercontent.com/coreos/flannel/v0.10.0/Documentation/kube-flannel.yml
```
<!--
For more information about `flannel`, see [the CoreOS flannel repository on GitHub
](https://github.com/coreos/flannel).
-->
`flannel` に関するより詳しい情報は、[こちら](https://github.com/coreos/flannel) をご覧ください。

{{% /tab %}}

{{% tab name="Kube-router" %}}
<!--
Set `/proc/sys/net/bridge/bridge-nf-call-iptables` to `1` by running `sysctl net.bridge.bridge-nf-call-iptables=1`
to pass bridged IPv4 traffic to iptables' chains. This is a requirement for some CNI plugins to work, for more information
please see [here](https://kubernetes.io/docs/concepts/cluster-administration/network-plugins/#network-plugin-requirements).
-->
実行時に `/proc/sys/net/bridge/bridge-nf-call-iptables` を `1` にセットすると、 `sysctl net.bridge.bridge-nf-call-iptables=1` によって IPv4 トラフィックを iptables のチェーンにブリッジします。これはいくつかの CNI プラグインを実行するために必要であり、より詳しい情報は [こちら](/jp/docs/concepts/cluster-administration/network-plugins/#network-plugin-requirements) をご覧ください。

<!--
Kube-router relies on kube-controller-manager to allocate pod CIDR for the nodes. Therefore, use `kubeadm init` with the `--pod-network-cidr` flag.
-->
Kube-router がノードに対してポッド CIDR を割り当てるには、 kube-controller-manager に依存します。そのため、 `kubeadm init` で `--pod-network-cidr` フラグの指定が必要です。

<!--
Kube-router provides pod networking, network policy, and high-performing IP Virtual Server(IPVS)/Linux Virtual Server(LVS) based service proxy.
-->
Kube-router が提供するのはポッド・ネットワーク機能、ネットワーク・ポリシー、サービス・プロキシをベースと下高性能 IP 仮想サーバ（IPVS）/Linux 仮想サーバ(LVS)です。

<!--
For information on setting up Kubernetes cluster with Kube-router using kubeadm, please see official [setup guide](https://github.com/cloudnativelabs/kube-router/blob/master/docs/kubeadm.md).
-->
kubeadm で kube-router に対応した Kubernetes クラスタをセットアップするための詳細については、公式の [セットアップ・ガイド](https://github.com/cloudnativelabs/kube-router/blob/master/docs/kubeadm.md) をご覧ください。

{{% /tab %}}

{{% tab name="Romana" %}}
<!--
Set `/proc/sys/net/bridge/bridge-nf-call-iptables` to `1` by running `sysctl net.bridge.bridge-nf-call-iptables=1`
to pass bridged IPv4 traffic to iptables' chains. This is a requirement for some CNI plugins to work, for more information
please see [here](https://kubernetes.io/docs/concepts/cluster-administration/network-plugins/#network-plugin-requirements).
-->
実行時に `/proc/sys/net/bridge/bridge-nf-call-iptables` を `1` にセットすると、 `sysctl net.bridge.bridge-nf-call-iptables=1` によって IPv4 トラフィックを iptables のチェーンにブリッジします。これはいくつかの CNI プラグインを実行するために必要であり、より詳しい情報は  [こちら](/jp/docs/concepts/cluster-administration/network-plugins/#network-plugin-requirements) をご覧ください。

<!--
The official Romana set-up guide is [here](https://github.com/romana/romana/tree/master/containerize#using-kubeadm).
-->
公式の Romana セットアップ・ガイドは [こちら](https://github.com/romana/romana/tree/master/containerize#using-kubeadm) です。

<!--
Romana works on `amd64` only.
-->
Romana が動作するのは `amd64` のみです

```shell
kubectl apply -f https://raw.githubusercontent.com/romana/romana/master/containerize/specs/romana-kubeadm.yml
```
{{% /tab %}}

{{% tab name="Weave Net" %}}
<!--
Set `/proc/sys/net/bridge/bridge-nf-call-iptables` to `1` by running `sysctl net.bridge.bridge-nf-call-iptables=1`
to pass bridged IPv4 traffic to iptables' chains. This is a requirement for some CNI plugins to work, for more information
please see [here](https://kubernetes.io/docs/concepts/cluster-administration/network-plugins/#network-plugin-requirements).
-->
実行時に `/proc/sys/net/bridge/bridge-nf-call-iptables` を `1` にセットすると、 `sysctl net.bridge.bridge-nf-call-iptables=1` によって IPv4 トラフィックを iptables のチェーンにブリッジします。これはいくつかの CNI プラグインを実行するために必要であり、より詳しい情報は  [こちら](/jp/docs/concepts/cluster-administration/network-plugins/#network-plugin-requirements) をご覧ください。

<!--
The official Weave Net set-up guide is [here](https://www.weave.works/docs/net/latest/kube-addon/).
-->
公式の Weave Net セットアップ・ガイドは [こちら](https://www.weave.works/docs/net/latest/kube-addon/) です。

<!--
Weave Net works on `amd64`, `arm`, `arm64` and `ppc64le` without any extra action required.
Weave Net sets hairpin mode by default. This allows Pods to access themselves via their Service IP address
if they don't know their PodIP.
-->
Weave Net は追加設定を必要とせず  `amd64`、 `arm`、 `arm64`、 `ppc64le` で動作します。Weave Net はデフォルトではヘアピン・モード(hairpin mode)です。これにより、Pod は PodIP を知らなくても自身のサービス IP アドレスを経由してアクセスできます。

```shell
kubectl apply -f "https://cloud.weave.works/k8s/net?k8s-version=$(kubectl version | base64 | tr -d '\n')"
```
{{% /tab %}}
{{< /tabs >}}

<!--
Once a pod network has been installed, you can confirm that it is working by
checking that the CoreDNS pod is Running in the output of `kubectl get pods --all-namespaces`.
And once the CoreDNS pod is up and running, you can continue by joining your nodes.
-->
ポッド・ネットワークのインストール後、CoreDNS ポッドが正常に動作しているかどうかの確認は、 `kubectl get pods --all-namespaces` で行えます。CoreDNS ポッドが起動・実行中になれば、続けてノードを追加可能になります。

<!--
If your network is not working or CoreDNS is not in the Running state, check
out our [troubleshooting docs](/docs/setup/independent/troubleshooting-kubeadm/).
-->
ネットワークが稼働していないか CoreDNS が実行中の状態でなければ、 [トラブルシューティング・ドキュメント](/jp/docs/setup/independent/troubleshooting-kubeadm/)をご覧ください。

<!--
### Master Isolation
-->
### マスタの分離（isolation）

<!--
By default, your cluster will not schedule pods on the master for security
reasons. If you want to be able to schedule pods on the master, e.g. for a
single-machine Kubernetes cluster for development, run:
-->
デフォルトでは、安全上の理由により、クラスタはマスタ上にポッドをスケジュールしません。もしもマスタ上にポッドをスケジュールできるようにしたい場合、例えば、１台のマシンで 開発用 Kubernetes クラスタを動作させたい場合は、次のように実行します:

```bash
kubectl taint nodes --all node-role.kubernetes.io/master-
```
<!--
With output looking something like:
-->
実行結果は次のようになります。

```
node "test-01" untainted
taint key="dedicated" and effect="" not found.
taint key="dedicated" and effect="" not found.
```

<!--
This will remove the `node-role.kubernetes.io/master` taint from any nodes that
have it, including the master node, meaning that the scheduler will then be able
to schedule pods everywhere.
-->
これはマスタ・ノードを含むあらゆるノードから `node-role.kubernetes.io/master` テイントを削除するもので、これにより、スケジューラはポッドをどこでもスケジュール可能にします。

<!--
### Joining your nodes {#join-nodes}
-->
### ノードの追加 {#join-nodes}

<!--
The nodes are where your workloads (containers and pods, etc) run. To add new nodes to your cluster do the following for each machine:
-->
ノードとはワークロード（コンテナ、ポッド等）を実行する場所です。クラスタの各マシンで新しいノードを追加するには、次のようにします:

<!--
* SSH to the machine
* Become root (e.g. `sudo su -`)
* Run the command that was output by `kubeadm init`. For example:
-->
* マシンに SSH
* root になる (例: `sudo su -`)
* `kubeadm init` の出力結果のコマンドを実行。実行例:

``` bash
kubeadm join --token <token> <master-ip>:<master-port> --discovery-token-ca-cert-hash sha256:<hash>
```

{{< note >}}
<!--
**Note:** To specify an IPv6 tuple for `<master-ip>:<master-port>`, IPv6 address must be enclosed in square brackets, for example: `[fd00::101]:2073`.
-->
**メモ:** IPv6 タプルを `<master-ip>:<master-port>` で指定するには、 `[fd00::101]:2073` のように IPv6 アドレスを角括弧で囲む必要があります。
{{< /note >}}

<!--
The output should look something like:
-->
出力結果は次のようなものです。

```
[preflight] Running pre-flight checks

... (log output of join workflow) ...

Node join complete:
* Certificate signing request sent to master and response
  received.
* Kubelet informed of new secure connection details.

Run 'kubectl get nodes' on the master to see this machine join.
```

<!--
A few seconds later, you should notice this node in the output from `kubectl get
nodes` when run on the master.
-->
数秒の後、マスタ上で `kubectl get nodes` を実行すると、出力結果から対象ノードを確認できるでしょう。

<!--
### (Optional) Controlling your cluster from machines other than the master
-->
### (オプション) マスタ以外のマシンからクラスタを制御

<!--
In order to get a kubectl on some other computer (e.g. laptop) to talk to your
cluster, you need to copy the administrator kubeconfig file from your master
to your workstation like this:
-->
kubectl で他のコンピュータ（例: ノートPC）からクラスタと通信可能にするには、以下のように、管理用 kubeconfig ファイルをマスタから対象マシンに対してコピーする必要があります。

``` bash
scp root@<master ip>:/etc/kubernetes/admin.conf .
kubectl --kubeconfig ./admin.conf get nodes
```

{{< note >}}
<!--
**Note:** The example above assumes SSH access is enabled for root. If that is not the
case, you can copy the `admin.conf` file to be accessible by some other user
and `scp` using that other user instead.
-->
**メモ:** この例は root での SSH 接続が可能な場合です。そうでない場合は、 root に代り`admin.conf` を接続可能な他のユーザでコピーと `scp` を用います。

<!--
The `admin.conf` file gives the user _superuser_ privileges over the cluster.
This file should be used sparingly. For normal users, it's recommended to
generate an unique credential to which you whitelist privileges. You can do
this with the `kubeadm alpha phase kubeconfig user --client-name <CN>`
command. That command will print out a KubeConfig file to STDOUT which you
should save to a file and distribute to your user. After that, whitelist
privileges by using `kubectl create (cluster)rolebinding`.
-->
`admin.conf` ファイルはユーザに対し、クラスタを超えて _スーパーユーザ_ 特権をあたえます。このファイルは慎重に扱うべきです。通常のユーザでは、ホワイトリスト権限をあたえたユニークな認証情報（クレデンシャル）生成を推奨します。そのためには `kubeadm alpha phase kubeconfig user --client-name <CN>`コマンドを実行します。このコマンドによって KubeConfig ファイルの情報を標準出力し、出力結果をファイルに保存し、ユーザに与えられます。以降、ホワイトリスト権限は `kubectl create (cluster)rolebinding` を用います。
{{< /note >}}

<!--
### (Optional) Proxying API Server to localhost
-->
### (オプション) API サーバをローカルホストにプロキシ

<!--
If you want to connect to the API Server from outside the cluster you can use
`kubectl proxy`:
-->
クラスタ外から API サーバにアクセスしたい場合は、`kubectl proxy` が使えます。

```bash
scp root@<master ip>:/etc/kubernetes/admin.conf .
kubectl --kubeconfig ./admin.conf proxy
```
<!--
You can now access the API Server locally at `http://localhost:8001/api/v1`
-->
これでローカルから API サーバには `http://localhost:8001/api/v1` でアクセスできます。

<!--
## Tear down {#tear-down}
-->
## ティア・ダウン(Tear down) {#tear-down}

<!--
To undo what kubeadm did, you should first [drain the
node](/docs/reference/generated/kubectl/kubectl-commands#drain) and make
sure that the node is empty before shutting it down.
-->
kubeadm で行ったことを取り消したい場合、まず [ノードをドレーン（排出；drain）](/jp/docs/reference/generated/kubectl/kubectl-commands#drain) し、ノードを停止する前にノードが空なのを確認します。


<!--
Talking to the master with the appropriate credentials, run:
-->
マスタと適切な認証情報で通信し、実行します。

```bash
kubectl drain <node name> --delete-local-data --force --ignore-daemonsets
kubectl delete node <node name>
```

<!--
Then, on the node being removed, reset all kubeadm installed state:
-->
それから、削除するノード上で、kubeadm がインストールした状態をリセットします:

```bash
kubeadm reset
```

<!--
If you wish to start over simply run `kubeadm init` or `kubeadm join` with the
appropriate arguments.
-->
もしもやり直したい場合は、単純に `kubeadm init` や `kubeadm join` に適切な引数を付けるだけです。

<!--
More options and information about the
[`kubeadm reset command`](/docs/reference/setup-tools/kubeadm/kubeadm-reset/).
-->

詳しいオプションや追加情報については
[`kubeadm reset コマンド`](/jp/docs/reference/setup-tools/kubeadm/kubeadm-reset/) をご覧ください。

<!--
## Maintaining a cluster {#lifecycle}
-->
## クラスタの維持 {#lifecycle}

<!--
Instructions for maintaining kubeadm clusters (e.g. upgrades,downgrades, etc.) can be found [here.](/docs/tasks/administer-cluster/kubeadm)
-->
kubeadm クラスタを維持（メンテナンス）する手順（例：アップグレード、ダウングレード、等）は、[こちら](/jp/docs/tasks/administer-cluster/kubeadm) にあります。

<!--
## Explore other add-ons {#other-addons}
-->
## 他のアドオンを探す {#other-addons}

<!--
See the [list of add-ons](/docs/concepts/cluster-administration/addons/) to explore other add-ons,
including tools for logging, monitoring, network policy, visualization &amp;
control of your Kubernetes cluster.
-->

[アドオンの一覧](/docs/concepts/cluster-administration/addons/) ご覧いただくと、ログ記録、モニタリング、ネットワーク・ポリシー、可視化、Kubernetes クラスタの管理する他のアドオンを探せます。

<!--
## What's next {#whats-next}
-->
## 次は何をしますか {#whats-next}

<!--
* Verify that your cluster is running properly with [Sonobuoy](https://github.com/heptio/sonobuoy)
* Learn about kubeadm's advanced usage in the [kubeadm reference documentation](/docs/reference/setup-tools/kubeadm/kubeadm)
* Learn more about Kubernetes [concepts](/docs/concepts/) and [`kubectl`](/docs/user-guide/kubectl-overview/).
* Configure log rotation. You can use **logrotate** for that. When using Docker, you can specify log rotation options for Docker daemon, for example `--log-driver=json-file --log-opt=max-size=10m --log-opt=max-file=5`. See [Configure and troubleshoot the Docker daemon](https://docs.docker.com/engine/admin/) for more details.
-->
* [Sonobuoy](https://github.com/heptio/sonobuoy) でクラスタが正しく稼働しているか確認
* [kubeadm 参照ドキュメント](/jp/docs/reference/setup-tools/kubeadm/kubeadm) で kubeadm の高度な使い方を学ぶ
* Kubernetes [概念](/jp/docs/concepts/) and [`kubectl`](/jp/docs/user-guide/kubectl-overview/) について更に学ぶ
* **logrotate** でログ・ローテーション（入れ替え）を設定する。Docker の利用時は、Docker デーモンのオプションでログ・ローテーションを指定できます。例 `--log-driver=json-file --log-opt=max-size=10m --log-opt=max-file=5`。詳細は  [Configure and troubleshoot the Docker daemon](https://docs.docker.com/engine/admin/) をご覧ください。

<!--
## Feedback {#feedback}
-->
## フィードバック {#feedback}

<!--
* For bugs, visit [kubeadm Github issue tracker](https://github.com/kubernetes/kubeadm/issues)
* For support, visit kubeadm Slack Channel:
  [#kubeadm](https://kubernetes.slack.com/messages/kubeadm/)
* General SIG Cluster Lifecycle Development Slack Channel:
  [#sig-cluster-lifecycle](https://kubernetes.slack.com/messages/sig-cluster-lifecycle/)
* SIG Cluster Lifecycle [SIG information](#TODO)
* SIG Cluster Lifecycle Mailing List:
  [kubernetes-sig-cluster-lifecycle](https://groups.google.com/forum/#!forum/kubernetes-sig-cluster-lifecycle)
-->
* バグについては、[kubeadm Github issue tracker](https://github.com/kubernetes/kubeadm/issues) をご覧ください。
* サポートについては、kubeadm Slack Channel をご覧ください:
  [#kubeadm](https://kubernetes.slack.com/messages/kubeadm/)
* General SIG Cluster Lifecycle Development Slack Channel:
  [#sig-cluster-lifecycle](https://kubernetes.slack.com/messages/sig-cluster-lifecycle/)
* SIG Cluster Lifecycle [SIG information](#TODO)
* SIG Cluster Lifecycle Mailing List:
  [kubernetes-sig-cluster-lifecycle](https://groups.google.com/forum/#!forum/kubernetes-sig-cluster-lifecycle)

<!--
## Version skew policy {#version-skew-policy}
-->
## バージョンの差違に対するポリシー {#version-skew-policy}

<!--
The kubeadm CLI tool of version vX.Y may deploy clusters with a control plane of version vX.Y or vX.(Y-1).
kubeadm CLI vX.Y can also upgrade an existing kubeadm-created cluster of version vX.(Y-1).
-->
kubeadm CLI ツールのバージョン vX.Y は、コントロール・プレーンのバージョン vX.Y または vX.(Y-1) でも展開可能な場合があります。また、kubeadm CLI vX.Y は、既存の kubeadm バージョン vX.(Y-1)で作成した場合はアップロードできます。

<!--
Due to that we can't see into the future, kubeadm CLI vX.Y may or may not be able to deploy vX.(Y+1) clusters.
-->
将来的に期限切れで利用できなくなる kubeadm CLI vX.Y は、 vX.(Y+1) クラスタに展開できない場合があります。

<!--
Example: kubeadm v1.8 can deploy both v1.7 and v1.8 clusters and upgrade v1.7 kubeadm-created clusters to
v1.8.
-->
例: kubeadm v1.8 は v1.7 と v1.8 クラスタの両方にデプロイでき、v1.7 kubeadm で作成したクラスタは v1.8 にアップグレードできる。

<!--
Please also check our [installation guide](/docs/setup/independent/install-kubeadm/#installing-kubeadm-kubelet-and-kubectl)
for more information on the version skew between kubelets and the control plane.
-->
また、kubeletes と コントロール・プレーン間のバージョンずれに関しては [インストール・ガイド](/jp/docs/setup/independent/install-kubeadm/#installing-kubeadm-kubelet-and-kubectl) をご確認ください。

<!--
## kubeadm works on multiple platforms {#multi-platform}
-->
## 複数のプラットフォームで動作する kubeadm {#multi-platform}

<!--
kubeadm deb/rpm packages and binaries are built for amd64, arm (32-bit), arm64, ppc64le, and s390x
following the [multi-platform
proposal](https://github.com/kubernetes/community/blob/master/contributors/design-proposals/multi-platform.md).
-->
kubeadm の deb/rpm パッケージとバイナリを構築している対象は、amd64、 arm (32-bit)、 arm64、 ppc64le、s390x、そして以下の [マルチプラットフォーム提案](https://github.com/kubernetes/community/blob/master/contributors/design-proposals/multi-platform.md) です。

<!--
Only some of the network providers offer solutions for all platforms. Please consult the list of
network providers above or the documentation from each provider to figure out whether the provider
supports your chosen platform.
-->
ネットワーク・プロバイダによっては各プラットフォームに対応した解決策を提供している場合があります。先ほどのネットワーク・プロバイダと相談するか、選択したプラットフォームがサポートしているプロバイダが指示するドキュメントをご覧ください。

<!--
## Limitations {#limitations}
-->
## 制限事項 {#limitations}

<!--
Please note: kubeadm is a work in progress and these limitations will be
addressed in due course.
-->
ご注意ください: kubeadm の開発は進行中であり、用途によっては制限が加わる場合もあります。

<!--
1. The cluster created here has a single master, with a single etcd database
   running on it. This means that if the master fails, your cluster may lose
   data and may need to be recreated from scratch. Adding HA support
   (multiple etcd servers, multiple API servers, etc) to kubeadm is
   still a work-in-progress.

   Workaround: regularly
   [back up etcd](https://coreos.com/etcd/docs/latest/admin_guide.html). The
   etcd data directory configured by kubeadm is at `/var/lib/etcd` on the master.
-->
1. ここで作成したクラスタはマスタが１つだけで、１つの etcd データベースを動かしています。つまり、もしもマスタで障害が起こればデータを失うかもしれませんし、場合によってはゼロから環境再構築の必要が出てくるかもしれません。HA サポート（複数の ecd サーバ、複数の API サーバ等）の追加により、kubeadm は動作し続けるようになります。

   回避方法: 通常は [etcd をバックアップ](https://coreos.com/etcd/docs/latest/admin_guide.html) 。マスタ上の kubeadm が設定する etcd データ・ディレクトリは `/var/lib/etcd` です。

<!--
## Troubleshooting {#troubleshooting}
-->
## トラブルシューティング {#troubleshooting}

<!--
If you are running into difficulties with kubeadm, please consult our [troubleshooting docs](/docs/setup/independent/troubleshooting-kubeadm/).
-->
kubeadm の挙動で問題があれば、 [トラブルシューティング・ドキュメント](/jp/docs/setup/independent/troubleshooting-kubeadm/)  をご覧ください。



