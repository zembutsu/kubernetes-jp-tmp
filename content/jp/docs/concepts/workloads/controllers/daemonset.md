---
reviewers:
- enisoc
- erictune
- foxish
- janetkuo
- kow3ns
title: DaemonSet（デーモンセット）
content_template: templates/concept
weight: 50
---

{{% capture overview %}}

<!--
A _DaemonSet_ ensures that all (or some) Nodes run a copy of a Pod.  As nodes are added to the
cluster, Pods are added to them.  As nodes are removed from the cluster, those Pods are garbage
collected.  Deleting a DaemonSet will clean up the Pods it created.
-->
_DaemonSet（デーモンセット）_ は、ポッドのコピー（複製）を全て（または一部の）ノードで実行します。
ノードがクラスタに追加されると、ポッドもそのノードに対して追加されます。
ノードがクラスタから削除されれば、ポッドも削除（ゴミ収集）されます。
DaemonSet を削除すると、DaemonSet によって作成されたポッドも削除されます。

<!--
Some typical uses of a DaemonSet are:
-->
DaemonSet の典型的な使い方は：

<!--
- running a cluster storage daemon, such as `glusterd`, `ceph`, on each node.
- running a logs collection daemon on every node, such as `fluentd` or `logstash`.
- running a node monitoring daemon on every node, such as [Prometheus Node Exporter](
  https://github.com/prometheus/node_exporter), `collectd`, Datadog agent, New Relic agent, or Ganglia `gmond`.
-->
- クラスタ・ストレージ・デーモンを各ノードで実行。`glustered` や `ceph` など
- 全てのノードでログ収集デーモンを実行。 `fluentd` や `logstash` など。
- 全てのノードでノード監視用デーモンを実行。[Prometheus Node Exporter](https://github.com/prometheus/node_exporter)、 `collectd` 、Datadog エージェント、New Relic エージェント、Ganglia `gmond` など。

<!--
In a simple case, one DaemonSet, covering all nodes, would be used for each type of daemon.
A more complex setup might use multiple DaemonSets for a single type of daemon, but with
different flags and/or different memory and cpu requests for different hardware types.
-->
簡単な例としては、DaemonSet は全てのノードをカバーし、様々な種類のデーモンを動かすために使えます。
複雑なセットアップ例としては、１つのデーモンのために、複数 DaemonSet のセットアップです。ただし、ハードウェアの種類によって、異なるフラグをつけたり、異なるメモリや CPU 要求を指定します。


{{% /capture %}}

{{< toc >}}

{{% capture body %}}

<!--
## Writing a DaemonSet Spec
-->
## DaemonSet Spec を書くには {#writing-a-daemonset-spec}

<!--
### Create a DaemonSet
-->
### DaemonSet の作成 {#create-a-daemonset}

<!--
You can describe a DaemonSet in a YAML file. For example, the `daemonset.yaml` file below describes a DaemonSet that runs the fluentd-elasticsearch Docker image:
-->
DaemonSet は YAML ファイルで記述できます。
たとえば以下の `daemonset.yaml` ファイルで DaemonSet を記述する例では、fluent-elasticsearch Docker イメージを実行します。

{{< codenew file="controllers/daemonset.yaml" >}}

<!--
* Create a DaemonSet based on the YAML file:
```
kubectl create -f https://k8s.io/examples/controllers/daemonset.yaml
```
-->

YAML ファイルをベースとした DaemonSet を作成するには：
```
kubectl create -f https://k8s.io/examples/controllers/daemonset.yaml
```

<!--
### Required Fields
-->
### 必要なフィールド {#required-fields}

<!--
As with all other Kubernetes config, a DaemonSet needs `apiVersion`, `kind`, and `metadata` fields.  For
general information about working with config files, see [deploying applications](/docs/user-guide/deploying-applications/),
[configuring containers](/docs/tasks/), and [object management using kubectl](/docs/concepts/overview/object-management-kubectl/overview/) documents.
-->
他の Kubernetes 設定情報ファイルと同様に、DaemonSet では `apiVersion` 、 `kind` 、 `metadata`  フィールドが必要です。
設定情報ファイルの働きに関する一般的な情報は、[アプリケーションのデプロイ](/jp/docs/user-guide/deploying-applications/)、
[コンテナの設定](/jp/docs/tasks/)、[kubectl によるオブジェクト管理](/jp/docs/concepts/overview/object-management-kubectl/overview/)の各ドキュメントをご覧ください。

<!--
A DaemonSet also needs a [`.spec`](https://git.k8s.io/community/contributors/devel/api-conventions.md#spec-and-status) section.
-->
また、DaemonSet には [`.spec`](https://git.k8s.io/community/contributors/devel/api-conventions.md#spec-and-status) セクションも必要です。

<!--
### Pod Template
-->
### ポッド・テンプレート {#pod-tempalte}

<!--
The `.spec.template` is one of the required fields in `.spec`.
-->
`.spec.template` は `.spec` フィールドに必要な要素です。

<!--
The `.spec.template` is a [pod template](/docs/concepts/workloads/pods/pod-overview/#pod-templates). It has exactly the same schema as a [Pod](/docs/concepts/workloads/pods/pod/), except it is nested and does not have an `apiVersion` or `kind`.
-->
`.spec.template` は  [ポッド・テンプレート](/jp/docs/concepts/workloads/pods/pod-overview/#pod-templates) です。
これは [ポッド](/jp/docs/concepts/workloads/pods/pod/) と完全に同じスキーマですが、ネスト化せず、 `apiVersion` と `kind` を持ちません。

<!--
In addition to required fields for a Pod, a Pod template in a DaemonSet has to specify appropriate
labels (see [pod selector](#pod-selector)).
-->
ポッドに対して必要なフィールドに加えて、DaemonSet 内のポッド・テンプレートには適切なラベルの指定が必要です（詳細は [ポッド・セレクタ](#pod-selector) をご覧ください。）。

<!--
A Pod Template in a DaemonSet must have a [`RestartPolicy`](/docs/user-guide/pod-states)
 equal to `Always`, or be unspecified, which defaults to `Always`.
-->
DaemonSet 内のポッド・セレクタは、 [`RestartPolicy`（再起動方針）](/jp/docs/user-guide/pod-states)が常に `Always` （常時）と同等であり、もし指定が無ければ、デフォルトで `Always`  になります。

<!---
### Pod Selector
-->
### ポッド・セレクタ {#pod-selector}

<!--
The `.spec.selector` field is a pod selector.  It works the same as the `.spec.selector` of
a [Job](/docs/concepts/jobs/run-to-completion-finite-workloads/).
-->
`.spec.selector` フィールドはポッド/セレクタです。
これは [ジョブ](/jp/docs/concepts/jobs/run-to-completion-finite-workloads/) の `.spec.selector` と同じ挙動です。

<!--
As of Kubernetes 1.8, you must specify a pod selector that matches the labels of the
`.spec.template`. The pod selector will no longer be defaulted when left empty. Selector
defaulting was not compatible with `kubectl apply`. Also, once a DaemonSet is created,
its `.spec.selector` can not be mutated. Mutating the pod selector can lead to the
unintentional orphaning of Pods, and it was found to be confusing to users.
-->
Kubernetes 1.8 では、 マシンの`.spec.templaet` のラベルと一致するポッド・セレクタを指定する必要があります。
ポッド・セレクタは、デフォルトでは何も指定されていません。
セレクタのデフォルト化は `kubectl apply` と互換性がありません。
また、 DaemonSet が作成されても、 `.spec.selector` はマウントされません。
ポッド・セレクタの変更（mutating）によって、意図しないポッドの孤立をもたらし、見つけられなくなるのでユーザに対して混乱を与えるでしょう。

<!--
The `.spec.selector` is an object consisting of two fields:
-->
`.spec.selector` は２つのフィールドによって構成されるオブジェクトです。

<!--
* `matchLabels` - works the same as the `.spec.selector` of a [ReplicationController](/docs/concepts/workloads/controllers/replicationcontroller/).
* `matchExpressions` - allows to build more sophisticated selectors by specifying key,
  list of values and an operator that relates the key and values.
-->
* `matchLabels` - [ReplicationController](/jp/docs/concepts/workloads/controllers/replicationcontroller/) の `.spec.selector` と同じ挙動です。
* `matchExpressions` - より適切なセレクタによって構築できるようになります。関係するキーと値によって、キー、値のリスト、演算子（オペレータ）を指定します。

<!--
When the two are specified the result is ANDed.
-->
２つが指定されると、結果は AND として処理されます。

<!--
If the `.spec.selector` is specified, it must match the `.spec.template.metadata.labels`. Config with these not matching will be rejected by the API.
-->
`.spec.selector` が指定されると、これは  `.spec.template.metadata.labels` と一致する必要があります。
設定が一致しなければ、API によって拒否されます。

<!--
Also you should not normally create any Pods whose labels match this selector, either directly, via
another DaemonSet, or via other controller such as ReplicaSet.  Otherwise, the DaemonSet
controller will think that those Pods were created by it.  Kubernetes will not stop you from doing
this.  One case where you might want to do this is manually create a Pod with a different value on
a node for testing.
-->
また、通常はこのポッドに対するラベルは以下のものと一致すべきではありません。セレクタのみならず、他の DaemonSet を経由して作られたラベル、他の ReplicaSet のようなコントローラを通して作成されたものラベル。
そうしなければ、DaemonSet コントローラは、それぞれのポッドが各コントローラによって作成されたとみなされます。
Kubernetes は、この指定を止められません。
やむを得ず手動でポッドを作成したい場合は、テストのために異なったノードで実施してください。

<!--
### Running Pods on Only Some Nodes
-->
### 同じノード上でのみポッドを実行 {#running-pods-on-only-some-nodes}

<!--
If you specify a `.spec.template.spec.nodeSelector`, then the DaemonSet controller will
create Pods on nodes which match that [node
selector](/docs/concepts/configuration/assign-pod-node/). Likewise if you specify a `.spec.template.spec.affinity`,
then DaemonSet controller will create Pods on nodes which match that [node affinity](/docs/concepts/configuration/assign-pod-node/).
If you do not specify either, then the DaemonSet controller will create Pods on all nodes.
-->
`.spec.template.spec.nodeSelector` を指定すると、DaemonSet コントローラは  [ノード・セレクタ](/jp/docs/concepts/configuration/assign-pod-node/) に一致するノード上でポッドを作成します。
同様に、.spec.template.spec.affinity` を指定すると、DaemonSet コントローラは [ノード親和性（node affinity）](/jp/docs/concepts/configuration/assign-pod-node/) に一致するノード上でポッドを作成します。
もし、どちらも指定しなければ、DaemonSet コントローラは全てのノード上でポッドを作成します。

<!--
## How Daemon Pods are Scheduled
-->
## どのようにしてデーモン・ポッドがスケジュールされるのか {#how-daemon-pods-are-scheduled}

<!--
### Scheduled by DaemonSet controller (default)
-->
### DaemonSet コントローラによってスケジュールされる（デフォルト） {#scheduled-by-daemonset-controller-default}

<!--
Normally, the machine that a Pod runs on is selected by the Kubernetes scheduler. However, Pods
created by the DaemonSet controller have the machine already selected (`.spec.nodeName` is specified
when the Pod is created, so it is ignored by the scheduler).  Therefore:
-->
通常、ポッドを実行するマシンを選ぶのは Kubernetes スケジューラです。
しかしながら、DaemonSet コントローラによって作成されたポッドは、すでにマシンが選択されています（ポッド作成時に `.spec.nodeName` が指定されていても、スケジューラによって無視されます ）。
つまり、

<!--
 - The [`unschedulable`](/docs/admin/node/#manual-node-administration) field of a node is not respected
   by the DaemonSet controller.
 - The DaemonSet controller can make Pods even when the scheduler has not been started, which can help cluster
   bootstrap.
-->
  - DaemonSet コントローラによってノードの [`unschedulable`（スケジュールしない）](/jp/docs/admin/node/#manual-node-administration)は無視されます。
  - DaemonSet コントローラはスケジューラが開始されていなくても、クラスタのブートストラップの助けを得て、ポッドを作成できます。

<!--
### Scheduled by default scheduler
-->
### デフォルトのスケジューラによってスケジュール {#scheduled-by-default-scheduler}

{{< feature-state state="alpha" for-kubernetes-version="1.11" >}}

<!--
A DaemonSet ensures that all eligible nodes run a copy of a Pod. Normally, the
node that a Pod runs on is selected by the Kubernetes scheduler. However,
DaemonSet pods are created and scheduled by the DaemonSet controller instead.
That introduces the following issues:
-->
DaemonSet は全ての適切なノードでポッドのコピーを確実に実行します。
通常、Kubernetes スケジューラによってポッドを実行するノードが選択されます。
しかしながら、DaemonSet ポッドは DaemonSet コントローラが代わりにスケジュールして作成するものです。
これには以下の課題があります：

<!--
 * Inconsistent Pod behavior: Normal Pods waiting to be scheduled are created
   and in `Pending` state, but DaemonSet pods are not created in `Pending`
   state. This is confusing to the user.
 * [Pod preemption](/docs/concepts/configuration/pod-priority-preemption/)
   is handled by default scheduler. When preemption is enabled, the DaemonSet controller
   will make scheduling decisions without considering pod priority and preemption.
 -->
   * ポッドの挙動に一貫性がない：通常、スケジュールされたポッドが作成されると `Pending` （保留中）のステータスになります。しかしDaemonSet ポッドは `Pending` 状態としては作成されません。これはユーザに混乱をもたらします。
   * デフォルトのスケジューラによって  [Pod preemption（ポッド先行取得）](/jp/docs/concepts/configuration/pod-priority-preemption/) が扱われます。先行取得を有効化すると、DaemonSet コントローラはポッドの優先度と先行取得を一切考慮せずスケジュールを決定できるようにします。
  
 <!--
`ScheduleDaemonSetPods` allows you to schedule DaemonSets using the default
scheduler instead of the DaemonSet controller, by adding the `NodeAffinity` term
to the DaemonSet pods, instead of the `.spec.nodeName` term. The default
scheduler is then used to bind the pod to the target host. If node affinity of
the DaemonSet pod already exists, it is replaced. The DaemonSet controller only
performs these operations when creating or modifying DaemonSet pods, and no
changes are made to the `spec.template` of the DaemonSet.
-->
`ScheduleDaemonSetPods` は DaemonSet コントローラのかわりにデフォルトのスケジューラを使って DaemonSet をスケジュールできるようにします。
そのためには DaemonSet ポッドに対して `.spec.nodeName` 項目にかわり `NodeAffinity` （ノード親和性）項目を指定します。
デフォルトのスケジューラは、対象となるホスト上でポッドを割り当てるために使います。
もしもDaemonSet ポッドの ノード親和性が既に指定されていれば、置き換えられます。
DaemonSet コントローラが処理するのは、DaemonSet ポッドによって作成または変更されたものだけであり、 DaemonSet の `spec.template` によって作成されたものには変更を加えません。

```yaml
nodeAffinity:
  requiredDuringSchedulingIgnoredDuringExecution:
    nodeSelectorTerms:
    - matchFields:
      - key: metadata.name
        operator: In
        values:
        - target-host-name
```

<!--
In addition, `node.kubernetes.io/unschedulable:NoSchedule` toleration is added
automatically to DaemonSet Pods. The DaemonSet controller ignores
`unschedulable` Nodes when scheduling DaemonSet Pods. You must enable
`TaintModesByCondition` to ensure that the default scheduler behaves the same
way and schedules DaemonSet pods on `unschedulable` nodes.
-->
加えて、 `node.kubernetes.io/unschedulable:NoSchedule` 耐性が DaemonSet ポッドに対して自動的に追加されます。
DaemonSet ポッドのスケジュール時には、DaemonSet コントローラは `unschedulable` （スケジュール不可）ノードを無視します。
デフォルトのスケジューラでも同じ挙動にする場合は、 `TaintModesByCondition` を有効化する必要があります。それから `unschedulable` なノードに対して DaemonSet ポッドをスケジュールします。

<!--
When this feature and `TaintNodesByCondition` are enabled together, if DaemonSet
uses the host network, you must also add the
`node.kubernetes.io/network-unavailable:NoSchedule toleration`.
-->
この機能と `TaintNodesByCondition` が一緒に有効化されている場合、もしも DaemonSet がホストネットワークを使う場合は、`node.kubernetes.io/network-unavailable:NoSchedule toleration` の指定が必須です。

<!--
### Taints and Tolerations
-->
### テイント（汚染：taint）と耐性（toleration）{#taints-and-tolerations}

<!--
Although Daemon Pods respect
[taints and tolerations](/docs/concepts/configuration/taint-and-toleration),
the following tolerations are added to DamonSet Pods automatically according to
the related features.
-->
Daemon ポッドは [テイントと耐性](/jp/docs/concepts/configuration/taint-and-toleration) を尊重しますが、以下の耐性が関連する機能によって DaemonSet ポッドに対して自動的に付与されます。

<!--
| Toleration Key                           | Effect     | Alpha Features                                               | Version | Description                                                  |
| ---------------------------------------- | ---------- | ------------------------------------------------------------ | ------- | ------------------------------------------------------------ |
| `node.kubernetes.io/not-ready`           | NoExecute  | `TaintBasedEvictions`                                        | 1.8+    | when `TaintBasedEvictions`  is enabled,they will not be evicted when there are node problems such as a network partition. |
| `node.kubernetes.io/unreachable`         | NoExecute  | `TaintBasedEvictions`                                        | 1.8+    | when `TaintBasedEvictions`  is enabled,they will not be evicted when there are node problems such as a network partition. |
| `node.kubernetes.io/disk-pressure`       | NoSchedule | `TaintNodesByCondition`                                      | 1.8+    |                                                              |
| `node.kubernetes.io/memory-pressure`     | NoSchedule | `TaintNodesByCondition`                                      | 1.8+    |                                                              |
| `node.kubernetes.io/unschedulable`       | NoSchedule | `ScheduleDaemonSetPods`, `TaintNodesByCondition`             | 1.11+   | When ` ScheduleDaemonSetPods` is enabled, ` TaintNodesByCondition` is necessary to make sure DaemonSet pods tolerate unschedulable attributes by default scheduler. |
| `node.kubernetes.io/network-unavailable` | NoSchedule | `ScheduleDaemonSetPods`, `TaintNodesByCondition`, hostnework | 1.11+   | When ` ScheduleDaemonSetPods` is enabled, ` TaintNodesByCondition` is necessary to make sure DaemonSet pods, who uses host network, tolerate network-unavailable attributes by default scheduler. |
| `node.kubernetes.io/out-of-disk`         | NoSchedule | `ExperimentalCriticalPodAnnotation` (critical pod only), `TaintNodesByCondition` | 1.8+    |                                                              |
-->

| Toleration（耐性）Key                           | 効果     | アルファ機能                                               | バージョン | 説明                                                  |
| ---------------------------------------- | ---------- | ------------------------------------------------------------ | ------- | ------------------------------------------------------------ |
| `node.kubernetes.io/not-ready`           | 何も実行しない  | `TaintBasedEvictions`                                        | 1.8+    | `TaintBasedEvictions` を有効化すると、ネットワーク分割によってノードで問題が発生してもノードを退去しない 。|
| `node.kubernetes.io/unreachable`         | 何も実行しない  | `TaintBasedEvictions`                                        | 1.8+    | `TaintBasedEvictions` を有効化すると、ネットワーク分割によってノードで問題が発生してもノードを退去しない 。 |
| `node.kubernetes.io/disk-pressure`       | 何もスケジュールしない | `TaintNodesByCondition`                                      | 1.8+    |                                                              |
| `node.kubernetes.io/memory-pressure`     | 何もスケジュールしない | `TaintNodesByCondition`                                      | 1.8+    |                                                              |
| `node.kubernetes.io/unschedulable`       | 何もスケジュールしない | `ScheduleDaemonSetPods`, `TaintNodesByCondition`             | 1.11+   |  `ScheduleDaemonSetPods` を有効化すると、DaemonoSet ポッドがデフォルト・スケジューラによってスケジュールできない属性にします。 |
| `node.kubernetes.io/network-unavailable` | 何もスケジュールしない | `ScheduleDaemonSetPods`, `TaintNodesByCondition`, hostnework | 1.11+   | `ScheduleDaemonSetPods` を有効化すると、ホスト・ネットワークを使っていても、DaemonoSet ポッドがデフォルト・スケジューラによってスケジュールできない属性にします。  |
| `node.kubernetes.io/out-of-disk`         | 何もスケジュールしない | `ExperimentalCriticalPodAnnotation` (critical pod only), `TaintNodesByCondition` | 1.8+    |                                                              |


<!--
## Communicating with Daemon Pods
-->
## デーモン・ポッドとの通信 {$communicating-with-daemon-pods}

<!--
Some possible patterns for communicating with Pods in a DaemonSet are:
-->
DaemonSet がポッドと通信する可能性があるパターンは複数あります：

<!--
- **Push**: Pods in the DaemonSet are configured to send updates to another service, such
  as a stats database.  They do not have clients.
- **NodeIP and Known Port**: Pods in the DaemonSet can use a `hostPort`, so that the pods are reachable via the node IPs.  Clients know the list of node IPs somehow, and know the port by convention.
- **DNS**: Create a [headless service](/docs/concepts/services-networking/service/#headless-services) with the same pod selector,
  and then discover DaemonSets using the `endpoints` resource or retrieve multiple A records from
  DNS.
- **Service**: Create a service with the same Pod selector, and use the service to reach a
  daemon on a random node. (No way to reach specific node.)
-->
- **送信（push）**: DaemonSet 内のポッドは、データベースの状態など、他のサービスに対して更新情報を送るための設定が可能です。これらはクライアントを持ちません。
- ***ノード IP と既知ポート*: DaemonSet のポッドは `hostPort` を使えますので、ポッドにはノード IP を経由して到達できます。クライアントが何らかの手段でノード IP の一覧とポートが分かれば便利でしょう。
- **DNS**: 同じポッド・セレクタで [ヘッドレス・サービス（headless service）](/jp/docs/concepts/services-networking/service/#headless-services) を作成します。そして、DaemonSet は `endpoints` リソースや DNS の A レコードを代わりに使って発見できるようになります。
- **サービス**: 同じポッド・セレクタを使ってサービスを作成し、ランダムなノード上のデーモンにサービスを到達させます（到達先のノードを指定できません）。

<!--
## Updating a DaemonSet
-->
## DaemonSet の更新 {#updating-a-daeomonset}

<!--
If node labels are changed, the DaemonSet will promptly add Pods to newly matching nodes and delete
Pods from newly not-matching nodes.
-->
ノードのラベルを変更すると、DaemonSet は即座に一致するノードにポッドを追加し、一致しないノードからはポッドを削除します。

<!--
You can modify the Pods that a DaemonSet creates.  However, Pods do not allow all
fields to be updated.  Also, the DaemonSet controller will use the original template the next
time a node (even with the same name) is created.
-->
ポッドの調整は DaemonSet 作成時に行えます。
しかし、ポッドは全てのフィールドを更新できません。
また、DaemonSet コントローラは次回にノードが作成されるときはオリジナルの（元々あった）テンプレートを使います（たとえ同じ名前の場合であっても）。

<!--
You can delete a DaemonSet.  If you specify `--cascade=false` with `kubectl`, then the Pods
will be left on the nodes.  You can then create a new DaemonSet with a different template.
The new DaemonSet with the different template will recognize all the existing Pods as having
matching labels.  It will not modify or delete them despite a mismatch in the Pod template.
You will need to force new Pod creation by deleting the Pod or deleting the node.
-->
DaemonSet は削除可能です。
もし `kubectl` で `--cascade=false` を指定すると、ポッドは対象ノードを離れます。
それから別のテンプレートを使って新しい DaemonSet を作成できます。
別のテンプレートを使う新しい DaemonSet は、全ての既存のポットが持っているラベルが一致するかどうか認識します。
ポッド・テンプレートが一致しないの変更または削除できません。
新しいポッドを強制的に作成する必用がある場合は、ノードまたはポッドを削除する必要があります。

<!--
In Kubernetes version 1.6 and later, you can [perform a rolling update](/docs/tasks/manage-daemon/update-daemon-set/) on a DaemonSet.
-->
Kubernetes バージョン 1.6 以降では、DaemonSet 上で [ローリング・アップデートを処理](/jp/docs/tasks/manage-daemon/update-daemon-set/) できます。

<!--
## Alternatives to DaemonSet
-->
## DaemonSet の代替 {#alternatives-to-daemonset}

<!--
### Init Scripts
-->
### Init スクリプト{#init-scripts}

<!--
It is certainly possible to run daemon processes by directly starting them on a node (e.g. using
`init`, `upstartd`, or `systemd`).  This is perfectly fine.  However, there are several advantages to
running such processes via a DaemonSet:
-->
ノード上で確実に実行可能なデーモン・プロセスを直接実行します（例： `init`、`upstard`、`systemd` の使用 ）。
この方法が完全に正しいです。
しかし、DaemonSet を経由するプロセスを使う方法に複数の利点があります。

<!--
- Ability to monitor and manage logs for daemons in the same way as applications.
- Same config language and tools (e.g. Pod templates, `kubectl`) for daemons and applications.
- Running daemons in containers with resource limits increases isolation between daemons from app
  containers.  However, this can also be accomplished by running the daemons in a container but not in a Pod
  (e.g. start directly via Docker).
-->
- アプリケーションと同じ手法でデーモンにタンする監視やログ管理ができる
- デーモンとアプリケーションに対して同じ設定言語とツールを使える（例、ポッド・テンプレート、 `kubectl`）
- デーモンをコンテナ内で実行すると、リソースを制限し、デーモンとアプリケーション・コンテナ間の隔たり（isolation）を高めます。しかし、これによってデーモンはポッドではなく、完全にコンテナとして実行する必要があります（例：Docker を経由して直接起動）。

<!--
### Bare Pods
-->
### 単なるポッド（Bare Pods） {#bare-pods}

<!--
It is possible to create Pods directly which specify a particular node to run on.  However,
a DaemonSet replaces Pods that are deleted or terminated for any reason, such as in the case of
node failure or disruptive node maintenance, such as a kernel upgrade. For this reason, you should
use a DaemonSet rather than creating individual Pods.
-->
ポッドを作成するにあたり、特定のノード上での実行を直接指定できる場合があります。
しかしながら、DaemonSet は何らかの理由によって、ポッドを削除または停止して置き換える場合があります。たとえば、ノード障害やカーネル更新のような破壊的なノードのメンテナンスです。
このような理由のため、DaemonSet を使うよりは、個々のポッドを作成するほうが良い場合もあるでしょう。

<!--
### Static Pods
-->
### 静的なポッド {#static-pods}

<!--
It is possible to create Pods by writing a file to a certain directory watched by Kubelet.  These
are called [static pods](/docs/concepts/cluster-administration/static-pod/).
Unlike DaemonSet, static Pods cannot be managed with kubectl
or other Kubernetes API clients.  Static Pods do not depend on the apiserver, making them useful
in cluster bootstrapping cases.  Also, static Pods may be deprecated in the future.
-->
kubelet が監視している特定のディレクトリ上にファイルを書いて、ポッドを作成できます。
これらは [静的ポッド（static pods）](/jp/docs/concepts/cluster-administration/static-pod/) と呼びます。
DaemonSet とは違い、静的なポッドは kubectl や他の Kubernetes API クライアントで操作できません。
静的ポッドは apiserver に依存しないため、クラスタの立ち上げ時の構成に役立つでしょう。
また、静的ポッドは将来的に廃止の可能性があります。

<!--
### Deployments
-->
### デプロイメント {#deployment}

<!--
DaemonSets are similar to [Deployments](/docs/concepts/workloads/controllers/deployment/) in that
they both create Pods, and those Pods have processes which are not expected to terminate (e.g. web servers,
storage servers).
-->
DaemonSet は  [デプロイメント](/jp/docs/concepts/workloads/controllers/deployment/) と似ています。
どちらもポッドを作成し、どちらのポッドも停止しないプロセスを持ちます（例：ウェブサーバ、ストレージ・サーバ）。

<!--
Use a Deployment for stateless services, like frontends, where scaling up and down the
number of replicas and rolling out updates are more important than controlling exactly which host
the Pod runs on.  Use a DaemonSet when it is important that a copy of a Pod always run on
all or certain hosts, and when it needs to start before other Pods.
-->
フロントエンドなど、ステートレス（状態を持たない）サービスのためにデプロイメントを使うと、スケールのアップ・ダウン時に複製数とローリング・アウト（展開）は、ポッド上で実行するよりも確実に制御するのが重要になります。
DaemonSet を使うにあたり重要なのは、ポッドのコピーが常に全てまたは特定のホスト上で動いており、他のポッドよりも速く実行する必要がある時です。

{{% /capture %}}
