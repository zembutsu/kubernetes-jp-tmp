---
reviewers:
- enisoc
- erictune
- foxish
- janetkuo
- kow3ns
- smarterclayton
title: StatefulSets（ステートフル・セット）
content_template: templates/concept
weight: 40
---

{{% capture overview %}}
<!--
StatefulSet is the workload API object used to manage stateful applications.
-->
StatefulSet はステートフルなアプリケーションを管理するために使うワークロード API オブジェクトです。

{{< note >}}
<!--
**Note:** StatefulSets are stable (GA) in 1.9.
-->
**メモ：** StatefulSets はバージョン 1.9 から安定版（GA）です。
{{< /note >}}

{{< glossary_definition term_id="statefulset" length="all" >}}
{{% /capture %}}

{{% capture body %}}

<!--
## Using StatefulSets
-->
## StatefulSet を使う {#using-statefulsets}

<!--
StatefulSets are valuable for applications that require one or more of the
following.
-->
StatefulSet は以下の１つもしくは複数の要件があるアプリケーションに有益です。

<!--
* Stable, unique network identifiers.
* Stable, persistent storage.
* Ordered, graceful deployment and scaling.
* Ordered, graceful deletion and termination.
* Ordered, automated rolling updates.
-->
* 安定した、ユニークなネットワーク識別子
* 安定した、持続的ストレージ（保管領域）
* 手入れの行き届いた、丁寧なデプロイメント（展開）とスケーリング（規模変更）
* 手入れの行き届いた、丁寧な削除と停止
* 手入れの行き届いた、自動化ローリング・アップデート（逐次更新）

<!--
In the above, stable is synonymous with persistence across Pod (re)scheduling.
If an application doesn't require any stable identifiers or ordered deployment,
deletion, or scaling, you should deploy your application with a controller that
provides a set of stateless replicas. Controllers such as
[Deployment](/docs/concepts/workloads/controllers/deployment/) or
[ReplicaSet](/docs/concepts/workloads/controllers/replicaset/) may be better suited to your stateless needs.
-->
前述のとおり、ポッドの（再）スケーリングにおいて、安定性と持続性（persistence）とは同義語です。
もしも、アプリケーションが安定した識別子、規則正しいデプロイメント（展開）、削除、スケーリング（規模変更）を必要としないのであれば、
アプリケーションのデプロイはコントローラを使うべきでしょう。
コントローラはステートレス（状態を持たない）な複製（レプリカ）の集まりを提供します。
ステートレス（状態を持たない）が必要であれば、[Deployment（デプロイメント）](/jp/docs/concepts/workloads/controllers/deployment/) や [ReplicaSet（レプリカ・セット）](/jp/docs/concepts/workloads/controllers/replicaset/) のようなコントローラのほうが適しているでしょう。

<!--
## Limitations
-->
## 制限事項 {#Limitations}

