---
reviewers:
- davidopp
- thockin
title: サービスとポッドに対する DNS
content_template: templates/concept
weight: 20
---
{{% capture overview %}}
<!--
This page provides an overview of DNS support by Kubernetes.
-->
このページは Kubernetes がサポートする DNS の概要を説明します。
{{% /capture %}}

{{% capture body %}}

<!--
## Introduction
-->
## はじめに {#introduction}

<!--
Kubernetes DNS schedules a DNS Pod and Service on the cluster, and configures
the kubelets to tell individual containers to use the DNS Service's IP to
resolve DNS names.
-->
Kubernetes DNS は DNS ポッドとサービスをクラスタ上にスケジュールし、DNS サービスの IP を DNS 名前解決につかうために、kubelet が個々のコンテナに対して伝えられるように設定します。

<!--
### What things get DNS names?
-->
### DNS 名を取得するとは？

<!--
Every Service defined in the cluster (including the DNS server itself) is
assigned a DNS name.  By default, a client Pod's DNS search list will
include the Pod's own namespace and the cluster's default domain.  This is best
illustrated by example:
-->
クラスタ内で定義された各サービス（DNS サーバ自身も含みます）には DNS 名が割り当てられます。
デフォルトでは、クライアントのポッド DNS はポッド自身が含まれる名前空間とクラスタのデフォルト・ドメインを探します。
例を使ってこれがベストだと説明しましょう。

<!--
Assume a Service named `foo` in the Kubernetes namespace `bar`.  A Pod running
in namespace `bar` can look up this service by simply doing a DNS query for
`foo`.  A Pod running in namespace `quux` can look up this service by doing a
DNS query for `foo.bar`.
-->
Kubernetes 名前空間 `bar` でサービス名 `foo` があるものとします。
名前空間内 `bar`  で実行中のポッドは、 `foo` の DNS を問い合わせる（クエリする）だけで簡単にサービスを名前解決できます。
`quux` 名前空間内で実行中のポッドが、このサービスに対する DNS 問い合わせをするには、 `foo.bar` で行います。

