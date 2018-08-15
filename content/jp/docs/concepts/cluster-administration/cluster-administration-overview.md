---
reviewers:
- davidopp
- lavalamp
title: クラスタ管理概要
content_template: templates/concept
weight: 10
---

{{% capture overview %}}
<!--
The cluster administration overview is for anyone creating or administering a Kubernetes cluster.
It assumes some familiarity with core Kubernetes [concepts](/docs/concepts/).
-->
クラスタ管理概要の対象は、Kubernetes クラスタを構築または管理をする方です。
核心となる Kubernetes [概念](/jp/docs/concepts/) をある程度熟知しているのが前提です。
{{% /capture %}}

{{% capture body %}}
<!--
## Planning a cluster
-->
## クラスタの設計 {#planning-a-cluster}

<!--
See the guides in [Picking the Right Solution](/docs/setup/pick-right-solution/) for examples of how to plan, set up, and configure Kubernetes clusters. The solutions listed in this article are called *distros*.
-->
Kubernetes クラスタをどのように設計（計画）、セットアップ（構築）、設定するかの例は、[適切な解決策（ソリューション）の選択](/jp/docs/setup/pick-right-solution/) にあるガイドをご覧ください。

<!--
Before choosing a guide, here are some considerations:
-->
ガイドを選ぶ前に、いくつかの検討事項があります：

