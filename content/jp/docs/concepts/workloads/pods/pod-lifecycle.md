---
title: ポッドのライフサイクル
content_template: templates/concept
weight: 30
---

{{% capture overview %}}

{{< comment >}}Updated: 4/14/2015{{< /comment >}}
{{< comment >}}Edited and moved to Concepts section: 2/2/17{{< /comment >}}

<!--
This page describes the lifecycle of a Pod.
-->
このページはコンテナのライフサイクルについて説明します。

{{% /capture %}}


{{% capture body %}}

<!--
## Pod phase
-->
## ポッドの段階（Pod phase） {#pod-phase}

<!--
A Pod's `status` field is a
[PodStatus](/docs/reference/generated/kubernetes-api/{{< param "version" >}}/#podstatus-v1-core)
object, which has a `phase` field.
-->
ポッドの `status` （状態）フィールドは [PodStatus（ポッド・ステータス）](/jp/docs/reference/generated/kubernetes-api/{{< param "version" >}}/#podstatus-v1-core) オブジェクトであり、 `phase` （段階：フェーズ）フィールドがあります。

<!--
The phase of a Pod is a simple, high-level summary of where the Pod is in its
lifecycle. The phase is not intended to be a comprehensive rollup of observations
of Container or Pod state, nor is it intended to be a comprehensive state machine.
-->
ポッドの段階（phase）とはシンプルであり、ポッドのライフサイクルにおいてはハイレベルな位置にまとめられています。段階とは、コンテナやポッド状態を包括的に監視できるようにするのを目的としていません。また、包括的なマシン状態のためでもありません。

<!--
The number and meanings of Pod phase values are tightly guarded.
Other than what is documented here, nothing should be assumed about Pods that
have a given `phase` value.
-->
ポッドの段階で表示される値の数字と意味は、厳密に固定されています。ここでドキュメント化されているもの以外で、ポッドにおける  `phase` （段階：フェーズ）の値に関する記述はありません。

<!--
Here are the possible values for `phase`:
-->
以下が `phase` として取り得る値です：

<!--
Value | Description
:-----|:-----------
`Pending` | The Pod has been accepted by the Kubernetes system, but one or more of the Container images has not been created. This includes time before being scheduled as well as time spent downloading images over the network, which could take a while.
`Running` | The Pod has been bound to a node, and all of the Containers have been created. At least one Container is still running, or is in the process of starting or restarting.
`Succeeded` | All Containers in the Pod have terminated in success, and will not be restarted.
`Failed` | All Containers in the Pod have terminated, and at least one Container has terminated in failure. That is, the Container either exited with non-zero status or was terminated by the system.
`Unknown` | For some reason the state of the Pod could not be obtained, typically due to an error in communicating with the host of the Pod.
-->
値 | 説明
:-----|:-----------
`Pending`（保留中） | ポッドは Kubernetes システムに受諾されましたが、１つまたは複数のコンテナ・イメージ作成が完了していません。これに含まれるのはスケジュールされる前の時間だけでなく、ネットワークを越えてイメージをダウンロードするのにかかる時間も含むので、時間がかかる場合があります。
`Running`（実行中） | ポッドがノードに割り当てられ、全てのコンテナが作成された状態です。少なくとも１つのコンテナが実行中であるか、あるいはプロセスが開始中または再起動中です。
`Succeeded`（成功） | ポッドの全てのコンテナが処理終了（terminated）に成功し、再起動しません。
`Failed`（失敗） | ポッド内の全てのコンテナの処理終了に失敗し、少なくとも１つのコンテナの処理終了が失敗しました。つまり、コンテナは０以外のステータスで以上終了したか、システムによって終了させられました。
`Unknown`（不明） | 何らかの理由によって、ポッドから状態を取得できません。典型的なのはポットのホストとの通信がエラーです。

<!--
## Pod conditions
-->
## ポッドの状態（Pod conditions） {#pod-conditions}

<!--
A Pod has a PodStatus, which has an array of
[PodConditions](/docs/reference/generated/kubernetes-api/{{< param "version" >}}/#podcondition-v1-core)
through which the Pod has or has not passed. Each element of the PodCondition
array has six possible fields:
-->
ポッドには PodStatus があります。これはポッドにある [PodConditions](/jp/docs/reference/generated/kubernetes-api/{{< param "version" >}}/#podcondition-v1-core) 配列を通してであり、そうでなければ得られません。PodCondition 配列の各要素は６つのフィールドを持つでしょう：

<!--
* The `lastProbeTime` field provides a timestamp for when the Pod condition
  was last probed.

* The `lastTransitionTime` field provides a timestamp for when the Pod
  last transitioned from one status to another.

* The `message` field is a human-readable message indicating details
  about the transition.
-->
* `lastProbeTime` フィールドが提供するのは、ポッドの状態を調べた直近のタイムスタンプです。
* `lastTransitionTime` フィールドが提供するのは、あるポッドのステータスが別のステータスに移行した、直近のタイムスタンプです。
* `message` フィールドが提供するのは、人間が読める移行に関する詳細なメッセージを表示します。

<!--
A Pod has a PodStatus, which has an array of
[PodConditions](/docs/reference/generated/kubernetes-api/{{< param "version" >}}/#podcondition-v1-core). Each element
of the PodCondition array has a `type` field and a `status` field. The `type`
field is a string with the following possible values:
-->
ポッドには PodStatus があり、これには [PodConditions](/jp/docs/reference/generated/kubernetes-api/{{< param "version" >}}/#podcondition-v1-core) の配列があります。PodCondition 配列の各要素には `type` フィールドと `status` フィールドがあります。 `type` フィールドには、次の値の文字列がある可能性があります：

<!--
* `PodScheduled`: the Pod has been scheduled to a node;
* `Ready`: the Pod is able to serve requests and should be added to the load
  balancing pools of all matching Services;
* `Initialized`: all [init containers](/docs/concepts/workloads/pods/init-containers)
  have started successfully;
* `Unschedulable`: the scheduler cannot schedule the Pod right now, for example
  due to lacking of resources or other constraints;
* `ContainersReady`: all containers in the Pod are ready.
-->
* `PodScheduled`（ポッドがスケジュール済み）: ポッドがノードにスケジュールされた
* `Ready`（待機）: ポットが全ての対象サービスを負荷分散プールに追加して、リクエストを処理可能な状態
* `Initialized`（初期化）: 全ての [コンテナ初期化（init containers）](/jp/docs/concepts/workloads/pods/init-containers) 処理に成功
* `Unschedulable`（スケジュール不可）: スケジューラがポッドを適切に処理できない。他のコンテナによってリソースが欠乏しているなど
* `ContainersReady`（コンテナ準備完了）: ポッド内の全コンテナの準備が完了

<!--
The `status` field is a string, with possible values "`True`", "`False`", and
"`Unknown`".
-->
`status` （状態）フィールドは "``True" か "`Flase`" か "`Unknown`" の文字列です。

<!--
## Container probes
-->
## コンテナ調査（Container probes） {#container-probes}

<!--
A [Probe](/docs/reference/generated/kubernetes-api/{{< param "version" >}}/#probe-v1-core) is a diagnostic
performed periodically by the [kubelet](/docs/admin/kubelet/)
on a Container. To perform a diagnostic,
the kubelet calls a
[Handler](https://godoc.org/k8s.io/kubernetes/pkg/api/v1#Handler) implemented by
the Container. There are three types of handlers:
-->
[Probe（調査）](/jp/docs/reference/generated/kubernetes-api/{{< param "version" >}}/#probe-v1-core) とは  [kubelet](/jp/docs/admin/kubelet/) がコンテナ上で行う定期的な性能診断です。診断の処理は kubelet がコンテナに実装された [Handlerアンドラ）](https://godoc.org/k8s.io/kubernetes/pkg/api/v1#Handler) を呼び出します。ハンドラは3種類あります：

<!--
* [ExecAction](/docs/reference/generated/kubernetes-api/{{< param "version" >}}/#execaction-v1-core):
  Executes a specified command inside the Container. The diagnostic
  is considered successful if the command exits with a status code of 0.

* [TCPSocketAction](/docs/reference/generated/kubernetes-api/{{< param "version" >}}/#tcpsocketaction-v1-core):
  Performs a TCP check against the Container's IP address on
  a specified port. The diagnostic is considered successful if the port is open.

* [HTTPGetAction](/docs/reference/generated/kubernetes-api/{{< param "version" >}}/#httpgetaction-v1-core):
  Performs an HTTP Get request against the Container's IP
  address on a specified port and path. The diagnostic is considered successful
  if the response has a status code greater than or equal to 200 and less than 400.
-->
* [ExecAction](/jp/docs/reference/generated/kubernetes-api/{{< param "version" >}}/#execaction-v1-core)（実行アクション）：コンテナ内で指定されたコマンドを実行します。コマンドがステータス・コードが 0 で終了すると、診断に成功したとみなします。

* [TCPSocketAction](/jp/docs/reference/generated/kubernetes-api/{{< param "version" >}}/#tcpsocketaction-v1-core)（TCP ソケット・アクション）：コンテナ IP アドレス上の特定ポートに対して TCP チェックを実行します。ポートがオープンであれば（開いていれば）、診断に成功したとみなします。

* [HTTPGetAction](/jpdocs/reference/generated/kubernetes-api/{{< param "version" >}}/#httpgetaction-v1-core)（HTTP 取得アクション）：コンテナ IP アドレス上の特定ポートとパスに対する HTTP Get （取得）リクエストを実行します。200 ～ 400 までのステータス・コードを返す応答があれば、診断に成功したとみなします。

<!--
Each probe has one of three results:
-->
各調査には３つの結果があります：

<!--
* Success: The Container passed the diagnostic.
* Failure: The Container failed the diagnostic.
* Unknown: The diagnostic failed, so no action should be taken.
-->
* Success（成功）：コンテナは診断に合格。
* Failure:(（失敗）：コンテナは診断に不合格。
* Unknown（不明）：診断が失敗したため、何も行動を起こせない。

<!--
The kubelet can optionally perform and react to two kinds of probes on running
Containers:
-->
オプションにより、kubelet は実行中のコンテナ上で、２種類の診断と対応ができます。

<!--
* `livenessProbe`: Indicates whether the Container is running. If
   the liveness probe fails, the kubelet kills the Container, and the Container
   is subjected to its [restart policy](#restart-policy). If a Container does not
   provide a liveness probe, the default state is `Success`.
-->
* `livenessProbe`（生存性診断）：コンテナが実行中かどうかを示します。もし も生存性診断に失敗すると、kubelet はコンテナを強制停止（kill）し、[再起動方針](#restart-policy) に従ってコンテナを処理します。もしもコンテナが生存性診断を提供しない場合、デフォルトン状態は `Success` （成功）です。

<!--
* `readinessProbe`: Indicates whether the Container is ready to service requests.
   If the readiness probe fails, the endpoints controller removes the Pod's IP
   address from the endpoints of all Services that match the Pod. The default
   state of readiness before the initial delay is `Failure`. If a Container does
   not provide a readiness probe, the default state is `Success`.
-->
* `readinessProbe`（読込性診断）：はコンテナがサービス・リクエストに対応できるかどうかを示します。もしも読込性診断に失敗すると、全サービスのエンドポイントに一致するポッドがあれば、エンドポイント・コントローラはポッドの IP アドレスを削除します。読込性診断のデフォルト値は `Failuer` （失敗）です。もしコンテナが読込性診断を提供しない場合、デフォルトの状態は `Success` （成功）です。

<!--
### When should you use liveness or readiness probes?
-->
### いつ生存性診断や読込性診断を使うべきですか？ {#when-should-you-use-liveness-or-readiness-probes}

<!--
If the process in your Container is able to crash on its own whenever it
encounters an issue or becomes unhealthy, you do not necessarily need a liveness
probe; the kubelet will automatically perform the correct action in accordance
with the Pod's `restartPolicy`.
-->
コンテナ内のプロセスが自分自身でクラッシュできる場合は、問題が発生すると不健全（unhealthy）になります。そのため生存性診断は不要です。言い換えれば、kubelet はポッドの `restartPolicy`（再起動方針）に従って自動的に適切な処理を行うよういします。

<!--
If you'd like your Container to be killed and restarted if a probe fails, then
specify a liveness probe, and specify a `restartPolicy` of Always or OnFailure.
-->
診断に失敗した場合、コンテナを停止して再起動したければ、生存性診断を設定し、 `restartPolicy`（再起動方針）を Always（常時） か OnFailuer （障害時）に指定します。

<!--
If you'd like to start sending traffic to a Pod only when a probe succeeds,
specify a readiness probe. In this case, the readiness probe might be the same
as the liveness probe, but the existence of the readiness probe in the spec means
that the Pod will start without receiving any traffic and only start receiving
traffic after the probe starts succeeding.
-->
ポッドに対する診断が成功する時のみトラフィックを送信開始したい場合は、読込性診断を指定します。この場合、読込性診断は生存性診断と同じように見えるかもしれません。しかし、spec 上で読込性診断の指定が存在していれば、トラフィックを受診していなくてもポッドを開始し、診断に成功した後のみトラフィックを受診するという意味があります。

<!--
If your Container needs to work on loading large data, configuration files, or migrations during startup, specify a readiness probe.
-->
もしもコンテナが大きなデータを読み込む処理を必要とする場合は、設定ファイルや起動中のマイグレーションで読込性診断を指定してください。

<!--
If you want your Container to be able to take itself down for maintenance, you
can specify a readiness probe that checks an endpoint specific to readiness that
is different from the liveness probe.
-->
コンテナが自分自身をメンテナンスでダウン可能にしたければ、読込性診断を指定します。これは、特定のエンドポイントに対する読込性の確認を設定するものであり、製造性診断とは異なります。

<!--
Note that if you just want to be able to drain requests when the Pod is deleted,
you do not necessarily need a readiness probe; on deletion, the Pod automatically
puts itself into an unready state regardless of whether the readiness probe exists.
The Pod remains in the unready state while it waits for the Containers in the Pod
to stop.
-->
ポッドの削除時にドレイン（drain：排出）要求をしたい場合は、読込性診断を指定する必要がありませんのでご注意ください。削除時は、読込性診断があるかどうかに拘わらず、ポッドが自動的に読込性の状態を unready （準備ができていない）に変えるからです。ポッド内のコンテナが待機している間中、ポッドはずっと unready のまま停止します。

<!--
For more information about how to set up a liveness or readiness probe, see
[Configure Liveness and Readiness Probes](/docs/tasks/configure-pod-container/configure-liveness-readiness-probes/).
-->
生存性診断または読込性診断をセットアップする更なる情報は、[ポッドの生存性および読込性診断](/docs/tasks/configure-pod-container/configure-liveness-readiness-probes/) をご覧ください。

<!--
## Pod and Container status
-->
## ポッドとコンテナの状態 {#pod-and-container-status}

<!--
For detailed information about Pod Container status, see
[PodStatus](/docs/reference/generated/kubernetes-api/{{< param "version" >}}/#podstatus-v1-core)
and
[ContainerStatus](/docs/reference/generated/kubernetes-api/{{< param "version" >}}/#containerstatus-v1-core).
Note that the information reported as Pod status depends on the current
[ContainerState](/docs/reference/generated/kubernetes-api/{{< param "version" >}}/#containerstatus-v1-core).
-->
ポッド・コンテナ状態に関する詳しい情報は
[PodStatus（ポッド状態）](/jp/docs/reference/generated/kubernetes-api/{{< param "version" >}}/#podstatus-v1-core)
と
[ContainerStatus（コンテナ状態）](/jp/docs/reference/generated/kubernetes-api/{{< param "version" >}}/#containerstatus-v1-core).
をご覧ください。
ポット状態として表示される情報は
[ContainerState（コンテナ状態）](/docs/reference/generated/kubernetes-api/{{< param "version" >}}/#containerstatus-v1-core)
に依存しますのでご注意ください。

<!--
## Pod readiness gate
-->
## ポッド待機ゲート（Pod readiness gate）{#pod-readiness-gate}

{{< feature-state for_k8s_version="v1.11" state="alpha" >}}

<!--
In order to add extensibility to Pod readiness by enabling the injection of
extra feedbacks or signals into `PodStatus`, Kubernetes 1.11 introduced a
feature named [Pod ready++](https://github.com/kubernetes/community/blob/master/keps/sig-network/0007-pod-ready%2B%2B.md).
You can use the new field `ReadinessGate` in the `PodSpec` to specify additional
conditions to be evaluated for Pod readiness. If Kubernetes cannot find such a
condition in the `status.conditions` field of a Pod, the status of the condition 
is default to "`False`". Below is an example:
-->
ポッドに対して拡張性を付与するため、外部のフィードバックまたはシグナルを `PodStatus` （ポッド状態）に投入できる機能、Kubernetes 1.11 では [Pod ready++](https://github.com/kubernetes/community/blob/master/keps/sig-network/0007-pod-ready%2B%2B.md) という名称で機能が導入されました。
`PodSpec` 内で新しいフィールド `ReadinessGate` （待機ゲート）を利用して、追加の状況（condition）を指定すると、ポッドの準備中に評価できるようになります。もしも Kubernetes がポッドの `status.condition` フィールド内に対象の状況（condition）がなければ、状態のデフォルトは "`False`" です。以下が例になります：


```yaml
Kind: Pod
...
spec:
  readinessGates:
    - conditionType: "www.example.com/feature-1"
status:
  conditions:
    - type: Ready  # こちらの初期の PodCondition
      status: "True"
      lastProbeTime: null
      lastTransitionTime: 2018-01-01T00:00:00Z
    - type: "www.example.com/feature-1"   # こちらは追加した PodCondition
      status: "False"
      lastProbeTIme: null
      lastTransitionTime: 2018-01-01T00:00:00Z
  containerStatuses:
    - containerID: docker://abcd...
      ready: true
...
```

<!--
The new Pod conditions must comply with Kubernetes [label key format](/docs/concepts/overview/working-with-objects/labels/#syntax-and-character-set).
Since the `kubectl patch` command still doesn't support patching object status,
the new Pod conditions have to be injected through the `PATCH` action using
one of the [KubeClient libraries](/docs/reference/using-api/client-librarie/).
-->
新しいポッド状態の指定は Kubernetel [ラベル・キー書式](/jp/docs/concepts/overview/working-with-objects/labels/#syntax-and-character-set)に従う必要があります。
`kubectl patch` コマンドはオブジェクト状態に対するパッチをまだサポートしていないため、新しい Pod 状態を設定するには、[KubeClient ライブラリ](/jp/docs/reference/using-api/client-librarie/) の１つを使い `PATCH` アクションを投入する必要があります。

<!--
With the introduction of new Pod conditions, a Pod is evaluated to be ready **only**
when both the following statements are true:
-->
新しいポッド状態を導入するには、ポッドが以下両方の状態が真（True）の時 **のみ** 評価します：

<!--
* All containers in the Pod are ready.
* All conditions specified in `ReadinessGates` are "`True`".
-->
* ポッド内のコンテナ全てが準備完了
* 全ての `ReadinessGate`（待機中ゲート）状態 が `True` に指定

<!--
To facilitate this change to Pod readiness evaluation, a new Pod condition
`ContainersReady` is introduced to capture the old Pod `Ready` condition.
-->
ポッドに対する読込性評価を調整するために、新しいポッド状態 `ContainersReady` を古いポッド状態 `Ready` から取得します。

<!--
As an alpha feature, the "Pod Ready++" feature has to be explicitly enabled by
setting the `PodReadinessGates` [feature gate](/docs/reference/command-line-tools-reference/feature-gates/)
to True.
-->
"Pod Ready++" はアルファ機能のため、利用するには `PodReadinessGates` [待機ゲート](/jp/docs/reference/command-line-tools-reference/feature-gates/) 機能を `True` に設定する必要があります。

<!--
## Restart policy
-->
## 再起動方針 {#restart-policy}

<!--
A PodSpec has a `restartPolicy` field with possible values Always, OnFailure,
and Never. The default value is Always.
`restartPolicy` applies to all Containers in the Pod. `restartPolicy` only
refers to restarts of the Containers by the kubelet on the same node. Exited
Containers that are restarted by the kubelet are restarted with an exponential
back-off delay (10s, 20s, 40s ...) capped at five minutes, and is reset after ten
minutes of successful execution. As discussed in the
[Pods document](/docs/user-guide/pods/#durability-of-pods-or-lack-thereof),
once bound to a node, a Pod will never be rebound to another node.
-->
PodSpec にある `restartPolicy` （再起動方針）フィールドの値に入るのは、Always （常に再起動）か OnFailure （障害時に再起動）か Never （再起動しない）です。デフォルトは Always です。 `restartPolicy`  が適用されるのは、ポッド内の全てのコンテナです。 `restartPolicy` はコンテナを再起動するために、同じノード上にある kubelet が参照します。終了したコンテナは kubelet によって自動的に再起動されます。この再起動（のタイミング）は簡単な指数関数的に遅延（10秒、20秒、40秒）します。上限は5分です。また実行に成功すると10分後にリセットされます。[Pod ドキュメント](/jp/docs/user-guide/pods/#durability-of-pods-or-lack-thereof) に議論があるように、ポッドはノードと共に稼働しますので、ポッドは他のノードに再配置されません。

<!--
## Pod lifetime
-->
## ポッドの生存期間（ライフタイム） {#pod-lifetime}

<!--
In general, Pods do not disappear until someone destroys them. This might be a
human or a controller. The only exception to
this rule is that Pods with a `phase` of Succeeded or Failed for more than some
duration (determined by `terminated-pod-gc-threshold` in the master) will expire and be automatically destroyed.
-->
通常、ポッドは何らかによって破棄されない限り、消滅しません。破棄するのは人間かコントローラです。この規則には唯一の例外があります。それはポッドの `phase` （段階）が成功か失敗が一定時間（マスタの `terminated-pod-gc-threshold` によって決定）経過すると、期限切れとなり自動的にポッドが破棄されます。

<!--
Three types of controllers are available:
-->
3種類のコントローラで使えます：

<!--
- Use a [Job](/docs/concepts/jobs/run-to-completion-finite-workloads/) for Pods that are expected to terminate,
  for example, batch computations. Jobs are appropriate only for Pods with
  `restartPolicy` equal to OnFailure or Never.

- Use a [ReplicationController](/docs/concepts/workloads/controllers/replicationcontroller/),
  [ReplicaSet](/docs/concepts/workloads/controllers/replicaset/), or
  [Deployment](/docs/concepts/workloads/controllers/deployment/)
  for Pods that are not expected to terminate, for example, web servers.
  ReplicationControllers are appropriate only for Pods with a `restartPolicy` of
  Always.

- Use a [DaemonSet](/docs/concepts/workloads/controllers/daemonset/) for Pods that need to run one per
  machine, because they provide a machine-specific system service.
-->

- 例えばバッチ処理などで、ポッドに [ジョブ（Job）](/jp/docs/concepts/jobs/run-to-completion-finite-workloads/) を使って停止できるようにします。ポッドに対するジョブの `restartPolicy`（再起動方針）は OnFailure（障害時のみ） か Never （再起動しない）です。

- 例えばウェブサーバで、ポッドに [ReplicationController](/jp/docs/concepts/workloads/controllers/replicationcontroller/)、[ReplicaSet](/jp/docs/concepts/workloads/controllers/replicaset/)、 [Deployment](/jp/docs/concepts/workloads/controllers/deployment/) を使い、停止しないようにします。ポッドに対する ReplicationController は `restartPolicy` が Awlays（常時）のみ適用されます。

- ポッドに [DaemonSet](/jp/docs/concepts/workloads/controllers/daemonset/) を使うのは、マシンごとにシステム・サービスを提供するので、マシンごとにポッドを実行する必要があるからです。

<!--
All three types of controllers contain a PodTemplate. It
is recommended to create the appropriate controller and let
it create Pods, rather than directly create Pods yourself. That is because Pods
alone are not resilient to machine failures, but controllers are.
-->
３種類のコントローラは PodTemplate （ポッド・テンプレート）を含みます。推奨するのは、直接自分でポッドを作成するのではなく、 適切なコントローラを作成して、コントローラにポッドを作成させる方法です。これはマシン障害発生時にポッドには回復力はありません。しかし、コントローラには回復力があるからです。

<!--
If a node dies or is disconnected from the rest of the cluster, Kubernetes
applies a policy for setting the `phase` of all Pods on the lost node to Failed.
-->
ノードが停止するかクラスタから切り離されると、Kubernetes は対象となる全てのポッドの `phase` を Failed（障害）に変更します。

<!--
## Examples
-->
## 例 {#examples}

<!--
### Advanced liveness probe example
-->
### 高度な生存性診断の例 {#advanced-liveness-probe-example}

<!--
Liveness probes are executed by the kubelet, so all requests are made in the
kubelet network namespace.
-->
生存性診断は kubelet によって実行されるため、kubelet 名前空間の中で全ての要求が作成されます。

```yaml
apiVersion: v1
kind: Pod
metadata:
  labels:
    test: liveness
  name: liveness-http
spec:
  containers:
  - args:
    - /server
    image: k8s.gcr.io/liveness
    livenessProbe:
      httpGet:
        # when "host" is not defined, "PodIP" will be used
        # host: my-host
        # when "scheme" is not defined, "HTTP" scheme will be used. Only "HTTP" and "HTTPS" are allowed
        # scheme: HTTPS
        path: /healthz
        port: 8080
        httpHeaders:
        - name: X-Custom-Header
          value: Awesome
      initialDelaySeconds: 15
      timeoutSeconds: 1
    name: liveness
```

<!--
### Example states
-->
### 状態の例 {#example-states}

<!--
   * Pod is running and has one Container. Container exits with success.
     * Log completion event.
     * If `restartPolicy` is:
       * Always: Restart Container; Pod `phase` stays Running.
       * OnFailure: Pod `phase` becomes Succeeded.
       * Never: Pod `phase` becomes Succeeded.

   * Pod is running and has one Container. Container exits with failure.
     * Log failure event.
     * If `restartPolicy` is:
       * Always: Restart Container; Pod `phase` stays Running.
       * OnFailure: Restart Container; Pod `phase` stays Running.
       * Never: Pod `phase` becomes Failed.

   * Pod is running and has two Containers. Container 1 exits with failure.
     * Log failure event.
     * If `restartPolicy` is:
       * Always: Restart Container; Pod `phase` stays Running.
       * OnFailure: Restart Container; Pod `phase` stays Running.
       * Never: Do not restart Container; Pod `phase` stays Running.
     * If Container 1 is not running, and Container 2 exits:
       * Log failure event.
       * If `restartPolicy` is:
         * Always: Restart Container; Pod `phase` stays Running.
         * OnFailure: Restart Container; Pod `phase` stays Running.
         * Never: Pod `phase` becomes Failed.

   * Pod is running and has one Container. Container runs out of memory.
     * Container terminates in failure.
     * Log OOM event.
     * If `restartPolicy` is:
       * Always: Restart Container; Pod `phase` stays Running.
       * OnFailure: Restart Container; Pod `phase` stays Running.
       * Never: Log failure event; Pod `phase` becomes Failed.

   * Pod is running, and a disk dies.
     * Kill all Containers.
     * Log appropriate event.
     * Pod `phase` becomes Failed.
     * If running under a controller, Pod is recreated elsewhere.

   * Pod is running, and its node is segmented out.
     * Node controller waits for timeout.
     * Node controller sets Pod `phase` to Failed.
     * If running under a controller, Pod is recreated elsewhere.
-->
   * ポッドを実行中で１つのコンテナがある。コンテナの終了が成功する。
     * 完了したイベントを記録する。
     * もし `restartPolicy` （再起動方針）が：
       * Always（常時）の場合：コンテナを再起動し、ポッドの `phase` は Running（実行中）のまま。
       * OnFailure（障害時に再起動）の場合：ポッドの `phase`  は Succeeded（成功）になる。
       * Never（再起動しない）の場合：ポッドの `phase`  は Succeeded（成功）になる。

   * ポッドを実行中で１つのコンテナがある。コンテナの終了が失敗する。
     * 失敗したイベントを記録する。
     * もし `restartPolicy` （再起動方針）が：
       * Always（常時）の場合：コンテナを再起動し、ポッドの `phase` は Running（実行中）のまま。
       * OnFailure（障害時に再起動）の場合：コンテナを再起動し、ポッドの `phase` は Running（実行中）のまま。
       * Never（再起動しない）の場合：ポッドの `phase`  は Failed（失敗）になる。

   * ポッドを実行中で２つのコンテナがある。コンテナ１の終了が失敗する。
     * 失敗したイベントを記録する。
     * もし `restartPolicy` （再起動方針）が：
       * Always（常時）の場合：コンテナを再起動し、ポッドの `phase` は Running（実行中）のまま。
       * OnFailure（障害時に再起動）の場合：コンテナを再起動し、ポッドの `phase` は Running（実行中）にまま。
       * Never（再起動しない）の場合：コンテナを再起動せず、ポッドの `phase`  は Running（実行中）のまま。
     * もしコンテナ１が実行中ではなく、コンテナ２が終了した場合：
     * 失敗したイベントを記録する。
     * もし `restartPolicy` （再起動方針）が：
       * Always（常時）の場合：コンテナを再起動し、ポッドの `phase` は Running（実行中）のまま。
       * OnFailure（障害時に再起動）の場合：コンテナを再起動し、ポッドの `phase` は Running（実行中）のまま。
       * Never（再起動しない）の場合：ポッドの `phase`  は Failed（失敗）になる。

   * ポッドを実行中で１つのコンテナがある。コンテナでメモリ不足が発生する。
     * コンテナは障害によって停止（terminate）する。
     * OOM イベントを記録する。
     * もし `restartPolicy` （再起動方針）が：
       * Always（常時）の場合：コンテナを再起動し、ポッドの `phase` は Running（実行中）のまま。
       * OnFailure（障害時に再起動）の場合：コンテナを再起動し、ポッドの `phase` は Running（実行中）のまま。
       * Never（再起動しない）の場合：イベントの失敗を記録し、ポッドの `phase`  は Failed（失敗）になる。

   * ポッドを実行中で、ディスクが故障する。
     * 全てのコンテナを停止。
     * 適切なイベントを記録。
     * ポッドの `phase` は Failed（障害）になる。
     * コントローラが稼働中であれば、ポッドをどこかで再作成する。

   * ポッドが実行中で、ノードがセグメント・アウトする。
     * ノード・コントローラはタイムアウトを待つ。
     * ノード・コントローラはポッドの `phase` を Failed （失敗）に設定。
     * コントローラが稼働中であれば、ポッドをどこかで再作成する。

{{% /capture %}}


{{% capture whatsnext %}}

<!--
* Get hands-on experience
  [attaching handlers to Container lifecycle events](/docs/tasks/configure-pod-container/attach-handler-lifecycle-event/).

* Get hands-on experience
  [configuring liveness and readiness probes](/docs/tasks/configure-pod-container/configure-liveness-readiness-probes/).

* Learn more about [Container lifecycle hooks](/docs/concepts/containers/container-lifecycle-hooks/).
-->
* ハンズオンを試す  [コンテナ・ライフサイクル・イベントの扱いを割り当て](/jp/docs/tasks/configure-pod-container/attach-handler-lifecycle-event/).

* ハンズオンを試す
  [生存性または読込性診断の設定](/jp/docs/tasks/configure-pod-container/configure-liveness-readiness-probes/).

* [コンテナ・ライフサイクル・フックs](/jp/docs/concepts/containers/container-lifecycle-hooks/) について学ぶ。


{{% /capture %}}



