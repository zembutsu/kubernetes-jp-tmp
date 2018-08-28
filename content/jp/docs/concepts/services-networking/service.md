---
reviewers:
- bprashanth
title: サービス
content_template: templates/concept
weight: 10
---

{{< toc >}}

{{% capture overview %}}
<!--
Kubernetes [`Pods`](/docs/concepts/workloads/pods/pod/) are mortal. They are born and when they die, they
are not resurrected.  [`ReplicaSets`](/docs/concepts/workloads/controllers/replicaset/) in
particular create and destroy `Pods` dynamically (e.g. when scaling up or down).  While each `Pod` gets its own IP address, even
those IP addresses cannot be relied upon to be stable over time. This leads to
a problem: if some set of `Pods` (let's call them backends) provides
functionality to other `Pods` (let's call them frontends) inside the Kubernetes
cluster, how do those frontends find out and keep track of which backends are
in that set?
-->
Kubernetes の [`ポッド`](/jp/docs/concepts/workloads/pods/pod/) は死を免れません（停止を免れません）。
作成されたポッドが死んでも、生き返りません。
特に  [`ReplicaSet`](/jp/docs/concepts/workloads/controllers/replicaset/) では `ポッド` の作成・破棄は動的です（例：スケールアップやダウン時）。
各 `ポッド` は自身の IP アドレスを持ちます。たとえ各 IP アドレスが常に安定していない場合でもです。
この挙動によって、問題に至ります。
Kubernetes クラスタ内で、もしも `ポッド` の集まり（これらをバックエンドと呼びます）が他の `ポッド` （フロントエンドと呼びます）に機能を提供している場合、どのようにしてフロントエンドはバックエンドの場所を追跡し続けられるでしょうか。

<!--
Enter `Services`.
-->
`サービス（Service）` に参加します。

<!--
A Kubernetes `Service` is an abstraction which defines a logical set of `Pods`
and a policy by which to access them - sometimes called a micro-service.  The
set of `Pods` targeted by a `Service` is (usually) determined by a [`Label
Selector`](/docs/concepts/overview/working-with-objects/labels/#label-selectors) (see below for why you might want a
`Service` without a selector).
-->
Kubernetes `サービス（Service）` は、論理的な `ポッド` の集まりと、それらへの接続方針（ポリシー）をまとめた抽象概念であり、これらは時々マイクロサービスと呼ばれます。
`ポッド` の集まりを `サービス` として絞り込むのに決めるのは（通常は） [`ラベル・セレクタ（Label Selector）`](/jp/docs/concepts/overview/working-with-objects/labels/#label-selectors) です（セレクタがない `サービス` を使いたい場合は、以下をご覧ください）。

<!--
As an example, consider an image-processing backend which is running with 3
replicas.  Those replicas are fungible - frontends do not care which backend
they use.  While the actual `Pods` that compose the backend set may change, the
frontend clients should not need to be aware of that or keep track of the list
of backends themselves.  The `Service` abstraction enables this decoupling.
-->
たとえば、３つのレプリカ（複製）をバックエンドで実行する処理を考えましょう。
各レプリカ（複製）は代替可能です。
つまり、フロントエンドはバックエンドで何を使おうが気にしません。
バックエンドを構成する実際の `ポッド` が変わったとしても、フロントエンドのクライアントは注意を払う必要がありません。。
また、バックエンドの一覧を追跡し続ける必要もありません。
`サービス` の抽象化によって、ｋのような分離（分断）を実現します。

<!--
For Kubernetes-native applications, Kubernetes offers a simple `Endpoints` API
that is updated whenever the set of `Pods` in a `Service` changes.  For
non-native applications, Kubernetes offers a virtual-IP-based bridge to Services
which redirects to the backend `Pods`.
-->
Kubernetes ネイティブなアプリケーションでは、Kubernetes はシンプルな `エンドポイント（Endpoint）` API を提供するので、ここから `サービス` 内の `ポッド` がどのような状態に変更したかを把握できます。

{{% /capture %}}

{{% capture body %}}

<!--
## Defining a service
-->
## サービスの定義 {#defining-a-service}

<!--
A `Service` in Kubernetes is a REST object, similar to a `Pod`.  Like all of the
REST objects, a `Service` definition can be POSTed to the apiserver to create a
new instance.  For example, suppose you have a set of `Pods` that each expose
port 9376 and carry a label `"app=MyApp"`.
-->
Kubernetes 内の `サービス（Service）` とは REST オブジェクトであり、 `ポッド（Pod）` と似ています。
全ての REST オブジェクト同様に、 apiserver に対して `Service（サービス）` の定義を投げる（POSTする）と、新しいサービスを作成します。
たとえば、ポート 9376 を外部に公開してラベル '"app=MyAPp"' がある `ポッド` の集まりを持っていると家庭しましょう。

```yaml
kind: Service
apiVersion: v1
metadata:
  name: my-service
spec:
  selector:
    app: MyApp
  ports:
  - protocol: TCP
    port: 80
    targetPort: 9376
```

<!--
This specification will create a new `Service` object named "my-service" which
targets TCP port 9376 on any `Pod` with the `"app=MyApp"` label.  This `Service`
will also be assigned an IP address (sometimes called the "cluster IP"), which
is used by the service proxies (see below).  The `Service`'s selector will be
evaluated continuously and the results will be POSTed to an `Endpoints` object
also named "my-service".
-->
この指定は、「mpo-to 
 y-service」という名称の新しい `Service` を作成します。
サービスは `"app=MyApp"` ラベルを持つあらゆる `ポッド` 上の、 TCP ポート 9376 を対象（ターゲット）とします。
また、`Service` には IP アドレスが割り当てられ（「クラスタ IP」とも呼ばれます）、サービスのプロキシとして使われます（以下をご覧ください）。
`Service` のセレクタ（selector）は継続的に評価を行います。そして、その結果を名称「my-service」オブジェクトの `Endpoint（エンドポイント）` として apiserver に投稿します。

<!--
Note that a `Service` can map an incoming port to any `targetPort`.  By default
the `targetPort` will be set to the same value as the `port` field.  Perhaps
more interesting is that `targetPort` can be a string, referring to the name of
a port in the backend `Pods`.  The actual port number assigned to that name can
be different in each backend `Pod`. This offers a lot of flexibility for
deploying and evolving your `Services`.  For example, you can change the port
number that pods expose in the next version of your backend software, without
breaking clients.
-->
`Service` は受信用のポート（incoming port）として、あらゆる `targetPort` （対象ポート／ターゲット・ポート）として割り当て（マップ）できるのに注目します。
デフォルトでは、 `targetPort` （対象ポート）は `port` フィールドと同じ値が設定されます。
面白いことに、`targetPort` を文字列にして、 `ポッド` のバックエンドにあるポートを名前で参照できます。
名前のところは、バックエンドの `ポッド` ごとにそれぞれ実際のポート番号が割り当てられます。
これにより、 `サービス`  に対する配置（デプロイ）と発達に大きな柔軟性をもたらします。
たとえば、次のバージョンのバックエンド・ソフトウェアで、ポッドが公開するポート番号を変更したとしても、クライアントの停止はありません。

<!--
Kubernetes `Services` support `TCP` and `UDP` for protocols.  The default
is `TCP`.
-->

pKubernetes `サービス` は `TCP` と `UDP`  プロトコルをサポートします。
デフォルトは `TCP` です。

<!--
### Services without selectors
-->
### セレクタのないサービス {#services-without-selectors}

<!--
Services generally abstract access to Kubernetes `Pods`, but they can also
abstract other kinds of backends.  For example:
-->
サービスは通常 Kubernetes `ポッド` に対するアクセスを抽象化しますが、他のバックエンドなども抽象化します。たとえば、

<!--
  * You want to have an external database cluster in production, but in test
    you use your own databases.
  * You want to point your service to a service in another
    [`Namespace`](/docs/concepts/overview/working-with-objects/namespaces/) or on another cluster.
  * You are migrating your workload to Kubernetes and some of your backends run
    outside of Kubernetes.
-->
  * 本番環境（プロダクション）では外部のデータベース・クラスタを使いたいが、テストでは自身のデータベースを使う
  * サービスから他の [`名前空間（Namespace）`](/jp/docs/concepts/overview/working-with-objects/namespaces/) や他のクラスタにあるサービスを示したい
  * Kubernetes 処理の移行や、バックエンドのいくつかを Kubernetes の外で実行したい

<!--
In any of these scenarios you can define a service without a selector:
-->
これら３つのシナリオの場合、セレクタの無いサービスを定義できます。

```yaml
kind: Service
apiVersion: v1
metadata:
  name: my-service
spec:
  ports:
  - protocol: TCP
    port: 80
    targetPort: 9376
```

<!--
Because this service has no selector, the corresponding `Endpoints` object will not be
created. You can manually map the service to your own specific endpoints:
-->
このサービスはセレクタを持たないため、 `Endpoints` に相当するオブジェクトは作成されません。
自分自身でエンドポイントを指定して、サービスに手動で割り当て（マップ）できます。

```yaml
kind: Endpoints
apiVersion: v1
metadata:
  name: my-service
subsets:
  - addresses:
      - ip: 1.2.3.4
    ports:
      - port: 9376
```

{{< note >}}
<!--
**NOTE** The endpoint IPs may not be loopback (127.0.0.0/8), link-local
(169.254.0.0/16), or link-local multicast (224.0.0.0/24). They cannot be the
cluster IPs of other Kubernetes services either because the `kube-proxy`
component doesn't support virtual IPs as destination yet.
-->
**メモ：** エンドポイントの IP アドレスから除外されるのは、ループバック（127.0.0.0/8）、リンク・ローカル（169.254.0.0/16）、リンク・ローカル・マルチキャスト（224.0.0.0/24）です。
他のこれらは Kubernetes サービス用のクラスタ IP として使えないだけでなく、 `kube-proxy` コンポーネントでも、仮想 IP を到達先として指定できません。
{{< /note >}}

<!--
Accessing a `Service` without a selector works the same as if it had a selector.
The traffic will be routed to endpoints defined by the user (`1.2.3.4:9376` in
this example).
-->
セレクタがない `Service` に接続しても、セレクタがあるのと同じように処理できます。
トラフィックはユーザが定義したエンドポイントに対して転送（ルート：routed）されます（この例では `1.2.3.4:9376` です）。

<!--
An ExternalName service is a special case of service that does not have
selectors. It does not define any ports or Endpoints. Rather, it serves as a
way to return an alias to an external service residing outside the cluster.
-->
ExternalName（外部名）サービスは、セレクタを持たない特別なサービスの例です。
ポートやエンドポイントを定義していません。
そのかわりに、クラスタの外に位置する外部サービスに対する別名（エイリアス）を返します。

```yaml
kind: Service
apiVersion: v1
metadata:
  name: my-service
  namespace: prod
spec:
  type: ExternalName
  externalName: my.database.example.com
```

<!--
When looking up the host `my-service.prod.svc.CLUSTER`, the cluster DNS service
will return a `CNAME` record with the value `my.database.example.com`. Accessing
such a service works in the same way as others, with the only difference that
the redirection happens at the DNS level and no proxying or forwarding occurs.
Should you later decide to move your database into your cluster, you can start
its pods, add appropriate selectors or endpoints and change the service `type`.
-->
ホスト `my-service.prod.svc.CLUSTER` を調べる（名前解決する）と、クラスタの DNS サービスは `my.database.example.com` の `CNAME` レコードの値を返します。
サービスを処理するための接続は他と同様ですが、唯一違うのは、DNS レベルでの転送（リダイレクト）が発生することであり、プロキシや転送処理は発生しません。
後でデータベースを自分のクラスタ内にいれることにあっても、このポッドを使って開始できますし、適切なセレクタやエンドポイントと、サービスの `type` を変更するだけです。

<!--
## Virtual IPs and service proxies
-->
## 仮想 IP とサービス・プロキシ {#virtual-ips-and-service-proxies}

<!--
Every node in a Kubernetes cluster runs a `kube-proxy`.  `kube-proxy` is
responsible for implementing a form of virtual IP for `Services` of type other
than `ExternalName`.
In Kubernetes v1.0, `Services` are a "layer 4" (TCP/UDP over IP) construct, the
proxy was purely in userspace.  In Kubernetes v1.1, the `Ingress` API was added
(beta) to represent "layer 7"(HTTP) services, iptables proxy was added too,
and become the default operating mode since Kubernetes v1.2. In Kubernetes v1.8.0-beta.0,
ipvs proxy was added.
-->
Kubernetes クラスタの各ノードで実行では `kube-proxy` が動作します。
`ExternalName` 以外の `Service` タイプに対しては、`kube-proxy` が仮想 IP 方式の実装に責任を持ちます。
Kubernetes 1.0 では、 `Service` は「レイヤ４」（TCP/UDP over IP）を構築し、プロキシは純粋にユーザ空間（userspace）におけるものです。
Kubernetes 1.1 では、 `Ingress` API が追加されました（ベータ）。これは「レイヤ７」（HTTP）サービスに相当し、iptables プロキシも追加されました。
そして、Kubernetes v1.2 からは、これがデフォルトの運用状態（オペレーティング・モード）となりました。
Kubernetes v1.8.0-beta.0で、ipvs プロキシが追加されました。

<!--
### Proxy-mode: userspace
-->
### プロキシ・モード：userspace（ユーザ空間） {#proxy-mode-userspace}


In this mode, kube-proxy watches the Kubernetes master for the addition and
removal of `Service` and `Endpoints` objects. For each `Service` it opens a
port (randomly chosen) on the local node.  Any connections to this "proxy port"
will be proxied to one of the `Service`'s backend `Pods` (as reported in
`Endpoints`).  Which backend `Pod`  to use is decided based on the
`SessionAffinity` of the `Service`.  Lastly, it installs iptables rules which
capture traffic to the `Service`'s `clusterIP` (which is virtual) and `Port`
and redirects that traffic to the proxy port which proxies the backend `Pod`.
By default, the choice of backend is round robin.

このモードでは、kube-proxy は Kubernetes マスタに対し、 `Service` と `Endpoints`  オブジェクトの追加と削除を監視します。
各 `Service` に対して、ノーカル・ノード上のポートを（ランダムに選択して）開きます。
この「proxy port」に対するあらゆる接続は、`Service` のバックエンドにある `ポッド`（ `Endpoints` として申請している ）の１つにプロキシされます。
バックエンド `ポッド` を決定するにあたっては、 `Service`  の `SessionAffinity` をベースとしています。
最終的にインストールする iptables のルールとは、 `サービス` の `クラスタ IP`（仮想的なもの）と `ポート` とリダイレクトするためにトラフィックを集めるものです。これにより、プロキシ・ポートに対するトラフィックが、バックエンドの `ポッド` に対するプロキシ（代理）となります。
デフォルトでは、バックエンドの選択は、ラウンド・ロビンです。

![Services overview diagram for userspace proxy](/images/docs/services-userspace-overview.svg)

<!--
Note that in the above diagram, `clusterIP` is shown as `ServiceIP`.
-->
上図で `ServiceIP` （サービスIP）として書かれている部分が `clouserIP` （クラスタIP）ですのでご注意ください。

<!--
### Proxy-mode: iptables
-->
### プロキシ・モード：iptables

<!--
In this mode, kube-proxy watches the Kubernetes master for the addition and
removal of `Service` and `Endpoints` objects. For each `Service`, it installs
iptables rules which capture traffic to the `Service`'s `clusterIP` (which is
virtual) and `Port` and redirects that traffic to one of the `Service`'s
backend sets.  For each `Endpoints` object, it installs iptables rules which
select a backend `Pod`. By default, the choice of backend is random.
-->
このモードでは、kube-proxy は Kubernetes マスタで `Service` と `Endpoints` オブジェクトの追加と削除を監視します。
`Service` ごとに iptables ルールをインストールし、 `Service` の `clusterIP` （仮想のもの）と `Port`に対するトラフィックを集めます（キャプチャします）。そして、 `Service` のバックエンド・セットの１つに対してトラフィックをリダイレクト（出力先を変更）します。
各 `Endpoints` オブジェクトに対して、選択されたバックエンド `ポッド` に対する iptables ルールを投入します。
でforとでは、ランダムがバックエンドとして選択されます。

<!--
Obviously, iptables need not switch back between userspace and kernelspace, it should be
faster and more reliable than the userspace proxy. However, unlike the
userspace proxier, the iptables proxier cannot automatically retry another
`Pod` if the one it initially selects does not respond, so it depends on
having working [readiness probes](/docs/tasks/configure-pod-container/configure-liveness-readiness-probes/#defining-readiness-probes).
-->
言う前でもなく、iptables はユーザ空間（userspace）とカーネル空間（kernelspace）間にスイッチバック（切り替え復帰）が不要でした。
これがユーザ空間プロキシよりも速く信頼できたであろうからです。
しかしながら、ユーザ空間 proxier のように、仮に初めに設定された `ポッド` が応答していなくても、 iptables の proxier は自動的に他の `ポッド` に対してリトライできません。
そのため、[readiness robes](/jp/docs/tasks/configure-pod-container/configure-liveness-readiness-probes/#defining-readiness-probes) の動作に依存します。

![Services overview diagram for iptables proxy](/images/docs/services-iptables-overview.svg)

<!--
Note that in the above diagram, `clusterIP` is shown as `ServiceIP`.
-->
上図で `ServiceIP` （サービスIP）として書かれている部分が `clouserIP` （クラスタIP）ですのでご注意ください。

<!--
### Proxy-mode: ipvs
-->
### プロキシ・モード：ipvs {#proxy-mode:ipvs}

{{< feature-state for_k8s_version="v1.9" state="beta" >}}

<!--
In this mode, kube-proxy watches Kubernetes Services and Endpoints,
calls `netlink` interface to create ipvs rules accordingly and syncs ipvs rules with Kubernetes
Services and Endpoints  periodically, to make sure ipvs status is
consistent with the expectation. When Service is accessed, traffic will
be redirected to one of the backend Pods.
-->
このモードでは、Kube-proxy は Kubernetes サービスとエンドポイントを監視するのに、 `netlink` と呼ぶインターフェースを呼び出します。
このインターフェースで、 ipss  ルールが Kubernetes サービスとエンドポイントの ipvs ルールと同期しているかどうかを定期的に監視し、ipvs の挙動が確実に期待通りになるようにします。
サービスにアクセスがあれば、トラフィックはバックエンド・ポッドの１つに対してリダイレクトします。

<!--
Similar to iptables, Ipvs is based on netfilter hook function, but uses hash
table as the underlying data structure and works in the kernel space.
That means ipvs redirects traffic much faster, and has much
better performance when syncing proxy rules. Furthermore, ipvs provides more
options for load balancing algorithm, such as:
-->
iptablesと同様に、 Ipvs はネットフィルタ・フック機能をベースにしています。
しかし、基本のデータ構造はハッシュ・テーブルを使い、カーネル空間で動作します。
つまり、ipvs はトラフィックをより速くリダイレクトし、プロキシ・ルールの同期も性能が良くなります。
それだけでなく、ipvs は以下の負荷分散アルゴリズムにも対応しています：

<!--
- `rr`: round-robin
- `lc`: least connection
- `dh`: destination hashing
- `sh`: source hashing
- `sed`: shortest expected delay
- `nq`: never queue
-->
- `rr`: ラウンド・ロビン（round-robin）
- `lc`: 最小接続数（least connection）
- `dh`: 送信先ハッシュ（destination hashing）
- `sh`: 送信元ハッシュ（source hashing）
- `sed`:最短遅延（shortest expected delay）
- `nq`: キュー無し（never queue）

<!--
**Note:** ipvs mode assumes IPVS kernel modules are installed on the node
before running kube-proxy. When kube-proxy starts with ipvs proxy mode,
kube-proxy would validate if IPVS modules are installed on the node, if
it's not installed kube-proxy will fall back to iptables proxy mode.
-->
**メモ：** ipvs モードでは、ノードで kube-proxy を実行する前に IPVS カーネル・モジュールがインストールされているのを想定しています。
kube-proxy が ipvs プロキシ・モードで開始すると、 kube-proxy はノード上に IPVS モジュールがインストールされているかどうかを確認します。
もしもインストールされていなければ、iptables プロキシ・モードに後退します。

![Services overview diagram for ipvs proxy](/images/docs/services-ipvs-overview.svg)

<!--
In any of these proxy model, any traffic bound for the Service’s IP:Port is
proxied to an appropriate backend without the clients knowing anything
about Kubernetes or Services or Pods. Client-IP based session affinity
can be selected by setting `service.spec.sessionAffinity` to "ClientIP"
(the default is "None"), and you can set the max session sticky time by
setting the field `service.spec.sessionAffinityConfig.clientIP.timeoutSeconds`
if you have already set `service.spec.sessionAffinity` to "ClientIP"
(the default is “10800”).
-->
これらの各プロキシ・モデルにおいて、あらゆるトラフィックはサービスの IP に対して向けられます。
つまり、クライアントは Kubernetes やサービスやポッドに関する貞節なバックエンドを知らなくても、ポートに対してプロキシされます。
クライアント IP をベースとしたセッション・アフィニティ（session affinity）を選択するには、 `ClientIP` に対する `service.spec.sessionAffinity`  の設定 （デフォルトは "None"）によって作成されます。
そして、  `service.spec.sessionAffinity` で "ClientIP" が指定されていれば、`service.spec.sessionAffinityConfig.clientIP.timeoutSeconds ` フィールドの設定によって最大セッション・スティッキー・タイム（max session sticky time）を指定できます（デフォルトは "10800"）。

<!--
## Multi-Port Services
-->
## マルチ・ポート・サービス {#multi-port-services}

<!--
Many `Services` need to expose more than one port.  For this case, Kubernetes
supports multiple port definitions on a `Service` object.  When using multiple
ports you must give all of your ports names, so that endpoints can be
disambiguated.  For example:
-->
多くの 'Service' は複数のポートを外部に対して公開する必要があります。
そのために、Kubernetes は `Service` オブジェクト上に複数のポート定義をサポートしています。
複数のポートを使うとき、全てのポート名の指定が必要なため、エンドポイントの曖昧さを無くします。以下が例です。



```yaml
kind: Service
apiVersion: v1
metadata:
  name: my-service
spec:
  selector:
    app: MyApp
  ports:
  - name: http
    protocol: TCP
    port: 80
    targetPort: 9376
  - name: https
    protocol: TCP
    port: 443
    targetPort: 9377
```

<!-
Note that the port names must only contain lowercase alphanumeric characters and `-`, and must begin & end with an alphanumeric character. `123-abc` and `web` are valid, but `123_abc` and `-web` are not valid names.
-->
ポート名に含められるのは、小文字の英数字の `-` のみであり、先頭と末尾は英数委ｊの必要があります。
`123-abc` と `web` は有効ですが、 `123_abc` や `-web` は有効な名前ではありません。

<!--
## Choosing your own IP address
-->
## 自分が持つ IP アドレスを選択 {#choosing-your-own-ip-address}

<!--
You can specify your own cluster IP address as part of a `Service` creation
request.  To do this, set the `.spec.clusterIP` field. For example, if you
already have an existing DNS entry that you wish to replace, or legacy systems
that are configured for a specific IP address and difficult to re-configure.
The IP address that a user chooses must be a valid IP address and within the
`service-cluster-ip-range` CIDR range that is specified by flag to the API
server.  If the IP address value is invalid, the apiserver returns a 422 HTTP
status code to indicate that the value is invalid.
-->
`Service` 作成要求（リクエスト）の一部として、自分自身のクラスタ IP アドレスを指定できます。
そのためには、 `.spec.clusterIP` フィールドを設定します。
たとえば、既に既存の DNS エントリを持っており置き換えたいか、レガシー（古い）システムのため IP アドレスの設定や再設定が困難な場合があります。
ユーザが選択する IP アドレスが必ず有効なものである必要があり、API サーバに対するフラグを `service-cluster-ip-range` CIDR 範囲内である必要もあります。
もしも IP アドレスの値が無効であれば、apiserver は 422 HTTP ステータスコードを返し、値が無効だと示します。

<!--
### Why not use round-robin DNS?
-->
### なぜラウンド・ロビン DNS を使わないのですか？ {#why-not-use-round-robin-dns}

<!--
A question that pops up every now and then is why we do all this stuff with
virtual IPs rather than just use standard round-robin DNS.  There are a few
reasons:
-->
標準的なラウンド・ロビン DNS を使うのではなく、起動するたびに仮想 IP を割り当てるのはなぜでしょうか。
これには複数の理由があります。

<!--
   * There is a long history of DNS libraries not respecting DNS TTLs and
     caching the results of name lookups.
   * Many apps do DNS lookups once and cache the results.
   * Even if apps and libraries did proper re-resolution, the load of every
     client re-resolving DNS over and over would be difficult to manage.
-->
    * 名前解決（name lookup）にあたっては、DNS ライブラリが DNS TTL とキャッシュを尊重しない長い歴史があるため
    * 多くのアプリケーションが DNS 名前解決を１度だけ行い、結果をキャッシュをするため
    * アプリケーションとライブラリが適切に再決定しても、比べてのクライアントに対して再反映する管理が大変なため

<!--
We try to discourage users from doing things that hurt themselves.  That said,
if enough people ask for this, we may implement it as an alternative.
-->
このような状態からユーザを失望させないように取り組んでいます。
もしも、この件について多くの人が問い合わせに対応するため、私達は別の実装を試みます。


<!--
## Discovering services
-->
## サービス発見 {#discovering-services}

<!--
Kubernetes supports 2 primary modes of finding a `Service` - environment
variables and DNS.
-->
Kubernetes は `サービス` を見つけるための主なモードが２つあります。
それが、環境変数と DNS です。

<!--
### Environment variables
-->
### 環境変数 {#environment-variables}

<!--
When a `Pod` is run on a `Node`, the kubelet adds a set of environment variables
for each active `Service`.  It supports both [Docker links
compatible](https://docs.docker.com/userguide/dockerlinks/) variables (see
[makeLinkVariables](http://releases.k8s.io/{{< param "githubbranch" >}}/pkg/kubelet/envvars/envvars.go#L49))
and simpler `{SVCNAME}_SERVICE_HOST` and `{SVCNAME}_SERVICE_PORT` variables,
where the Service name is upper-cased and dashes are converted to underscores.
-->
`Node` 上で `ポッド` を実行する時、 kubelet は各アクティブな `サービス` に対して環境変数のセットを追懐ｓｍ差宇。
サポートしているのは、 [Docker link 互換](https://docs.docker.com/userguide/dockerlinks/) （ [makeLinkVariables](http://releases.k8s.io/{{< param "githubbranch" >}}/pkg/kubelet/envvars/envvars.go#L49) をご覧ください）と、シンプルな  `{SVCNAME}_SERVICE_HOST` and `{SVCNAME}_SERVICE_PORT` 変数です。サービス名（SVCNAME）は大文字化され、ダッシュはアンダースコアに変換されます。

<!--
For example, the Service `"redis-master"` which exposes TCP port 6379 and has been
allocated cluster IP address 10.0.0.11 produces the following environment
variables:
-->
例えば「`redis-master`」が TCP ポート 6379 を外部に対して公開し、割り当てられたクラスタ IP アドレスが 10.0.0.11 であれば、以下の環境変数が生成されます。

```shell
REDIS_MASTER_SERVICE_HOST=10.0.0.11
REDIS_MASTER_SERVICE_PORT=6379
REDIS_MASTER_PORT=tcp://10.0.0.11:6379
REDIS_MASTER_PORT_6379_TCP=tcp://10.0.0.11:6379
REDIS_MASTER_PORT_6379_TCP_PROTO=tcp
REDIS_MASTER_PORT_6379_TCP_PORT=6379
REDIS_MASTER_PORT_6379_TCP_ADDR=10.0.0.11
```

<!--
*This does imply an ordering requirement* - any `Service` that a `Pod` wants to
access must be created before the `Pod` itself, or else the environment
variables will not be populated.  DNS does not have this restriction.
->
*暗黙的な順序づけが必要です* 。
あらゆる `サービス` では、 `ポッド` は `ポッド` 自身を作成する前にアクセスしたい場合があっても環境変数が作成されていない場合があるでしょう。
DNS ではこの制約がありません。

### DNS
<!--
An optional (though strongly recommended) [cluster
add-on](/docs/concepts/cluster-administration/addons/) is a DNS server.  The
DNS server watches the Kubernetes API for new `Services` and creates a set of
DNS records for each.  If DNS has been enabled throughout the cluster then all
`Pods` should be able to do name resolution of `Services` automatically.
-->
オプション（ではありますが強く推奨）の [クラスタ・アドオン](/jpdocs/concepts/cluster-administration/addons/) は DNS サーバです。
DNS サーバは Kubernetes API に新しい `サービス` と DNS レコードの集まりが生成されるのをお互いに監視します。
もし DNS がクラスタを通して有効になれば、全ての `ポッド` は自動的に `サービス` で名前解決できるようになります。

<!--
For example, if you have a `Service` called `"my-service"` in Kubernetes
`Namespace` `"my-ns"` a DNS record for `"my-service.my-ns"` is created.  `Pods`
which exist in the `"my-ns"` `Namespace` should be able to find it by simply doing
a name lookup for `"my-service"`.  `Pods` which exist in other `Namespaces` must
qualify the name as `"my-service.my-ns"`.  The result of these name lookups is the
cluster IP.
-->
たとえば、 `"my-service"` という名前の `サービスを` Kubernetes `名前空間` `"my-ns"`に持っていれば、 `"my-service.my-ns"` の DNS レコードを持ちます。
'"my-ns"' `名前空間` にある `ポッド` からは、単純に `"my-service"` の名前で単純に名前解決できます。
他の `名前空間` にある `ポッド`は `"my-service.my-ns"` を満たす必要があります。
これらの名前を使ってクラスタ IP を名前解決します。

<!--
Kubernetes also supports DNS SRV (service) records for named ports.  If the
`"my-service.my-ns"` `Service` has a port named `"http"` with protocol `TCP`, you
can do a DNS SRV query for `"_http._tcp.my-service.my-ns"` to discover the port
number for `"http"`.
-->
また、 Kubernetes は名前を付けたポートに対して DNS SRV（サービス）レコードをサポートします。
`"my-service.my-ns"` `サービス` が `TCP` プロトコルの `"http"` という名前のポートを持つ場合、 "_http._tcp.my-service.my-ns" で `"http"` ポート番号を発見するための DNS SRV 問い合わせ（クエリ）ができます。

<!--
The Kubernetes DNS server is the only way to access services of type
`ExternalName`.  More information is available in the [DNS Pods and Services](/docs/concepts/services-networking/dns-pod-service/).
-->
Kubernetes DNS サーバは `ExternalName` のサービス・タイプに対する唯一のアクセス手段です。
詳しい情報は [DNS ポッドとサービス](/jp/docs/concepts/services-networking/dns-pod-service/) にあります。

<!--
## Headless services
-->
## ヘッドレス・サービス（Headless services） {#headless-services}

<!--
Sometimes you don't need or want load-balancing and a single service IP.  In
this case, you can create "headless" services by specifying `"None"` for the
cluster IP (`.spec.clusterIP`).
-->
場合によっては負荷分散が不要でありサービス IP を１つだけ欲しい場合があるでしょう。
このような場合、クラスタ IP （`.spec.clusterIP`） に '`None`' を指定して、 "headless" サービスを作成できます。

<!--
This option allows developers to reduce coupling to the Kubernetes system by
allowing them freedom to do discovery their own way.  Applications can still use
a self-registration pattern and adapters for other discovery systems could easily
be built upon this API.
-->
このオプションは、開発者が Kubernetes システムに対する結合を減らすもので、各々の手法で自由に発見（ディスカバリ）できるようにします。
アプリケーションは自分で登録したパターンを使えますし、この API 上に構築された他のディスカバリ・システムとも簡単に接続できます。

<!--
For such `Services`, a cluster IP is not allocated, kube-proxy does not handle
these services, and there is no load balancing or proxying done by the platform
for them. How DNS is automatically configured depends on whether the service has
selectors defined.
-->
この `サービス` に対してクラスタ IP が割り振られなければ、kube-prox は各サービスを扱えません。
また、プラットフォームからの負荷分散やプロキシも提供されません。
DNS がどのように自動調整するかは、セレクタで定義したサービスの場所に依存します。
<!--
### With selectors
-->
### セレクタあり {#with-selectors}

<!--
For headless services that define selectors, the endpoints controller creates
`Endpoints` records in the API, and modifies the DNS configuration to return A
records (addresses) that point directly to the `Pods` backing the `Service`.
ｰｰ>
各ヘッドレス・サービスには、API 中にエンドポイント・コントローラが作成した `Endpoints` レコードでセレクタの定義があります。
そして、DNSE 設定が A （アドレス）レコードを返すように調整し、 'ポッド' のバックエンドにある `サービス` を直接示します。

<!--
### Without selectors
-->
### セレクタ無し

<!--
For headless services that do not define selectors, the endpoints controller does
not create `Endpoints` records. However, the DNS system looks for and configures
either:
-->
ヘッドレス・サービスにセレクタの定義が無ければ、エンドポイント・コントローラは `Endpoints` レコードを作成しません。
しかしながら、DNS システムは以下いずれかを探して設定します。

<!--
  * CNAME records for `ExternalName`-type services.
  * A records for any `Endpoints` that share a name with the service, for all
    other types.
-->
   *  `ExternalName`-タイプ・サービスに対する CNAME レコード
   * `Endpoints` に対する A レコードで、他のタイプに対するサービス名を共有する

<!--
## Publishing services - service types
-->
## サービスの公開 - サービス・タイプ

<!--
For some parts of your application (e.g. frontends) you may want to expose a
Service onto an external (outside of your cluster) IP address.
--->
アプリケーションの一部で（例：フロントエンド）、サービスを外部（クラスタ外）の IP アドレスに対して公開（expose）したい場合があるでしょう。

<!--
Kubernetes `ServiceTypes` allow you to specify what kind of service you want.
The default is `ClusterIP`.
-->
Kubernetes `ServiceType（サービス・タイプ）` は、自分が必要なサービスの種類が何かを指定できます。

<!--
`Type` values and their behaviors are:
-->
`Type` の値と挙動は以下の通りです。

<!--
   * `ClusterIP`: Exposes the service on a cluster-internal IP. Choosing this value
     makes the service only reachable from within the cluster. This is the
     default `ServiceType`.
   * `NodePort`: Exposes the service on each Node's IP at a static port (the `NodePort`).
     A `ClusterIP` service, to which the `NodePort` service will route, is automatically
     created.  You'll be able to contact the `NodePort` service, from outside the cluster,
     by requesting `<NodeIP>:<NodePort>`.
   * `LoadBalancer`: Exposes the service externally using a cloud provider's load balancer.
     `NodePort` and `ClusterIP` services, to which the external load balancer will route,
     are automatically created.
   * `ExternalName`: Maps the service to the contents of the `externalName` field
     (e.g. `foo.bar.example.com`), by returning a `CNAME` record with its value.
     No proxying of any kind is set up. This requires version 1.7 or higher of
     `kube-dns`.
-->

* `ClusterIP`：クラスタ内部の IP 上にあるサービスを公開します。ここで値として指定できるのは、クラスタ内で到達可能なサービスのみです。これがデフォルトの `ServiceType` です。
* `NodePort`：各ノード IP 上の固定ポート（`NodePort`）を公開します。 `ClusterIP` サービスは `NodePort` サービスの径路であり、自動的に作成されます。 `NodePort` サービスには、クラスタ外からも `<NodeIP>:<NodePort>` を要求（リクエスト）して接続できます。
* `LoadBalancer`：クラウド・プロバイダのロードバランサを使って、サービスを外部に公開します。 `NodePort` と `ClusterIP` サービスで、外部のロードバランサに径路付け（route）するもので、自動的に作成されます。
* 'ExternalName'： `CNAME` レコードが返す値で `externalName` フィールド（例： `foo.bar.example.com` ）の内容とサービスを割り当てます（マップします）。あらゆる種類のプロキシは設定不要です。使うにはバージョン 1.7 以上の `kube-dns` が必要です。

<!--
### Type NodePort
-->
### タイプ NodePort

<!--
If you set the `type` field to `NodePort`, the Kubernetes master will
allocate a port from a range specified by `--service-node-port-range` flag (default: 30000-32767), and each
Node will proxy that port (the same port number on every Node) into your `Service`.
That port will be reported in your `Service`'s `.spec.ports[*].nodePort` field.
-->
`type` フィールドを `NodePort` に指定すると、 Kubernetes マスタは `--service-node-port-range` フラグで指定した範囲内のポートを割り当てます（デフォルトは 30000 ～ 32767）。
そして、各ノードは `サービス` に対するポートの代理（プロキシ）をします（各ノード上で同じポートです）。
ポートは `サービス` の `.spec.ports[*].nodePort` フィールドで表示されます。

<!--
If you want to specify particular IP(s) to proxy the port, you can set the `--nodeport-addresses` flag in kube-proxy to particular IP block(s) (which is supported since Kubernetes v1.10). A comma-delimited list of IP blocks (e.g. 10.0.0.0/8, 1.2.3.4/32) is used to filter addresses local to this node. For example, if you start kube-proxy with flag `--nodeport-addresses=127.0.0.0/8`, kube-proxy will select only the loopback interface for NodePort Services. The `--nodeport-addresses` is defaulted to empty (`[]`), which means select all available interfaces and is in compliance with current NodePort behaviors.
-->
もしもポートの代理（プロキシ）となる IP アドレスを指定したい場合には、 `--nodeport-addresses` フラグで kube-proxy に対する特定の IP ブロックを指定できます（Kubernetes v1.10 からサポート）。
IP ブロックのカンマ区切りのリスト（例 10.0.0.0/8, 1.2.3.4/32）で、ローカルからアクセスするアドレスのフィルタに使います。
たとば、 kube-proxy に `--nodeport-addresses=127.0.0.0/8` フラグを付けて起動すると、kube-proxy は NodePort サービスに対してループバック・インターフェースのみを指定します。
`--nodeport-addresses` はデフォルトでは空（ `[]` ）です。これは全ての利用可能なインターフェースを選択し、現在の NodePort 挙動に従うものです。

<!--
If you want a specific port number, you can specify a value in the `nodePort`
field, and the system will allocate you that port or else the API transaction
will fail (i.e. you need to take care about possible port collisions yourself).
The value you specify must be in the configured range for node ports.
-->
ポート番号を指定したい場合は、 `nodePort` フィールドの値を指定できます。
そして、システムはポートを割り当てるか、そうでなければ API トランザクションを失敗にします（例：ポート衝突が発生する可能性があり、自分自身で対応する必要があります）。
この値によって、ノード・ポートのための範囲を指定できます。

<!--
This gives developers the freedom to set up their own load balancers, to
configure environments that are not fully supported by Kubernetes, or
even to just expose one or more nodes' IPs directly.
-->
これにより、開発者は環境変数を指定し、Kubernetes に完全にサポートされていなかったり、ノードの IP を直接公開したり、
自身で自由に負荷分散を設定できるようにします。

<!--
Note that this Service will be visible as both `<NodeIP>:spec.ports[*].nodePort`
and `.spec.clusterIP:spec.ports[*].port`. (If the `--nodeport-addresses` flag in kube-proxy is set, <NodeIP> would be filtered NodeIP(s).)
-->

サービスは `<NodeIP>:spec.ports[*].nodePort` と `.spec.clusterIP:spec.ports[*].port`の両方で見えるのでご注意ください。
（もしも kube-proxy で `--nodeport-addresses` フラグが設定されていれば、<NodeIP> は NodeIP でフィルタされます）

<!--
### Type LoadBalancer
-->
### タイプ LoadBalancer {#type-loadbalancer}

<!--
On cloud providers which support external load balancers, setting the `type`
field to `LoadBalancer` will provision a load balancer for your `Service`.
The actual creation of the load balancer happens asynchronously, and
information about the provisioned balancer will be published in the `Service`'s
`.status.loadBalancer` field.  For example:
-->
外部のロードバランサをサポートしているクラウド・プロバイダ上では、 `Service` に対するロードバランサをプロビジョン（自動設定）するために `type` フィールドを `LoadBalancer`  に設定できます。
実際のロード・バランサ作成は非同期であり、プロビジョンされたロードバランサの情報は `Service` の `.status.loadBalancer` フィールにあります。以下は例です。

```yaml
kind: Service
apiVersion: v1
metadata:
  name: my-service
spec:
  selector:
    app: MyApp
  ports:
  - protocol: TCP
    port: 80
    targetPort: 9376
  clusterIP: 10.0.171.239
  loadBalancerIP: 78.11.24.19
  type: LoadBalancer
status:
  loadBalancer:
    ingress:
    - ip: 146.148.47.155
```

<!--
Traffic from the external load balancer will be directed at the backend `Pods`,
though exactly how that works depends on the cloud provider. Some cloud providers allow
the `loadBalancerIP` to be specified. In those cases, the load-balancer will be created
with the user-specified `loadBalancerIP`. If the `loadBalancerIP` field is not specified,
an ephemeral IP will be assigned to the loadBalancer. If the `loadBalancerIP` is specified, but the
cloud provider does not support the feature, the field will be ignored.
-->
外部ロードバランサからのトラフィックはバックエンドの 'ポッド' に対して向けられますが、実際にどのようにするかは用クラウド・プロバイダイの処理に依存します。
あるクラウド・プロバイダでは 'loadBalancerIP' で指定します。
その場合、ロードバランサは `loadBalancerIP` をユーザで指定します。
もし `loadBalancerIP` フィールドの指定がなければ、一時的（ephemeral）な IP が loadBalancer に割り当てられます。
もし `loadBalancerIP` の指定があっても、クラウド・プロバイダが機能をサポートしていなければ、このフィールドは無視されます。

<!--
**Special notes for Azure**: To use user-specified public type `loadBalancerIP`, a static type
public IP address resource needs to be created first, and it should be in the same resource
group of the cluster. Specify the assigned IP address as loadBalancerIP. Verify you have 
securityGroupName in the cloud provider configuration file.
-->
**Azure 向けの特別な注意**：ユーザが指定するパブリック・タイプ `loadBalancerIP` は、作成して最初に固定パブリック IP アドレスのリソースが必要です。そして、クラスタ内の同じ死ソースグループにあるべきです。
loadBalancerIP として IP アドレスを指定します。
クラウド・プロバイダの設定情報ふぁいるにある securityGroupName が存在しているかどうかを確認します。

<!--
#### Internal load balancer
-->
#### 内部ロードバランサ（Internal load balancer）{#internal-load-balancer}

<!--
In a mixed environment it is sometimes necessary to route traffic from services inside the same VPC.
-->
サービスから同じ VPC 内に複数の環境からトラフィックを送る場合があります。

<!--
In a split-horizon DNS environment you would need two services to be able to route both external and internal traffic to your endpoints.
-->
水平分割 DNS 環境では、２つのサービスに対して、外部と内部のトラフィック両方をエンドポイントまで送る必要があります。

<!--
This can be achieved by adding the following annotations to the service based on cloud provider.
-->
実現のためには、以下のアノテーションをクラウド・プロバイダに対応したサービスに追加できます。

{{< tabs name="service_tabs" >}}
{{% tab name="デフォルト" %}}
<!--
Select one of the tabs.
-->
タブのどれか１つを選びます。
{{% /tab %}}
{{% tab name="GCP" %}}
```yaml
[...]
metadata:
    name: my-service
    annotations:
        cloud.google.com/load-balancer-type: "Internal"
[...]
```
<!--
Use `cloud.google.com/load-balancer-type: "internal"` for masters with version 1.7.0 to 1.7.3.
For more information, see the [docs](https://cloud.google.com/kubernetes-engine/docs/internal-load-balancing).
-->
master のバージョン 1.7.0 から 1.7.3 は `cloud.google.com/load-balancer-type: "internal"`  を使います。
詳細は [docs](https://cloud.google.com/kubernetes-engine/docs/internal-load-balancing) をご覧ください。

{{% /tab %}}
{{% tab name="AWS" %}}
```yaml
[...]
metadata:
    name: my-service
    annotations:
        service.beta.kubernetes.io/aws-load-balancer-internal: 0.0.0.0/0
[...]
```
{{% /tab %}}
{{% tab name="Azure" %}}
```yaml
[...]
metadata:
    name: my-service
    annotations:
        service.beta.kubernetes.io/azure-load-balancer-internal: "true"
[...]
```
{{% /tab %}}
{{% tab name="OpenStack" %}}
```yaml
[...]
metadata:
    name: my-service
    annotations:
        service.beta.kubernetes.io/openstack-internal-load-balancer: "true"
[...]
```
{{% /tab %}}
{{< /tabs >}}


#### SSL support on AWS
For partial SSL support on clusters running on AWS, starting with 1.3 three
annotations can be added to a `LoadBalancer` service:

```yaml
metadata:
  name: my-service
  annotations:
    service.beta.kubernetes.io/aws-load-balancer-ssl-cert: arn:aws:acm:us-east-1:123456789012:certificate/12345678-1234-1234-1234-123456789012
```

The first specifies the ARN of the certificate to use. It can be either a
certificate from a third party issuer that was uploaded to IAM or one created
within AWS Certificate Manager.

```yaml
metadata:
  name: my-service
  annotations:
    service.beta.kubernetes.io/aws-load-balancer-backend-protocol: (https|http|ssl|tcp)
```

The second annotation specifies which protocol a pod speaks. For HTTPS and
SSL, the ELB will expect the pod to authenticate itself over the encrypted
connection.

HTTP and HTTPS will select layer 7 proxying: the ELB will terminate
the connection with the user, parse headers and inject the `X-Forwarded-For`
header with the user's IP address (pods will only see the IP address of the
ELB at the other end of its connection) when forwarding requests.

TCP and SSL will select layer 4 proxying: the ELB will forward traffic without
modifying the headers.

In a mixed-use environment where some ports are secured and others are left unencrypted,
the following annotations may be used:

```yaml
    metadata:
      name: my-service
      annotations:
        service.beta.kubernetes.io/aws-load-balancer-backend-protocol: http
        service.beta.kubernetes.io/aws-load-balancer-ssl-ports: "443,8443"
```

In the above example, if the service contained three ports, `80`, `443`, and
`8443`, then `443` and `8443` would use the SSL certificate, but `80` would just
be proxied HTTP.

Beginning in 1.9, services can use [predefined AWS SSL policies](http://docs.aws.amazon.com/elasticloadbalancing/latest/classic/elb-security-policy-table.html)
for any HTTPS or SSL listeners. To see which policies are available for use, run
the awscli command:

```bash
aws elb describe-load-balancer-policies --query 'PolicyDescriptions[].PolicyName'
```

Any one of those policies can then be specified using the
"`service.beta.kubernetes.io/aws-load-balancer-ssl-negotiation-policy`"
annotation, for example:

```yaml
    metadata:
      name: my-service
      annotations:
        service.beta.kubernetes.io/aws-load-balancer-ssl-negotiation-policy: "ELBSecurityPolicy-TLS-1-2-2017-01"
```

#### PROXY protocol support on AWS

To enable [PROXY protocol](https://www.haproxy.org/download/1.8/doc/proxy-protocol.txt)
support for clusters running on AWS, you can use the following service
annotation:

```yaml
    metadata:
      name: my-service
      annotations:
        service.beta.kubernetes.io/aws-load-balancer-proxy-protocol: "*"
```

Since version 1.3.0 the use of this annotation applies to all ports proxied by the ELB
and cannot be configured otherwise.

#### ELB Access Logs on AWS

There are several annotations to manage access logs for ELB services on AWS.

The annotation `service.beta.kubernetes.io/aws-load-balancer-access-log-enabled`
controls whether access logs are enabled.

The annotation `service.beta.kubernetes.io/aws-load-balancer-access-log-emit-interval`
controls the interval in minutes for publishing the access logs. You can specify
an interval of either 5 or 60.

The annotation `service.beta.kubernetes.io/aws-load-balancer-access-log-s3-bucket-name`
controls the name of the Amazon S3 bucket where load balancer access logs are
stored.

The annotation `service.beta.kubernetes.io/aws-load-balancer-access-log-s3-bucket-prefix`
specifies the logical hierarchy you created for your Amazon S3 bucket.

```yaml
    metadata:
      name: my-service
      annotations:
        service.beta.kubernetes.io/aws-load-balancer-access-log-enabled: "true"
        # Specifies whether access logs are enabled for the load balancer
        service.beta.kubernetes.io/aws-load-balancer-access-log-emit-interval: "60"
        # The interval for publishing the access logs. You can specify an interval of either 5 or 60 (minutes).
        service.beta.kubernetes.io/aws-load-balancer-access-log-s3-bucket-name: "my-bucket"
        # The name of the Amazon S3 bucket where the access logs are stored
        service.beta.kubernetes.io/aws-load-balancer-access-log-s3-bucket-prefix: "my-bucket-prefix/prod"
        # The logical hierarchy you created for your Amazon S3 bucket, for example `my-bucket-prefix/prod`
```

#### Connection Draining on AWS

Connection draining for Classic ELBs can be managed with the annotation
`service.beta.kubernetes.io/aws-load-balancer-connection-draining-enabled` set
to the value of `"true"`. The annotation
`service.beta.kubernetes.io/aws-load-balancer-connection-draining-timeout` can
also be used to set maximum time, in seconds, to keep the existing connections open before deregistering the instances.


```yaml
    metadata:
      name: my-service
      annotations:
        service.beta.kubernetes.io/aws-load-balancer-connection-draining-enabled: "true"
        service.beta.kubernetes.io/aws-load-balancer-connection-draining-timeout: "60"
```

#### Other ELB annotations

There are other annotations to manage Classic Elastic Load Balancers that are described below.

```yaml
    metadata:
      name: my-service
      annotations:
        service.beta.kubernetes.io/aws-load-balancer-connection-idle-timeout: "60"
        # The time, in seconds, that the connection is allowed to be idle (no data has been sent over the connection) before it is closed by the load balancer

        service.beta.kubernetes.io/aws-load-balancer-cross-zone-load-balancing-enabled: "true"
        # Specifies whether cross-zone load balancing is enabled for the load balancer

        service.beta.kubernetes.io/aws-load-balancer-additional-resource-tags: "environment=prod,owner=devops"
        # A comma-separated list of key-value pairs which will be recorded as
        # additional tags in the ELB.

        service.beta.kubernetes.io/aws-load-balancer-healthcheck-healthy-threshold: ""
        # The number of successive successful health checks required for a backend to
        # be considered healthy for traffic. Defaults to 2, must be between 2 and 10

        service.beta.kubernetes.io/aws-load-balancer-healthcheck-unhealthy-threshold: "3"
        # The number of unsuccessful health checks required for a backend to be
        # considered unhealthy for traffic. Defaults to 6, must be between 2 and 10

        service.beta.kubernetes.io/aws-load-balancer-healthcheck-interval: "20"
        # The approximate interval, in seconds, between health checks of an
        # individual instance. Defaults to 10, must be between 5 and 300
        service.beta.kubernetes.io/aws-load-balancer-healthcheck-timeout: "5"
        # The amount of time, in seconds, during which no response means a failed
        # health check. This value must be less than the service.beta.kubernetes.io/aws-load-balancer-healthcheck-interval
        # value. Defaults to 5, must be between 2 and 60

        service.beta.kubernetes.io/aws-load-balancer-extra-security-groups: "sg-53fae93f,sg-42efd82e"
        # A list of additional security groups to be added to ELB
```

#### Network Load Balancer support on AWS [alpha]

**Warning:** This is an alpha feature and not recommended for production clusters yet.

Starting in version 1.9.0, Kubernetes supports Network Load Balancer (NLB). To
use a Network Load Balancer on AWS, use the annotation `service.beta.kubernetes.io/aws-load-balancer-type`
with the value set to `nlb`.

```yaml
    metadata:
      name: my-service
      annotations:
        service.beta.kubernetes.io/aws-load-balancer-type: "nlb"
```

Unlike Classic Elastic Load Balancers, Network Load Balancers (NLBs) forward the
client's IP through to the node. If a service's `.spec.externalTrafficPolicy` is
set to `Cluster`, the client's IP address will not be propagated to the end
pods.

By setting `.spec.externalTrafficPolicy` to `Local`, client IP addresses will be
propagated to the end pods, but this could result in uneven distribution of
traffic. Nodes without any pods for a particular LoadBalancer service will fail
the NLB Target Group's health check on the auto-assigned
`.spec.healthCheckNodePort` and not receive any traffic.

In order to achieve even traffic, either use a DaemonSet, or specify a
[pod anti-affinity](/docs/concepts/configuration/assign-pod-node/#inter-pod-affinity-and-anti-affinity-beta-feature)
to not locate pods on the same node.

NLB can also be used with the [internal load balancer](/docs/concepts/services-networking/service/#internal-load-balancer)
annotation.

In order for client traffic to reach instances behind an NLB, the Node security
groups are modified with the following IP rules:

| Rule | Protocol | Port(s) | IpRange(s) | IpRange Description |
|------|----------|---------|------------|---------------------|
| Health Check | TCP | NodePort(s) (`.spec.healthCheckNodePort` for `.spec.externalTrafficPolicy = Local`) | VPC CIDR | kubernetes.io/rule/nlb/health=\<loadBalancerName\> |
| Client Traffic | TCP | NodePort(s) | `.spec.loadBalancerSourceRanges` (defaults to `0.0.0.0/0`) | kubernetes.io/rule/nlb/client=\<loadBalancerName\> |
| MTU Discovery | ICMP | 3,4 | `.spec.loadBalancerSourceRanges` (defaults to `0.0.0.0/0`) | kubernetes.io/rule/nlb/mtu=\<loadBalancerName\> |

Be aware that if `.spec.loadBalancerSourceRanges` is not set, Kubernetes will
allow traffic from `0.0.0.0/0` to the Node Security Group(s). If nodes have
public IP addresses, be aware that non-NLB traffic can also reach all instances
in those modified security groups.

In order to limit which client IP's can access the Network Load Balancer,
specify `loadBalancerSourceRanges`.

```yaml
spec:
  loadBalancerSourceRanges:
  - "143.231.0.0/16"
```

**Note:** NLB only works with certain instance classes, see the [AWS documentation](http://docs.aws.amazon.com/elasticloadbalancing/latest/network/target-group-register-targets.html#register-deregister-targets)
for supported instance types.

<!--
### External IPs
-->
### 外部 IP（External IP） {#external-ip}

<!--
If there are external IPs that route to one or more cluster nodes, Kubernetes services can be exposed on those
`externalIPs`. Traffic that ingresses into the cluster with the external IP (as destination IP), on the service port,
will be routed to one of the service endpoints. `externalIPs` are not managed by Kubernetes and are the responsibility
of the cluster administrator.
-->
１つまたは複数のクラスタ・ノードに送る（route）ための外部 IP（external IP）があれば、Kubernetes サービスは各 `externalIP` に対してサービスを公開します。
外部の IP からクラスタの ingress（訳者注：内部ネットワーク）に対するトラフィックは、サービスのポートに対するもので、１つまたは複数のエンドポイントに対して送られます。
`externalIP` は Kubernetes によって管理されず、クラスタ・管理者の責任となります。

<!--
In the `ServiceSpec`, `externalIPs` can be specified along with any of the `ServiceTypes`.
In the example below, "`my-service`" can be accessed by clients on "`80.11.12.10:80`"" (`externalIP:port`)
-->
`ServiceSpec` 内では、 `externalIP` にはあらゆる `ServiceTypes`を指定できます。
以下の例では "`my-service`" は "'80.11.12.10:80'" （`externalIP:port`）に関連付けられます。

```yaml
kind: Service
apiVersion: v1
metadata:
  name: my-service
spec:
  selector:
    app: MyApp
  ports:
  - name: http
    protocol: TCP
    port: 80
    targetPort: 9376
  externalIPs:
  - 80.11.12.10
```

<!--
## Shortcomings
-->
## 欠点 {#shortcomings}

<!--
Using the userspace proxy for VIPs will work at small to medium scale, but will
not scale to very large clusters with thousands of Services.  See [the original
design proposal for portals](http://issue.k8s.io/1107) for more details.
-->
VIP に対して名前空間プロキシを使うのは、小規模から中規模のスケールで機能します。
しかし、何千ものサービスがある大きなクラスタにはスケールできません。
詳細は [ポータルにあるオリジナルの設計提案](http://issue.k8s.io/1107) をご覧ください。

<!--
Using the userspace proxy obscures the source-IP of a packet accessing a `Service`.
This makes some kinds of firewalling impossible.  The iptables proxier does not
obscure in-cluster source IPs, but it does still impact clients coming through
a load-balancer or node-port.
-->
ユーザ空間プロキシを使うと、`Service` に接続するパケットの送信元 IP が不明瞭になります。
これによってある種のファイアウォールが使えなくなります。
iptables はクラスタ内の接続も IP が不明瞭になりますが、ロードバランサやノードのポートに対しても影響があります。

<!--
The `Type` field is designed as nested functionality - each level adds to the
previous.  This is not strictly required on all cloud providers (e.g. Google Compute Engine does
not need to allocate a `NodePort` to make `LoadBalancer` work, but AWS does)
but the current API requires it.
-->
'Type' フィールドはネストした（入れ子になった）機能性のために設計されています。
正確には全てのクラウド・プロバイダで必要ではありません（例：Google Compute Engine は `LoadBalancer` を動かすのに `NodePort` は不要ですが、AWS では必要です）が、現在の API では必要となります。

<!--
## Future work
-->
## 今後の対応 {#future-work}

<!--
In the future we envision that the proxy policy can become more nuanced than
simple round robin balancing, for example master-elected or sharded.  We also
envision that some `Services` will have "real" load balancers, in which case the
VIP will simply transport the packets there.
-->
将来的なビジョンとしては、プロキシ方針（ポリシー）によって、シンプルな負荷分散なではなく、より細かな制御を視野に入れています。
たとえば、マスタの選出やシャード化（sharded）です。
また、 `Services` が実際のロードバランサになるのも想定しています。
この場合、VIP は単純にパケットを転送するだけです。

<!--
We intend to improve our support for L7 (HTTP) `Services`.
-->
私達は L7 (HTTP) `Service`  をサポートするように改良するつもりです。

<!--
We intend to have more flexible ingress modes for `Services` which encompass
the current `ClusterIP`, `NodePort`, and `LoadBalancer` modes and more.
-->
私達は `Service` に対するイングレス・モードをより柔軟にするつもりです。これは現在の `ClusterIP` 、 `NodePort` 、 `LoadBalaner`  モードなどを包括します。

<!--
## The gory details of virtual IPs
-->
## 仮想 IP の詳しい説明 {#the-gory-deta}

The previous information should be sufficient for many people who just want to
use `Services`.  However, there is a lot going on behind the scenes that may be
worth understanding.

### Avoiding collisions

One of the primary philosophies of Kubernetes is that users should not be
exposed to situations that could cause their actions to fail through no fault
of their own.  In this situation, we are looking at network ports - users
should not have to choose a port number if that choice might collide with
another user.  That is an isolation failure.

In order to allow users to choose a port number for their `Services`, we must
ensure that no two `Services` can collide.  We do that by allocating each
`Service` its own IP address.

To ensure each service receives a unique IP, an internal allocator atomically
updates a global allocation map in etcd prior to creating each service. The map object
must exist in the registry for services to get IPs, otherwise creations will
fail with a message indicating an IP could not be allocated. A background
controller is responsible for creating that map (to migrate from older versions
of Kubernetes that used in memory locking) as well as checking for invalid
assignments due to administrator intervention and cleaning up any IPs
that were allocated but which no service currently uses.

### IPs and VIPs

Unlike `Pod` IP addresses, which actually route to a fixed destination,
`Service` IPs are not actually answered by a single host.  Instead, we use
`iptables` (packet processing logic in Linux) to define virtual IP addresses
which are transparently redirected as needed.  When clients connect to the
VIP, their traffic is automatically transported to an appropriate endpoint.
The environment variables and DNS for `Services` are actually populated in
terms of the `Service`'s VIP and port.

We support three proxy modes - userspace, iptables and ipvs which operate
slightly differently.

#### Userspace

As an example, consider the image processing application described above.
When the backend `Service` is created, the Kubernetes master assigns a virtual
IP address, for example 10.0.0.1.  Assuming the `Service` port is 1234, the
`Service` is observed by all of the `kube-proxy` instances in the cluster.
When a proxy sees a new `Service`, it opens a new random port, establishes an
iptables redirect from the VIP to this new port, and starts accepting
connections on it.

When a client connects to the VIP the iptables rule kicks in, and redirects
the packets to the `Service proxy`'s own port.  The `Service proxy` chooses a
backend, and starts proxying traffic from the client to the backend.

This means that `Service` owners can choose any port they want without risk of
collision.  Clients can simply connect to an IP and port, without being aware
of which `Pods` they are actually accessing.

#### Iptables

Again, consider the image processing application described above.
When the backend `Service` is created, the Kubernetes master assigns a virtual
IP address, for example 10.0.0.1.  Assuming the `Service` port is 1234, the
`Service` is observed by all of the `kube-proxy` instances in the cluster.
When a proxy sees a new `Service`, it installs a series of iptables rules which
redirect from the VIP to per-`Service` rules.  The per-`Service` rules link to
per-`Endpoint` rules which redirect (Destination NAT) to the backends.

When a client connects to the VIP the iptables rule kicks in.  A backend is
chosen (either based on session affinity or randomly) and packets are
redirected to the backend.  Unlike the userspace proxy, packets are never
copied to userspace, the kube-proxy does not have to be running for the VIP to
work, and the client IP is not altered.

This same basic flow executes when traffic comes in through a node-port or
through a load-balancer, though in those cases the client IP does get altered.

#### Ipvs

Iptables operations slow down dramatically in large scale cluster e.g 10,000 Services. IPVS is designed for load balancing and based on in-kernel hash tables. So we can achieve performance consistency in large number of services from IPVS-based kube-proxy. Meanwhile, IPVS-based kube-proxy has more sophisticated load balancing algorithms (least conns, locality, weighted, persistence).

<!--
## API Object
-->
## API オブジェクト {#api-object}

<!--
Service is a top-level resource in the Kubernetes REST API. More details about the
API object can be found at:
[Service API object](/docs/reference/generated/kubernetes-api/{{< param "version" >}}/#service-v1-core).
-->
サービスとは Kubernetes REST API のなかでトップ・レベルのリソースです。
API オブジェクトに関する詳しい情報は [サービス API オブジェクト](/docs/reference/generated/kubernetes-api/{{< param "version" >}}/#service-v1-core) にあります。

{{% /capture %}}

{{% capture whatsnext %}}

Read [Connecting a Front End to a Back End Using a Service](/docs/tasks/access-application-cluster/connecting-frontend-backend/).

{{% /capture %}}

