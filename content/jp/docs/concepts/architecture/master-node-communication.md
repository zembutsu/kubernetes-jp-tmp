---
reviewers:
- dchen1107
- roberthbailey
- liggitt
title: マスタとノード間の通信
content_template: templates/concept
weight: 20
---

{{% capture overview %}}

<!--
This document catalogs the communication paths between the master (really the
apiserver) and the Kubernetes cluster. The intent is to allow users to
customize their installation to harden the network configuration such that
the cluster can be run on an untrusted network (or on fully public IPs on a
cloud provider).
-->
このドキュメントはマスタ（正確には apiserver）と Kubernetes クラスタ間との通信パスを列挙します。
目的は、ユーザがネットワーク設定を確固たるものとするため、インストールをカスタマイズするためです。
これには信頼できないネットワーク上でのクラスタ実行も含みます（あるいはクラウド事業者上の完全なパブリック IP 上でも含みます）。

{{% /capture %}}

{{< toc >}}

{{% capture body %}}

<!--
## Cluster -> Master
-->
## クラスタ -> マスタ {#cluster-master}

<!--
All communication paths from the cluster to the master terminate at the
apiserver (none of the other master components are designed to expose remote
services). In a typical deployment, the apiserver is configured to listen for
remote connections on a secure HTTPS port (443) with one or more forms of
client [authentication](/docs/admin/authentication/) enabled. One or more forms
of [authorization](/docs/admin/authorization/) should be enabled, especially
if [anonymous requests](/docs/admin/authentication/#anonymous-requests) or
[service account tokens](/docs/admin/authentication/#service-account-tokens)
are allowed.
-->
クラスタからマスタへの全ての通信径路（パス）は、apiserver が終点です（他のマスタ構成要素は、どれ１つとしてリモート・サービスを外に向けて公開するように設計されていません）。
典型的な展開（デプロイメント）では、apiserver は安全な HTTPS ポート (443) 上でリモート接続をリッスン（listen）し、１つまたは複数のクライアント [認証](/jp/docs/admin/authentication/) 形式を有効化するよう設定されています。
１つまたは複数の [認証](/jp/docs/admin/authentication/) 形式は有効化すべきです。
特に [匿名要求（anonymous requests）](/jp/docs/admin/authentication/#anonymous-requests) と
[サービス・アカウント・トークン（service account tokens）](/jp/docs/admin/authentication/#service-account-tokens) です。

<!--
Nodes should be provisioned with the public root certificate for the cluster
such that they can connect securely to the apiserver along with valid client
credentials. For example, on a default GCE deployment, the client credentials
provided to the kubelet are in the form of a client certificate. See
[kubelet TLS bootstrapping](/docs/reference/command-line-tools-reference/kubelet-tls-bootstrapping/)
for automated provisioning of kubelet client certificates.
-->
有効なクライアント信用証明書（credential）を持っている apiserver に対して、ノードが安全に接続できるようにするには、
ノードは公開ルート証明書（certificate）と一緒にして、クラスタへ自動構築（プロビジョン）すべきです。
たとえば、デフォルトの GCE での展開（デプロイ）では、kubelet に提供するクライアントの信用証明書（credential）は、クライアント証明書（certficate）の形式です。
Kubelet クライアント証明書を自動的に設定（プロビジョニング）するには、[kubelet TLS ブートストラッピング](/jp/docs/reference/command-line-tools-reference/kubelet-tls-bootstrapping/) をご覧ください。

<!--
Pods that wish to connect to the apiserver can do so securely by leveraging a
service account so that Kubernetes will automatically inject the public root
certificate and a valid bearer token into the pod when it is instantiated.
The `kubernetes` service (in all namespaces) is configured with a virtual IP
address that is redirected (via kube-proxy) to the HTTPS endpoint on the
apiserver.
-->
apiserver に接続したいポッドは、サービス・アカウントの活用によって、より安全に接続できるようになります。
そのためには、ポッドの初期化時、Kubernetes は公開ルート証明書と有効なベアラ－（bearer：運搬）トークンを自動的にポッドに対して投入します。
（全ての名前空間内に存在する）`kubernetes` サービスは、仮想 IP アドレスと一緒に設定されます。
IP アドレスは apiserver 上の HTTPS エンドポイントに対してリダイレクトするためです（kube-proxy を経由）。

<!--
The master components also communicate with the cluster apiserver over the secure port.
-->
また、マスタ構成要素（コンポーネント）も安全なポートを通してクラスタ apiserver と通信できます。

<!--
As a result, the default operating mode for connections from the cluster
(nodes and pods running on the nodes) to the master is secured by default
and can run over untrusted and/or public networks.
-->
結果として、クラスタ（ノードおよびポッドを実行中のノード）からマスタに対する通信のデフォルト操作モードが、デフォルトで安全です。そして、信頼されない、または、パブリック・ネットワークを越えた処理も可能です。

<!--
## Master -> Cluster
-->
## マスタ -> クラスタ {#master-cluster}

<!--
There are two primary communication paths from the master (apiserver) to the
cluster. The first is from the apiserver to the kubelet process which runs on
each node in the cluster. The second is from the apiserver to any node, pod,
or service through the apiserver's proxy functionality.
-->
マスタ（apiserver）からクラスタに対する通信経路は主に２つあります。
１つめは、apiserver からクラスタ内の各ノード上で実行している kubelet プロセスに対してです。
２つめは、apiserver から、apiserver のプロキシ機能を通して、あらゆるノード、ポッド、サービスに対してです。

### apiserver -> kubelet

<!--
The connections from the apiserver to the kubelet are used for:
-->
apiserver から kubelet への接続は、以下のために使います：

<!--
  * Fetching logs for pods.
  * Attaching (through kubectl) to running pods.
  * Providing the kubelet's port-forwarding functionality. 
-->
  * ポッド用のログを取得する
  * 実行中のポッドに対して（kubectl を通して）アタッチする
  * kubelet のポート転送機能を提供する

<!--
These connections terminate at the kubelet's HTTPS endpoint. By default, 
the apiserver does not verify the kubelet's serving certificate,
which makes the connection subject to man-in-the-middle attacks, and
**unsafe** to run over untrusted and/or public networks.
-->
通信は kubelet の HTTPS エンドポイントに到達します。
デフォルトでは apiserver は kubelet の提供する証明書（certificate）を確認しないため、接続対象の中間者攻撃や、信頼できないパブリックなネットワークでの実行は **安全ではありません** 。

<!--
To verify this connection, use the `--kubelet-certificate-authority` flag to
provide the apiserver with a root certificate bundle to use to verify the
kubelet's serving certificate.
-->
接続を検証するには apiserver に `--kubelet-certificate-authority` フラグを使い、kubelet が提供する証明書を確認するために、ルート証明書の束をを apiserver が提供するようにします。

<!--
If that is not possible, use [SSH tunneling](/docs/concepts/architecture/master-node-communication/#ssh-tunnels)
between the apiserver and kubelet if required to avoid connecting over an
untrusted or public network.
-->
これが不可能でも、信頼できないまたはパブリックなネットワークを越えての通信を避ける必要がある場合は、apiserver と kubelet 間で[SSH トンネリング](/jp/docs/concepts/architecture/master-node-communication/#ssh-tunnels) を使います。

<!--
Finally, [Kubelet authentication and/or authorization](/docs/admin/kubelet-authentication-authorization/)
should be enabled to secure the kubelet API.
-->
最終的には、 kubelet API を安全にするためには、 [Kubelet authentication（認証）および authorization（許可）](/jp/docs/admin/kubelet-authentication-authorization/) を使うべきでしょう。


<!--
### apiserver -> nodes, pods, and services
-->
### apiserer -> ノード、ポッド、サービス {#apiserver-nodes-pods-and-services}

<!--
The connections from the apiserver to a node, pod, or service default to plain
HTTP connections and are therefore neither authenticated nor encrypted. They
can be run over a secure HTTPS connection by prefixing `https:` to the node,
pod, or service name in the API URL, but they will not validate the certificate
provided by the HTTPS endpoint nor provide client credentials so while the
connection will be encrypted, it will not provide any guarantees of integrity.
These connections **are not currently safe** to run over untrusted and/or
public networks.
-->
apiserver からノード、ポッド、サービスに対する接続は、デフォルトでは単なる HTTP 接続です。
そのため、通信は認証されておらず、暗号化もされていません。
安全な HTTP 接続のためには、ノード、ポッド、サービス名の API URL の先頭に `https:` を付けますが、HTTPS エンドポイント証明書の正当性を確認していないだけでなく、クライアント信用証明（credential）も提供しません。
そのため、通信は暗号化されますが、整合性の保証は提供されません。
信頼できないおよび公開ネットワークを越える場合、これらの通信は **そのままでは安全ではあません** 。

{{% /capture %}}