<!--
* StatefulSet was a beta resource prior to 1.9 and not available in any Kubernetes release prior to 1.5.
* As with all alpha/beta resources, you can disable StatefulSet through the `--runtime-config` option passed to the apiserver.
* The storage for a given Pod must either be provisioned by a [PersistentVolume Provisioner](https://github.com/kubernetes/examples/tree/{{< param "githubbranch" >}}/staging/persistent-volume-provisioning/README.md) based on the requested `storage class`, or pre-provisioned by an admin.
* Deleting and/or scaling a StatefulSet down will *not* delete the volumes associated with the StatefulSet. This is done to ensure data safety, which is generally more valuable than an automatic purge of all related StatefulSet resources.
* StatefulSets currently require a [Headless Service](/docs/concepts/services-networking/service/#headless-services) to be responsible for the network identity of the Pods. You are responsible for creating this Service.
-->
* StatefulSet は 1.9 未満ではベータ・リソースであり、1.5 未満の Kubernetes では利用できません。
* 全てのアルファ/ベータのリソースと同様に、apiserver に渡すオプションで `--runtime-config` を指定すると、StatefulSet を無効化できます。
* ポッドに対して与えられるストレージは、 `storage class` の要求に基づく  [PersistentVolume Provisioner](https://github.com/kubernetes/examples/tree/{{< param "githubbranch" >}}/staging/persistent-volume-provisioning/README.md) によって供給（プロビジョン）されるか、管理者によって予め供給（プロビジョニング）済みのどちらの必要があります。
* StatefulSet の削除やスケーリング・ダウンでは、StatefulSet に関連付けられたボリュームを削除*しません* 。これはデータの安全性を確保するために行うものであり、StatefulSet リソースに関連する全てのリソースを自動的に切り離すよりも重要です。
* StatefulSet は、ポッドをネットワーク上で識別する役割ために現時点で [Headless Service](/jp/docs/concepts/services-networking/service/#headless-services) が必要です。あなたはサービスの作成に責任を持つ必要があります。

<!--
## Components
-->
## 構成要素（コンポーネント） {#components}

<!--
The example below demonstrates the components of a StatefulSet.
-->
以下の例は StatefulSet の構成要素を示します。

<!--
* A Headless Service, named nginx, is used to control the network domain.
* The StatefulSet, named web, has a Spec that indicates that 3 replicas of the nginx container will be launched in unique Pods.
* The volumeClaimTemplates will provide stable storage using [PersistentVolumes](/docs/concepts/storage/persistent-volumes/) provisioned by a PersistentVolume Provisioner.
-->
* nginx という名前の Headless Service（ヘッドレス・サービス） を、ネットワーク・ドメインの管理に使う
* web という名前の StatefulSet は、nginx コンテナの複製を３つを、それぞれユニークなポッドとして起動するSpec（仕様）があります。
* volumeClaimTemplates は PersistentVolume プロビジョナ（供給機能）による [PersistentVolumes（持続ボリューム）](/jp/docs/concepts/storage/persistent-volumes/)  を使う安定したストレージ（記憶領域）を提供します。

```yaml
apiVersion: v1
kind: Service
metadata:
  name: nginx
  labels:
    app: nginx
spec:
  ports:
  - port: 80
    name: web
  clusterIP: None
  selector:
    app: nginx
---
apiVersion: apps/v1
kind: StatefulSet
metadata:
  name: web
spec:
  selector:
    matchLabels:
      app: nginx # has to match .spec.template.metadata.labels
  serviceName: "nginx"
  replicas: 3 # by default is 1
  template:
    metadata:
      labels:
        app: nginx # has to match .spec.selector.matchLabels
    spec:
      terminationGracePeriodSeconds: 10
      containers:
      - name: nginx
        image: k8s.gcr.io/nginx-slim:0.8
        ports:
        - containerPort: 80
          name: web
        volumeMounts:
        - name: www
          mountPath: /usr/share/nginx/html
  volumeClaimTemplates:
  - metadata:
      name: www
    spec:
      accessModes: [ "ReadWriteOnce" ]
      storageClassName: "my-storage-class"
      resources:
        requests:
          storage: 1Gi
```

<!--
## Pod Selector
-->
## ポッド・セレクタ（Pod Selector）{#pod-selector}

<!--
You must set the `.spec.selector` field of a StatefulSet to match the labels of its `.spec.template.metadata.labels`. Prior to Kubernetes 1.8, the `.spec.selector` field was defaulted when omitted. In 1.8 and later versions, failing to specify a matching Pod Selector will result in a validation error during StatefulSet creation.
-->
StatefulSet の `.spec.selector` フィールドは、自身の `.spec.template.metadata.labels` ラベルと一致するように指定が必要です。
Kubernetes 1.8 未満では、 `.spec.selector` フィールドの省略はデフォルトでした。1.8 以降のバージョンでは、ポッド・セレクタの指定と一致しなければ、StatefulSet 作成時に整合性エラー（validation error）が起こり、作成に失敗します。

<!--
## Pod Identity
-->
## ポッド同一性（Pod Identity） {#pod-identity}

<!--
StatefulSet Pods have a unique identity that is comprised of an ordinal, a
stable network identity, and stable storage. The identity sticks to the Pod,
regardless of which node it's (re)scheduled on.
-->
StatefulSet のポッドはユニークな同一性（unique identity）があります。これを構成するのは順序、安定したネットワーク同一性、安定したストレージです。
識別子はポッドに張り付いているもので、どのノードに（再）スケジュールされても変わりません。

<!--
### Ordinal Index
-->
### オリジナル（原型）インデックス（Original Index） {#original-index}

<!--
For a StatefulSet with N replicas, each Pod in the StatefulSet will be
assigned an integer ordinal, from 0 up through N-1, that is unique over the Set.
-->
StatefulSet に N の複製があるとすると、StatefulSet 内の各ポッドにはオリジナル（原型）としての整数が、0 から N-1 まで割り当てられます。これはセットを超えてユニークです。

<!--
### Stable Network ID
-->
### 安定したネットワーク ID （Stable Network ID） {#stable-network-id}

<!--
Each Pod in a StatefulSet derives its hostname from the name of the StatefulSet
and the ordinal of the Pod. The pattern for the constructed hostname
is `$(statefulset name)-$(ordinal)`. The example above will create three Pods
named `web-0,web-1,web-2`.
A StatefulSet can use a [Headless Service](/docs/concepts/services-networking/service/#headless-services)
to control the domain of its Pods. The domain managed by this Service takes the form:
`$(service name).$(namespace).svc.cluster.local`, where "cluster.local" is the
cluster domain.
As each Pod is created, it gets a matching DNS subdomain, taking the form:
`$(podname).$(governing service domain)`, where the governing service is defined
by the `serviceName` field on the StatefulSet.
-->
StatefulSet 内の各ポッドに提供するホスト名（hostname）は、StatefulSet の名前からと、ポッドのオリジナル（原型）からです。
作成されるホスト名のパターンは `$(statefulsetの名前)-$(オリジナル)` です。
戦術の例では、３つのポッドを作成しました。名前は `web-0、web-1、web-2` です。
StatefulSet は [ヘッドレス・サービス（Headless Service）](/jp/docs/concepts/services-networking/service/#headless-services) を使ってポッドのドメインを制御します。
このサービスによって管理されるドメインが取る形式は、 `$(サービス名).$(名前空間).svc.cluster.local` です。
「cluster.local」はクラスタンおドメインです。
各ポッドが作成されると、ここには DNS サブドメインに一致する名前をとり、形式は `$(ポッド名).$(管理対象のサービス・ドメイン)` となります。管理対象のサービスとは、StatefulSet の `serviceName` （サービス名）フィールドで定義されていｍさう。

<!--
Here are some examples of choices for Cluster Domain, Service name,
StatefulSet name, and how that affects the DNS names for the StatefulSet's Pods.
-->
こちらにあるのは、クラスタ・ドメイン、サービス名、StatefulSet 名を選んだ例です。そして、StatefulSet のポッドに対して、DNS 名がどのような影響を与えているかです。

<!--
Cluster Domain | Service (ns/name) | StatefulSet (ns/name)  | StatefulSet Domain  | Pod DNS | Pod Hostname |
-------------- | ----------------- | ----------------- | -------------- | ------- | ------------ |
 cluster.local | default/nginx     | default/web       | nginx.default.svc.cluster.local | web-{0..N-1}.nginx.default.svc.cluster.local | web-{0..N-1} |
 cluster.local | foo/nginx         | foo/web           | nginx.foo.svc.cluster.local     | web-{0..N-1}.nginx.foo.svc.cluster.local     | web-{0..N-1} |
 kube.local    | foo/nginx         | foo/web           | nginx.foo.svc.kube.local        | web-{0..N-1}.nginx.foo.svc.kube.local        | web-{0..N-1} |
-->

クラスタ/ドメイン | サービス (名前空間/名前) | StatefulSet (名前空間/名前)  | StatefulSet ドメイン  | ポッド DNS | ポッド ホスト名 |
-------------- | ----------------- | ----------------- | -------------- | ------- | ------------ |
 cluster.local | default/nginx     | default/web       | nginx.default.svc.cluster.local | web-{0..N-1}.nginx.default.svc.cluster.local | web-{0..N-1} |
 cluster.local | foo/nginx         | foo/web           | nginx.foo.svc.cluster.local     | web-{0..N-1}.nginx.foo.svc.cluster.local     | web-{0..N-1} |
 kube.local    | foo/nginx         | foo/web           | nginx.foo.svc.kube.local        | web-{0..N-1}.nginx.foo.svc.kube.local        | web-{0..N-1} |

<!--
Note that Cluster Domain will be set to `cluster.local` unless
[otherwise configured](/docs/concepts/services-networking/dns-pod-service/#how-it-works).
-->
クラスタ・ドメインが [設定されていなければ](/jp/docs/concepts/services-networking/dns-pod-service/#how-it-works)  `cluster.local` に設定されるのでご注意ください。

<!--
### Stable Storage
-->
### 安定したストレージ（Stable Storage） {#stable-storage}

<!--
Kubernetes creates one [PersistentVolume](/docs/concepts/storage/persistent-volumes/) for each
VolumeClaimTemplate. In the nginx example above, each Pod will receive a single PersistentVolume
with a StorageClass of `my-storage-class` and 1 Gib of provisioned storage. If no StorageClass
is specified, then the default StorageClass will be used. When a Pod is (re)scheduled
onto a node, its `volumeMounts` mount the PersistentVolumes associated with its
PersistentVolume Claims. Note that, the PersistentVolumes associated with the
Pods' PersistentVolume Claims are not deleted when the Pods, or StatefulSet are deleted.
This must be done manually.
-->
Kubernetes は、それぞれの VolumeClaimTemplate （ボリューム要求テンプレート）に対して、１つの [PersistentVolume（持続型ボリューム）](/jp/docs/concepts/storage/persistent-volumes/) を割り当てます。
先述の nginx の例では、各ポッドは１つの PersistentVolume に `my-storage-class` の StorageClass（ストレージ・クラス）を与えられ、1Gib の容量が供給されます。
もし StorageClass を指定しなければ、デフォルトの StorageClass が使われます。
ポッドがノード内に（再）スケジュールされた時、 `volumeMounts` は PersistentVolume Claim（持続ボリューム要求）に関連付けられた PersistentVolume（持続ボリューム）をマウントします。
注意するのは、PersistentVolume（持続ボリューム）が関連付けられているのはポットに対する PersistentVolume Claim（持続ボリューム要求）であり、これはポッドや StatefulSet が削除されても削除されません。
削除をするには手動で行う必要があります。

<!--
### Pod Name Label
-->
### ポッド名ラベル（Pod Name Label）{#pod-name-label}

<!--
When the StatefulSet controller creates a Pod, it adds a label, `statefulset.kubernetes.io/pod-name`, 
that is set to the name of the Pod. This label allows you to attach a Service to a specific Pod in 
the StatefulSet.
-->
StatefulSet コントローラがポッドを作成すると、 `statefulset.kubernetes.io/pod-name` ラベルを追加します。
これはポッドの名前を追加します。
このラベルによって、StatefulSet が特定のポッドに対してサービス割り当て（アタッチする）可能になります。

<!--
## Deployment and Scaling Guarantees
-->
## デプロイメントとスケーリング保証 {#deployment-and-scaling-guarantees

<!--
* For a StatefulSet with N replicas, when Pods are being deployed, they are created sequentially, in order from {0..N-1}.
* When Pods are being deleted, they are terminated in reverse order, from {N-1..0}.
* Before a scaling operation is applied to a Pod, all of its predecessors must be Running and Ready.
* Before a Pod is terminated, all of its successors must be completely shutdown.
-->
* StatefulSet に N 複製（レプリカ）があり、ポッドがデプロイされると、 {0..N-1} まで順番に連続して作成される
* ポッドの削除が始まると、削除は逆順 {N-1..0} で削除される
* スケーリング作業がポッドに適用される前に、全ての先行作業（predecessor）が実行され、準備が整っている必要がある
* ポッドが削除される前に、その後継者（successor）が完全にシャットダウンしている必要がある

<!--
The StatefulSet should not specify a `pod.Spec.TerminationGracePeriodSeconds` of 0. This practice is unsafe and strongly discouraged. For further explanation, please refer to [force deleting StatefulSet Pods](/docs/tasks/run-application/force-delete-stateful-set-pod/).
-->
StetefulSet では `pod.Spec.TerminationGracePeriodSeconds` を 0 に指定すべきではありません。
この実行は安全ではなく、とても思い留めるべきです。
詳しい説明については  [StatefulSet ポッドの強制削除](/jp/docs/tasks/run-application/force-delete-stateful-set-pod/) を参照ください。

<!--
When the nginx example above is created, three Pods will be deployed in the order
web-0, web-1, web-2. web-1 will not be deployed before web-0 is
[Running and Ready](/docs/user-guide/pod-states/), and web-2 will not be deployed until
web-1 is Running and Ready. If web-0 should fail, after web-1 is Running and Ready, but before
web-2 is launched, web-2 will not be launched until web-0 is successfully relaunched and
becomes Running and Ready.
-->
先ほど作成した nginx の例では、３つのポッドが web-0、web-1、web-2 の順番で展開（デプロイ）されます。
web-1 は web-0 が [実行中かつ待機](/jp/docs/user-guide/pod-states/) にならないと展開しまｓねんし、web-2 は web1 が実行中かつ待機にならないと展開しません。
もし、web-0 の展開に失敗し、後の web-1 が実行中かつ待機になったとしても、web-2 が起動していなければ、web-0 の起動が完了しない限り web-2 は実行中かつ待機にはなりません。

<!--
If a user were to scale the deployed example by patching the StatefulSet such that
`replicas=1`, web-2 would be terminated first. web-1 would not be terminated until web-2
is fully shutdown and deleted. If web-0 were to fail after web-2 has been terminated and
is completely shutdown, but prior to web-1's termination, web-1 would not be terminated
until web-0 is Running and Ready.
ｰｰ>
もしもユーザが StatefulSet にパッチをあてて（追加で）デプロイするような場合、 `repicas=1` であれば、 web-2 がまずはじめに削除されます。
web-2 が完全にシャットダウンして削除されるまで、web-1 は停止（terminate）されません。
もしも web-2 が削除されて完全にシャットダウンされたあとに web-0 で障害が起こると、web-1 の削除よりも優先されます。web-1 は web-0 が実行・待機中となるまで削除されません。

<!--
### Pod Management Policies
-->
### ポッド管理（Pod Managemement）方針 {#pod-management-policies}

<!--
In Kubernetes 1.7 and later, StatefulSet allows you to relax its ordering guarantees while
preserving its uniqueness and identity guarantees via its `.spec.podManagementPolicy` field.
-->
Kubernetes 1.7 以降では、StatefulSet はユニーク性と同一性の保証が `.spec.podManagementPolicy` フィールドの指定によって、確実な順序づけ（ordering）を緩和できるようになりました。

<!--
#### OrderedReady Pod Management
-->
#### OrderedReady ポッド管理 {#orderedready-pod-management}

<!--
`OrderedReady` pod management is the default for StatefulSets. It implements the behavior
described [above](#deployment-and-scaling-guarantees).
-->
`OrderedReady` ポッド管理とは StatefulSet のためのデフォルトです。
挙動の実装については [前述]](#deployment-and-scaling-guarantees) の通りに説明があります。

<!--
#### Parallel Pod Management
--->
#### Parallel（並列）ポッド管理 {#parallel-pod-management}

<!--
`Parallel` pod management tells the StatefulSet controller to launch or
terminate all Pods in parallel, and to not wait for Pods to become Running
and Ready or completely terminated prior to launching or terminating another
Pod.
-->
`Parallel` （並列）ポッド管理は、StatefulSet コントローラに対して、ポッドの起動と停止を並列に行うように伝えます。また、他のポッドの起動や停止に対する優先度に拘わらず、完全な完了を待たずに、ポッドの起動や削除を行います。

<!--
## Update Strategies
-->
## 更新ストラテジ（Update Strategies） {#update-strategies}

<!--
In Kubernetes 1.7 and later, StatefulSet's `.spec.updateStrategy` field allows you to configure
and disable automated rolling updates for containers, labels, resource request/limits, and
annotations for the Pods in a StatefulSet.
-->
Kubernetes 1.7 以降では、 StatefulSet の `.spec.updateStrategy`  フィールドによって、コントローラ、ラベル、リソース要求・制限、StatefulSet 内のポッドに対する自動化に関して、自動的なローリング・アップデート（逐次更新）の調整や無効化が行えます。

<!--
### On Delete
-->
### On Delete（削除した状態で）{#on-delete}

<!--
The `OnDelete` update strategy implements the legacy (1.6 and prior) behavior. When a StatefulSet's
`.spec.updateStrategy.type` is set to `OnDelete`, the StatefulSet controller will not automatically
update the Pods in a StatefulSet. Users must manually delete Pods to cause the controller to
create new Pods that reflect modifications made to a StatefulSet's `.spec.template`.
-->
`OnDelete` 更新ストラテジの実装は、過去（1.6 以前）の挙動です。
StatefulSet の `.spec.updateStrategy.type` を `OnDelete` に指定すると、StatefulSet コントローラは StatefulSet 内のポッドを自動的に更新しません。
コントローラが新しいポッドを削除できるようにするには、ユーザはポッドを手動で削除する必要があります。StatefulSet の `.spec.template` によって設定が反映されます。

<!--
### Rolling Updates
-->
### Rolling Updates（逐次更新） {#rolling-updates}

<!--
The `RollingUpdate` update strategy implements automated, rolling update for the Pods in a
StatefulSet. It is the default strategy when `.spec.updateStrategy` is left unspecified. When a StatefulSet's `.spec.updateStrategy.type` is set to `RollingUpdate`, the
StatefulSet controller will delete and recreate each Pod in the StatefulSet. It will proceed
in the same order as Pod termination (from the largest ordinal to the smallest), updating
each Pod one at a time. It will wait until an updated Pod is Running and Ready prior to
updating its predecessor.
-->
`RollingUpdate` 更新ストラテジの実装は、StatefulSet 内のポッドに対して自動的なローリング・アップデート（逐次更新）をもたらします。
これは `.spec.updateStrategy`  を未定義な場合のデフォルトのストラテジです。
StatefulSet の `.spec.updateStrategy.type` を `RollingUpdate` にすると、StatefulSet コントローラは StatefulSet 内のポッドを削除および再作成します。
処理の進行はポッドを削除する順番と同じであり、各ポッドを同時に削除する場合と同じです（最も大きなオリジナルから小さなものへ）。
ポッドの更新に先立って、ポッドが実行中かつ待機になるまで待機します。

<!--
#### Partitions
-->
#### パーティション {#partitions}

<!--
The `RollingUpdate` update strategy can be partitioned, by specifying a
`.spec.updateStrategy.rollingUpdate.partition`. If a partition is specified, all Pods with an
ordinal that is greater than or equal to the partition will be updated when the StatefulSet's
`.spec.template` is updated. All Pods with an ordinal that is less than the partition will not
be updated, and, even if they are deleted, they will be recreated at the previous version. If a
StatefulSet's `.spec.updateStrategy.rollingUpdate.partition` is greater than its `.spec.replicas`,
updates to its `.spec.template` will not be propagated to its Pods.
In most cases you will not need to use a partition, but they are useful if you want to stage an
update, roll out a canary, or perform a phased roll out.
-->
`RollingUpdate` 更新ストラテジでは `.spec.updateStrategy.rollingUpdate.partition` の指定によって分割して行えます。
パーティションが指定されると、全てのポッドが順序づけされ、 StatefulSet の `.spec.template` が更新されると、パーティションと一致するか大きいものが更新されます。
順序付けられたポッドが、パーティションよりも小さければ更新されませんし、削除されたとしても、以前のバージョンのものを再作成します。
ほとんどの場合、パーティションを使う必要はありません。
しかし、ステージごとの更新や、カナリア方式のロールアウトや、段階的なロールアウトを処理するには役立つでしょう。


{{% /capture %}}
{{% capture whatsnext %}}

* Follow an example of [deploying a stateful application](/docs/tutorials/stateful-application/basic-stateful-set/).
* Follow an example of [deploying Cassandra with Stateful Sets](/docs/tutorials/stateful-application/cassandra/).

{{% /capture %}}