<!--
The following sections detail the supported record types and layout that is
supported.  Any other layout or names or queries that happen to work are
considered implementation details and are subject to change without warning.
For more up-to-date specification, see
[Kubernetes DNS-Based Service Discovery](https://github.com/kubernetes/dns/blob/master/docs/specification.md).
-->
以下のセクションではサポートしているレコード型とレイアウトの詳細を扱います。
他のレイアウトや名前やクエリも実装上は動作すると思われますが、警告無く変更が発生する場合があります。
詳細については [Kubernetes DNS-Based Service Discovery](https://github.com/kubernetes/dns/blob/master/docs/specification.md) をご覧ください。

<!--
## Services
-->
## サービス {#services}

<!--
### A records
-->
### A レコード {#a-records}

<!--
"Normal" (not headless) Services are assigned a DNS A record for a name of the
form `my-svc.my-namespace.svc.cluster.local`.  This resolves to the cluster IP
of the Service.
-->
「通常」の（ヘッドレスではない）サービスは `my-svc.my-namespace.svc.cluster.local` の形式で DNS A レコードが割り当てられます。

<!--
"Headless" (without a cluster IP) Services are also assigned a DNS A record for
a name of the form `my-svc.my-namespace.svc.cluster.local`.  Unlike normal
Services, this resolves to the set of IPs of the pods selected by the Service.
Clients are expected to consume the set or else use standard round-robin
selection from the set.
-->
"ヘッドレス"（クラスタ IP がない）サービスもまた `my-svc.my-namespace.svc.cluster.local` の形式で DNS A レコードが割り当て在られます。
通常のサービスとことなり、これで解決するのはサービスで選択されたポッドの IP の集まりです。
クライアントは集まり（セット）を使うにあたっては、セットの集まりを使わなければ、選ばれるのは標準的なラウンド・ロビンが使われているのを想定しています。

<!--
### SRV records
-->
### SRV レコード {#srv-records}

<!--
SRV Records are created for named ports that are part of normal or [Headless
Services](/docs/concepts/services-networking/service/#headless-services).
For each named port, the SRV record would have the form
`_my-port-name._my-port-protocol.my-svc.my-namespace.svc.cluster.local`.
For a regular service, this resolves to the port number and the CNAME:
`my-svc.my-namespace.svc.cluster.local`.
For a headless service, this resolves to multiple answers, one for each pod
that is backing the service, and contains the port number and a CNAME of the pod
of the form `auto-generated-name.my-svc.my-namespace.svc.cluster.local`.
-->
SRV レコードは名前を付けたポートのために使うものであり、通常もしくは  [ヘッドレス・サービス](/jp/docs/concepts/services-networking/service/#headless-services) の一部です。
名前を付けられたポート用の SRV レコードは `_my-port-name._my-port-protocol.my-svc.my-namespace.svc.cluster.local` の形式です。
通常のサービスであれば、これはポート番号を解決し、`my-svc.my-namespace.svc.cluster.local` の別名（CNAME）となります。
ヘッドレス・サービスであれば、各ポッドの複数の回答を名前解決します。結果は背後にあるサービスやコンテナのポート番号と、`auto-generated-name.my-svc.my-namespace.svc.cluster.local` の形式のポッド別名（CNAME）です。

<!--
## Pods
-->
## ポッド {#pods}

<!--
### A Records
-->
### A レコード {#a-records}

<!--
When enabled, pods are assigned a DNS A record in the form of
"`pod-ip-address.my-namespace.pod.cluster.local`".
-->
ポッドが有効になると、 "`pod-ip-address.my-namespace.pod.cluster.local`" の形式で DNS A レコードが割り当てられます。

<!--
For example, a pod with IP `1.2.3.4` in the namespace `default` with a DNS name
of `cluster.local` would have an entry: `1-2-3-4.default.pod.cluster.local`.
-->
たとえば、ポッドの IP が `1.2.3.4` 、名前空間は `default` 、クラスタの DNS 名が `cloud.local` であれば、エントリは ``1-2-3-4.default.pod.cluster.local` です。

<!--
### Pod's hostname and subdomain fields
-->
### ポッドのホスト名とサブドメイン・フィールド {#pods-hostname-and-subdomain-fields}

<!--
Currently when a pod is created, its hostname is the Pod's `metadata.name` value.
-->
ポッドを作成すると、 `metadata.name` の値がポッドのホスト名（hostname）になります。

<!--
The Pod spec has an optional `hostname` field, which can be used to specify the
Pod's hostname. When specified, it takes precedence over the Pod's name to be
the hostname of the pod. For example, given a Pod with `hostname` set to
"`my-host`", the Pod will have its hostname set to "`my-host`".
-->
ポッドの spec でオプションの `hostname` フィールドがあれば、それがポッドのホスト名指定に使われます。
もしも指定があれば、こちらがポッドのホスト名として優先されます。
たとえば、ポッドの `hostname` に "`my-host`" を指定すると、ポッドのホスト名も "`my-host`" に設定されます。

<!--
The Pod spec also has an optional `subdomain` field which can be used to specify
its subdomain. For example, a Pod with `hostname` set to "`foo`", and `subdomain`
set to "`bar`", in namespace "`my-namespace`", will have the fully qualified
domain name (FQDN) "`foo.bar.my-namespace.svc.cluster.local`".
-->
また、ポッド Spec にオプションの `subdomain` フィールドがあり、ここでサブドメインを指定できます。
たとえば、名前空間 "'my-namespace'" で、ポッドの `hostname` を "`foo`" に設定し、 `subdomain` を "`bar`" に設定すると、完全修飾ドメイン名（FQDN）は "`foo.bar.my-namespace.svc.cluster.local`" です。


<!--
Example:
-->
例：

```yaml
apiVersion: v1
kind: Service
metadata:
  name: default-subdomain
spec:
  selector:
    name: busybox
  clusterIP: None
  ports:
  - name: foo # Actually, no port is needed.
    port: 1234
    targetPort: 1234
---
apiVersion: v1
kind: Pod
metadata:
  name: busybox1
  labels:
    name: busybox
spec:
  hostname: busybox-1
  subdomain: default-subdomain
  containers:
  - image: busybox
    command:
      - sleep
      - "3600"
    name: busybox
---
apiVersion: v1
kind: Pod
metadata:
  name: busybox2
  labels:
    name: busybox
spec:
  hostname: busybox-2
  subdomain: default-subdomain
  containers:
  - image: busybox
    command:
      - sleep
      - "3600"
    name: busybox
```

<!--
If there exists a headless service in the same namespace as the pod and with
the same name as the subdomain, the cluster's KubeDNS Server also returns an A
record for the Pod's fully qualified hostname.
For example, given a Pod with the hostname set to "`busybox-1`" and the subdomain set to
"`default-subdomain`", and a headless Service named "`default-subdomain`" in
the same namespace, the pod will see its own FQDN as
"`busybox-1.default-subdomain.my-namespace.svc.cluster.local`". DNS serves an
A record at that name, pointing to the Pod's IP. Both pods "`busybox1`" and
"`busybox2`" can have their distinct A records.
-->
ヘッドレス・サービスがポッドと同じ名前空間に存在し、サブドメイン名と同じだれば、クラスタの KubeDNS サーバもポッドの完全修飾ドメイン名を返します。
たとえば、ポッドに対してホスト名を "`busybox-1`" と指定し、サブドメインを "`default-subdomain`" とし、同じ名前空間内のヘッドレス・サービスの名前を  "`default-subdomain`" とすると、ポッドは自身の FQDNS が "`busybox-1.default-subdomain.my-namespace.svc.cluster.local`" として見えます。
DNS が提供する A レコードが示す名前は、ポッドの IP を示します。
”`busybox1`” と "`busybox2`" は明確に異なる A レコードを持ちます。

<!--
The Endpoints object can specify the `hostname` for any endpoint addresses,
along with its IP.
-->
エンドポイント・オブジェクトは、あらゆるエンドポイントのアドレスのために `hostname` を IP アドレスを一緒に指定できます。

<!--
### Pod's DNS Policy
-->
### ポッドの DNS 方針 （ポリシー）{#pods-dns-entry}

<!--
DNS policies can be set on a per-pod basis. Currently Kubernetes supports the
following pod-specific DNS policies. These policies are specified in the
`dnsPolicy` field of a Pod Spec.
-->
DNS 方針（ポリシー）はポッドを基本とする単位で指定できます。
現時点では、Kubernetes は以下のポッド固有 DNS 方針（ポリシー）に従います。
以下の方針を指定するのは、ポッド Spec 内の `dnsPolicy` フィールドです。

<!--
- "`Default`": The Pod inherits the name resolution configuration from the node
  that the pods run on.
  See [related discussion](/docs/tasks/administer-cluster/dns-custom-nameservers/#inheriting-dns-from-the-node)
  for more details.
- "`ClusterFirst`": Any DNS query that does not match the configured cluster
  domain suffix, such as "`www.kubernetes.io`", is forwarded to the upstream
  nameserver inherited from the node. Cluster administrators may have extra
  stub-domain and upstream DNS servers configured.
  See [related discussion](/docs/tasks/administer-cluster/dns-custom-nameservers/#impacts-on-pods)
  for details on how DNS queries are handled in those cases.
- "`ClusterFirstWithHostNet`": For Pods running with hostNetwork, you should
  explicitly set its DNS policy "`ClusterFirstWithHostNet`".
- "`None`": A new option value introduced in Kubernetes v1.9 (Beta in v1.10). It
  allows a Pod to ignore DNS settings from the Kubernetes environment. All DNS
  settings are supposed to be provided using the `dnsConfig` field in the Pod Spec.
  See [DNS config](#dns-config) subsection below.
-->
- "`Default`"： ポッドを実行するノードから、ポッドは名前解決の設定情報を継承します。詳細は [関連する議論](/jp/docs/tasks/administer-cluster/dns-custom-nameservers/#inheriting-dns-from-the-node) をご覧ください。
- "`ClusterFirst`"： あらゆる DNS 問い合わせは "`www.kubernetes.io`" のような設定済みのドメイン・サフィックスと一致せず、ノードから継承した上流のネームサーバに転送します。クラスタ管理者は外部のサブドメインと序売るの DNS サーバを設定するでしょう。詳細は [関連する議論](/jp/docs/tasks/administer-cluster/dns-custom-nameservers/#impacts-on-pods) をご覧ください。
- "`ClusterFirstWithHostNet`"： hostNetwork で動作するポッドであれば、 DNS ポリシーを"`ClusterFirstWithHostNet`" で明示すべきでしょう。
- "`None`"：Kubernetes 1.9 で導入された新しい値です（v1.10 ではベータです）。Kubernetes 環境の DNS 設定を、ポッドが無視できるようにします。全ての DNS 設定はポッド Spec の `dnsConfig` を使って指定します。 [DNS config](#dns-config) については下の方のセクションをご覧ください。

{{< note >}}
<!--
**NOTE:** "Default" is not the default DNS policy. If `dnsPolicy` is not
explicitly specified, then “ClusterFirst” is used.
-->
**メモ：**Default** はデフォルトの DNS 方針ではありません。もし `dnsPolicy` が指定されていなければ、 "ClusterFirst" が使われます。
{{< /note >}}

<!--
The example below shows a Pod with its DNS policy set to
"`ClusterFirstWithHostNet`" because it has `hostNetwork` set to `true`.
-->
以下の例はポッドに対して DNS 方針を "`ClusterFirstWithHostNet`" に指定しますが、 `hostNetwork` を `true`  に指定しているからです。

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: busybox
  namespace: default
spec:
  containers:
  - image: busybox
    command:
      - sleep
      - "3600"
    imagePullPolicy: IfNotPresent
    name: busybox
  restartPolicy: Always
  hostNetwork: true
  dnsPolicy: ClusterFirstWithHostNet
```

<!--
### Pod's DNS Config
-->
### ポッドの DNS Config {#pods-dns-config}

<!--
Kubernetes v1.9 introduces an Alpha feature (Beta in v1.10) that allows users more
control on the DNS settings for a Pod. This feature is enabled by default in v1.10.
To enable this feature in v1.9, the cluster administrator
needs to enable the `CustomPodDNS` feature gate on the apiserver and the kubelet,
for example, "`--feature-gates=CustomPodDNS=true,...`".
When the feature gate is enabled, users can set the `dnsPolicy` field of a Pod
to "`None`" and they can add a new field `dnsConfig` to a Pod Spec.
-->
Kubernetes v1.9 ではアルファ機能として（v1.10 ではベータとして）、ポッドに対してユーザが DNS を設定できるようにしました。
この機能は v1.10 ではデフォルトになりました。
v1.9 でこの機能をデフォルトにするには、クラスタ管理者が `CustomPodDNS` 機能ゲート（feature gate）を apiserver で有効化するため、kubelet では  "`--feature-gates=CustomPodDNS=true,...`" を指定します。
機能ゲートが有効であれば、ポッドに対して `dnsPolicy`フィールドを "`None`" に指定子、ポッド Spec で新しい `dnsConfig` フィールドを追加できます。

<!--
The `dnsConfig` field is optional and it can work with any `dnsPolicy` settings.
However, when a Pod's `dnsPolicy` is set to "`None`", the `dnsConfig` field has
to be specified.
-->
`dnsConfig` フィールドはオプションであり、 様々な `dnsPolicy` の設定で使えます。
しかしながら、ポッドの `dnsPolicy` を "`none`" に指定すると、 `dnsConfig` フィールドは指定されません。

<!--
Below are the properties a user can specify in the `dnsConfig` field:
-->
以下はユーザが `dnsConfig` フィールドで指定できる属性です。

<!--
- `nameservers`: a list of IP addresses that will be used as DNS servers for the
  Pod. There can be at most 3 IP addresses specified. When the Pod's `dnsPolicy`
  is set to "`None`", the list must contain at least one IP address, otherwise
  this property is optional.
  The servers listed will be combined to the base nameservers generated from the
  specified DNS policy with duplicate addresses removed.
- `searches`: a list of DNS search domains for hostname lookup in the Pod.
  This property is optional. When specified, the provided list will be merged
  into the base search domain names generated from the chosen DNS policy.
  Duplicate domain names are removed.
  Kubernetes allows for at most 6 search domains.
- `options`: an optional list of objects where each object may have a `name`
  property (required) and a `value` property (optional). The contents in this
  property will be merged to the options generated from the specified DNS policy.
  Duplicate entries are removed.
-->
- `nameservers` - ポッドが DNS サーバとして使うための、IP アドレスの一覧です。最大で３つの IP アドレスを指定できます。ポッドの `dnsPolicy` を "`None`" に指定すると、一覧には少なくとも１つの IP アドレスが必要です。そうしなければ、属性指定がオプションとなります。サーバ一覧は DNS 方針を指定するにあたり、名前空間の生成に使われますが、重複するものは削除されます。
- `searches` - ポッドでホスト名の名前解決で DNS 検索ドメインに使う一覧です。指定すると、指定されたリストがベースの検索ドメイン名にマージされます。重複するドメイン名は削除されます。Kubernetes は６つの検索ドメインを指定できます。
- `options` - 各オブジェクトが持つオプションのオブジェクト一覧で、 `name` 属性（必須）と `value` 属性（オプション）があります。属性に含まれるのは、指定した DNS 方針に従ってオプションが統合され、重複があれば削除されます。

<!--
The following is an example Pod with custom DNS settings:
-->
以下はポッドに対するカスタム DNS 設定の例です。

{{< code file="custom-dns.yaml" >}}

<!--
When the Pod above is created, the container `test` gets the following contents
in its `/etc/resolv.conf` file:
-->
こちらのポッドが作成されると、コンテナ `test` は以下の内容を `/etc/resolv.conf` ファイル内で確認できます。

```
nameserver 1.2.3.4
search ns1.svc.cluster.local my.dns.search.suffix
options ndots:2 edns0
```

<!--
For IPv6 setup, search path and name server should be setup like this:
-->
IPv6 をセットアップするには、search パスとサーバ名は次のようにすべきです：

```
$ kubectl exec -it busybox -- cat /etc/resolv.conf
nameserver fd00:79:30::a
search default.svc.cluster.local svc.cluster.local cluster.local
options ndots:5
```

{{% /capture %}}

{{% capture whatsnext %}}

<!--
For guidance on administering DNS configurations, check
[Configure DNS Service](/docs/tasks/administer-cluster/dns-custom-nameservers/)
-->
DNS 設定情報に関する管理のガイドは、 [Configure DNS Service](/jp/docs/tasks/administer-cluster/dns-custom-nameservers/) をご覧ください。

{{% /capture %}}