<!--
 - Do you just want to try out Kubernetes on your computer, or do you want to build a high-availability, multi-node cluster? Choose distros best suited for your needs.
 - **If you are designing for high-availability**, learn about configuring [clusters in multiple zones](/docs/concepts/cluster-administration/federation/).
 - Will you be using **a hosted Kubernetes cluster**, such as [Google Kubernetes Engine](https://cloud.google.com/kubernetes-engine/), or **hosting your own cluster**?
 - Will your cluster be **on-premises**, or **in the cloud (IaaS)**? Kubernetes does not directly support hybrid clusters. Instead, you can set up multiple clusters.
 - **If you are configuring Kubernetes on-premises**, consider which [networking model](/docs/concepts/cluster-administration/networking/) fits best.
 - Will you be running Kubernetes on **"bare metal" hardware** or on **virtual machines (VMs)**?
 - Do you **just want to run a cluster**, or do you expect to do **active development of Kubernetes project code**? If the
   latter, choose an actively-developed distro. Some distros only use binary releases, but
   offer a greater variety of choices.
 - Familiarize yourself with the [components](/docs/admin/cluster-components/) needed to run a cluster.
-->
 - 自分のコンピュータ上で Kubernetes を試してみるだけですか？ 高可用性や複数ノードのクラスタを構築したいですか？ 必要に応じて最適なディストリビューションを選択してください。
 - **高可用性の設計をしている場合は** 、 [複数ゾーンでのクラスタ](/jp/docs/concepts/cluster-administration/federation/) 設定について学んでください。
 -  [Google Kubernetes Engine](https://cloud.google.com/kubernetes-engine/) や **自分自身でクラスタをホスティング（所有）**  して、**ホステッド（所有型） Kubernetes クラスタ** を使いますか？
 - クラスタは **オン・プレミス（on-premises）** ですか、 **クラウド（IaaS）内** ですか？ Kubernetes はハイブリッド／クラスタを直接サポートしません。そのかわりに、自分自身で複数のクラスタをセットアップ（構築）できます。
 - **Kubernetes をオンプレミスで設定する場合** 、どの [ネットワーク形成モデル](/jp/docs/concepts/cluster-administration/networking/) が一番適切なのか検討します。
 - **"ベア・メタル" ハードウェア** あるいは **仮想マシン（VM）** のどちらで Kubernetes を実行しようとしていますか？
 - **単位クラスタを動かしたいだけ**  ですか？ それとも **Kubernetes プロジェクトのコードをアクティブに展開（デプロイ）** するつもりですか？ 後者でしたら、活発に開発されているディストリビューションを選択します。ディストリビューションによってはバイナリ・リリースしか提供されていませんが、様々な選択肢が提供されています。
 - クラスタを実行に必要な [構成要素（コンポーネント）](/jp/docs/concepts/overview/components/) に自分自身が習熟している。

<!--
Note: Not all distros are actively maintained. Choose distros which have been tested with a recent version of Kubernetes.
-->
メモ：ディストリビューションは全てが活発に維持（メンテナンス）されているわけではありません。
最近の Kubernetes バージョンをテスト済みのディストリビューションを選んでください。

<!--
If you are using a guide involving Salt, see [Configuring Kubernetes with Salt](/docs/admin/salt/).
-->
既に Salt を導入しているガイドを使っている場合は、 [Salt で Kubernetes を設定](/jp/docs/admin/salt/) をご覧ください。

<!--
## Managing a cluster
-->
## クラスタ管理 {#managing-a-cluster}

<!--
* [Managing a cluster](/docs/tasks/administer-cluster/cluster-management/) describes several topics related to the lifecycle of a cluster: creating a new cluster, upgrading your cluster’s master and worker nodes, performing node maintenance (e.g. kernel upgrades), and upgrading the Kubernetes API version of a running cluster.

* Learn how to [manage nodes](/docs/concepts/nodes/node/).

* Learn how to set up and manage the [resource quota](/docs/concepts/policy/resource-quotas/) for shared clusters.
-->
* [クラスタの管理](/jp/docs/tasks/administer-cluster/cluster-management/) には、クラスタのライフサイクルに関連するトピックの記載が複数あります。
たとえば、新しいクラスタを作成、クラスタのマスタとワーカ・ノードを更新、ノードのメンテナンス対応（例：カーネル更新）、実行中のクラスタで Kubernetes API のバージョン更新です。

* [ノードの管理](/docs/concepts/nodes/node/) 方法を学びます。

* 共有されたクラスタで [リソース制限（quota）](/jp/docs/concepts/policy/resource-quotas/) の設定と管理方法を学びます。

<!--
## Securing a cluster
-->
## クラスタを安全に {#securing-a-cluster}

<!--
* [Certificates](/docs/concepts/cluster-administration/certificates/) describes the steps to generate certificates using different tool chains.

* [Kubernetes Container Environment](/docs/concepts/containers/container-environment-variables/) describes the environment for Kubelet managed containers on a Kubernetes node.

* [Controlling Access to the Kubernetes API](/docs/admin/accessing-the-api/) describes how to set up permissions for users and service accounts.

* [Authenticating](/docs/admin/authentication/) explains authentication in Kubernetes, including the various authentication options.

* [Authorization](/docs/admin/authorization/) is separate from authentication, and controls how HTTP calls are handled.

* [Using Admission Controllers](/docs/admin/admission-controllers/) explains plug-ins which intercepts requests to the Kubernetes API server after authentication and authorization.

* [Using Sysctls in a Kubernetes Cluster](/docs/concepts/cluster-administration/sysctl-cluster/) describes to an administrator how to use the `sysctl` command-line tool to set kernel parameters .

* [Auditing](/docs/tasks/debug-application-cluster/audit/) describes how to interact with Kubernetes' audit logs.
-->
* [証明書（Certificates）](/jp/docs/concepts/cluster-administration/certificates/) 
* [Kubernetes コンテナ環境変数](/jp/docs/concepts/containers/container-environment-variables/) 
* [Kubernetes API に対するアクセス制御](/jp/docs/admin/accessing-the-api/) 
* [認証（Authenticating）](/jp/docs/admin/authentication/) 
* [権限付与（Authorization）](/jp/docs/admin/authorization/) 
* [承認コントローラ（Admission Controller）を使う](/jp/docs/admin/admission-controllers/) 
* [Kubernetes クラスタで sysctl を使う](/jp/docs/concepts/cluster-administration/sysctl-cluster/)
* [監査](/jp/docs/tasks/debug-application-cluster/audit/) 

<!--
### Securing the kubelet
-->
## kubelet を安全に {#securing-the-kubelet}

<!--
  * [Master-Node communication](/docs/concepts/architecture/master-node-communication/)
  * [TLS bootstrapping](/docs/reference/command-line-tools-reference/kubelet-tls-bootstrapping/)
  * [Kubelet authentication/authorization](/docs/admin/kubelet-authentication-authorization/)
-->
  * [マスタ・ノード間の通信](/jp/docs/concepts/architecture/master-node-communication/)
  * [TLS 初期構築（ブートストラッピング）](/jp/docs/reference/command-line-tools-reference/kubelet-tls-bootstrapping/)
  * [Kubelet 認証/権限付与](/jp/docs/admin/kubelet-authentication-authorization/)

<!--
## Optional Cluster Services
-->
## オプションのクラスタ・サービス {#optional-cluster-services}

<!--
* [DNS Integration](/docs/concepts/services-networking/dns-pod-service/) describes how to resolve a DNS name directly to a Kubernetes service.

* [Logging and Monitoring Cluster Activity](/docs/concepts/cluster-administration/logging/) explains how logging in Kubernetes works and how to implement it.
-->
* [DNS 統合（インテグレーション）](/jp/docs/concepts/services-networking/dns-pod-service/) では、Kubernetes サービスに DNS 名で直接名前する方法を説明します。
* [クラスタ動作のログ出力と監視](/jp/docs/concepts/cluster-administration/logging/) では、Kubernetes のログ出力がどのように機能するかと、どのようにこれを実装しているかを説明します。

{{% /capture %}}


