---
reviewers:
- davidopp
- kevin-wangzefeng
- bsalamat
title: ノードにポッドを割り当て
content_template: templates/concept
weight: 30
---

{{< toc >}}

{{% capture overview %}}

<!--
You can constrain a [pod](/docs/concepts/workloads/pods/pod/) to only be able to run on particular [nodes](/docs/concepts/architecture/nodes/) or to prefer to
run on particular nodes. There are several ways to do this, and they all use
[label selectors](/docs/concepts/overview/working-with-objects/labels/) to make the selection.
Generally such constraints are unnecessary, as the scheduler will automatically do a reasonable placement
(e.g. spread your pods across nodes, not place the pod on a node with insufficient free resources, etc.)
but there are some circumstances where you may want more control on a node where a pod lands, e.g. to ensure
that a pod ends up on a machine with an SSD attached to it, or to co-locate pods from two different
services that communicate a lot into the same availability zone.
-->
特定の [ノード](/jp/docs/concepts/architecture/nodes/) 上での[ポッド](/jp/docs/concepts/workloads/pods/pod/) の実行制限や、詳細を指定したノード上での実行が望ましいといった制限が可能です。
制限をするにはいくつかの方法があります。また、選択にあたっては全て [ラベル・セレクタ（label selector）](/docs/concepts/overview/working-with-objects/labels/) を使います。
一般的に、このような制限は不要です。
スケジューラは適切かつ自動的に配置します（例：ノード全体にポッドを展開しますが、十分な空きリソースがないなどのノードにはポッドを置きません）。
しかし、いくつかの事情があれば、ポッドを置くためのノードをさらに制御したい場合など、確実に指定したい場合、例えば最終的には SSD のあるマシンで起動するためや、同じアベイラビリティ・ゾーンのなかで、異なる２つのサービスを一緒のポッドにして通信したい場合などがあります。

<!--
You can find all the files for these examples [in our docs
repo here](https://github.com/kubernetes/website/tree/{{< param "docsbranch" >}}/content/en/docs/concepts/configuration/).
-->
これらのサンプルに関するファイル全ては、 [こちらのドキュメント・リポジトリ](https://github.com/kubernetes/website/tree/{{< param "docsbranch" >}}/content/en/docs/concepts/configuration/) にあります。
{{% /capture %}}

{{% capture body %}}

## nodeSelector（ノード・セレクタ） {#nodeselector}

<!--
`nodeSelector` is the simplest form of constraint.
`nodeSelector` is a field of PodSpec. It specifies a map of key-value pairs. For the pod to be eligible
to run on a node, the node must have each of the indicated key-value pairs as labels (it can have
additional labels as well). The most common usage is one key-value pair.
-->
`nodeSelector` は最も簡単な制約（constraint）です。
`nodeSelector` は PodSpec のフィールドです。
キー・バリューのペアを指定します。
ポッドに対しては実行に適しているノードを指定します。
対象のノードを示すのは、ノードのラベルが持っているキーバリューのペアです（ラベルの追加も可能です）。
最も一般的な使い方は、１つのキーバリューのペアです。

<!--
Let's walk through an example of how to use `nodeSelector`.
-->
`nodeSelector` の使い方の例をみていきましょう。

<!--
### Step Zero: Prerequisites
-->
### ステップ０：準備 {#step-zero-prerequisites}

<!--
This example assumes that you have a basic understanding of Kubernetes pods and that you have [turned up a Kubernetes cluster](https://github.com/kubernetes/kubernetes#documentation).
-->
この例では、Kubernetes ポッドの基本を理解し、[Kubernetes クラスタ](https://github.com/kubernetes/kubernetes#documentation)を起動している前提です。

<!--
### Step One: Attach label to the node
-->
### ステップ１：ノードにラベルを割り当て（アタッチ） {#step-one-attach-label-to-the-node}

<!--
Run `kubectl get nodes` to get the names of your cluster's nodes. Pick out the one that you want to add a label to, and then run `kubectl label nodes <node-name> <label-key>=<label-value>` to add a label to the node you've chosen. For example, if my node name is 'kubernetes-foo-node-1.c.a-robinson.internal' and my desired label is 'disktype=ssd', then I can run `kubectl label nodes kubernetes-foo-node-1.c.a-robinson.internal disktype=ssd`.
-->
`kubectl get nodes` を実行し、クラスタのノード名を取得します。
ラベルを割り当てたい１つを選びます。
それから選択したノードに対してラベルを割り当てるために `kubectl label nodes <node-name> <label-key>=<label-value>` を実行します。
たとえば、ノード名が `kubernetes-foo-node-1.c.a-robinson.internal` で付けたいラベルが  'disktype=ssd' であれば、`kubectl label nodes kubernetes-foo-node-1.c.a-robinson.internal disktype=ssd` を実行します。

<!--
If this fails with an "invalid command" error, you're likely using an older version of kubectl that doesn't have the `label` command. In that case, see the [previous version](https://github.com/kubernetes/kubernetes/blob/a053dbc313572ed60d89dae9821ecab8bfd676dc/examples/node-selection/README.md) of this guide for instructions on how to manually set labels on a node.
-->
"invalid command" （無効なコマンド）のエラーが出る場合は、もしかすると古いバージョンの kubectl を使っているかもしれません。
その場合は、ノードに対して手動でラベルを割り当てるために、[以前のバージョン](https://github.com/kubernetes/kubernetes/blob/a053dbc313572ed60d89dae9821ecab8bfd676dc/examples/node-selection/README.md) のガイドをご覧ください。

<!--
You can verify that it worked by re-running `kubectl get nodes --show-labels` and checking that the node now has a label.
-->
再び `kubectl get nodes --show-labels` が実行できるかどうかを確認し、ノードに対してラベルが割り当てられているかを確認します。

<!--
### Step Two: Add a nodeSelector field to your pod configuration
-->
### ステップ２：ポッド設定に nodeSelector フィールドを追加 {#step-two-add-a-nodeselector-field-to-your-pod-configuration}

<!--
Take whatever pod config file you want to run, and add a nodeSelector section to it, like this. For example, if this is my pod config:
-->
実行したいポッド設定情報ファイルに nodeSelector セクションを追加し、以下の例のようにします。こちらは私の設定情報です：


```yaml
apiVersion: v1
kind: Pod
metadata:
  name: nginx
  labels:
    env: test
spec:
  containers:
  - name: nginx
    image: nginx
```

<!--
Then add a nodeSelector like so:
-->
そこに、このような nodeSelector を追加します。

{{< codenew file="pods/pod-nginx.yaml" >}}

<!--
When you then run `kubectl create -f https://k8s.io/examples/pods/pod-nginx.yaml`,
the Pod will get scheduled on the node that you attached the label to. You can
verify that it worked by running `kubectl get pods -o wide` and looking at the
"NODE" that the Pod was assigned to.
-->
`kubectl create -f https://k8s.io/examples/pods/pod-nginx.yaml` を実行すると、ラベルを割り当てたら対象ノード上に、ポッドがスケジュールされます。
`kubectl get pods -o wide` を実行し、 「NODE」を探してポッドが割り当てられたところにあるかどうか確認します。

<!--
## Interlude: built-in node labels
-->
## 休憩：内蔵ノード・ラベル {#interlude-built-in-node-labels}

<!--
In addition to labels you [attach](#step-one-attach-label-to-the-node), nodes come pre-populated
with a standard set of labels. As of Kubernetes v1.4 these labels are
-->
ラベルの [割り当て](#step-one-attach-label-to-the-node) に加え、ノードには事前割り当て済みの標準的なラベル群があります。
Kubernetes v1.4 では、ｋのようなラベルがあります。

* `kubernetes.io/hostname`
* `failure-domain.beta.kubernetes.io/zone`
* `failure-domain.beta.kubernetes.io/region`
* `beta.kubernetes.io/instance-type`
* `beta.kubernetes.io/os`
* `beta.kubernetes.io/arch`

{{< note >}}
<!--
**Note:** The value of these labels is cloud provider specific and is not guaranteed to be reliable.
For example, the value of `kubernetes.io/hostname` may be the same as the Node name in some environments
and a different value in other environments.
-->
**メモ：** これらのラベルはクラウド事業者固有であり、確実性を保証するものではありません。
たとえば、`kubernetes.io/hostname` が環境によってはノード名と同じ場合も有りますし、別の環境では値が異なる場合があります。
{{< /note >}}

<!--
## Affinity and anti-affinity
-->
## 親和性と非親和性 {#affinity-and-anti-affinity}

<!--
`nodeSelector` provides a very simple way to constrain pods to nodes with particular labels. The affinity/anti-affinity
feature, currently in beta, greatly expands the types of constraints you can express. The key enhancements are
-->
以前までは、ノードで動くポッドを制限するために、`nodeSelector` によってノードに細かくラベル付けするのが非常に簡単な手法でした。
現時点ではベータですが、親和性（affinity）と非親和性（anti-affinity）の機能によって、制約に関する様々な拡張が可能になります。
主な拡張点は以下の通りです。

<!--
1. the language is more expressive (not just "AND of exact match")
2. you can indicate that the rule is "soft"/"preference" rather than a hard requirement, so if the scheduler
   can't satisfy it, the pod will still be scheduled
3. you can constrain against labels on other pods running on the node (or other topological domain),
   rather than against labels on the node itself, which allows rules about which pods can and cannot be co-located
-->
1. より表現が豊かな言語（ AND の正確な一致だけではありません ）
2. 厳密な条件定義ではなく「ソフト（soft）」や「優先権（preference）」のようなルールを指定できるので、スケジューラでは満たせられない条件も、ポッドに対してスケジュールできる
3. ノード自身に対してラベルを割り当てられるだけでなく、ノード上で（あるいは他のネットワークに属する領域で）実行している他のポッドに対してもラベルで制限できるので、一緒の場所になりポッドに対してもルールを追加できる

<!--
The affinity feature consists of two types of affinity, "node affinity" and "inter-pod affinity/anti-affinity".
Node affinity is like the existing `nodeSelector` (but with the first two benefits listed above),
while inter-pod affinity/anti-affinity constrains against pod labels rather than node labels, as
described in the third item listed above, in addition to having the first and second properties listed above.
-->
親和性機能を構成するのは、２つの親和性、「ノード親和性（node affinity）」と「ポッド間親和性/非親和性」です。
ノード親和性は従来の `nodeSelector` のようなものです（しかし、先ほど挙げた１番目と２番目の利点があります）。
ノードのラベルというよりは、ポッドに対するラベルによって、ポッド間の親和性と非親和性を指定するもので、先ほどのリストにある３番目に説明した要素であり、１番目と２番目の属性に追加されるものです。

<!--
`nodeSelector` continues to work as usual, but will eventually be deprecated, as node affinity can express
everything that `nodeSelector` can express.
-->
`nodeSelector` は従来通り動作します。
しかし、いずれは非推奨となりますので、 `nodeSelector` で指定しているすべてをノード親和性に置き換えることになります。

<!--
### Node affinity (beta feature)
-->
### ノード親和性（ベータ機能） {#node-affinity-beta-feature}

<!--
Node affinity was introduced as alpha in Kubernetes 1.2.
Node affinity is conceptually similar to `nodeSelector` -- it allows you to constrain which nodes your
pod is eligible to be scheduled on, based on labels on the node.
-->
ノード親和性が Kubernetes 1.2 でアルファとして導入されました。
ノード親和性は概念的に `nodeSelector` と似ています。これは、ノード上のラベルに藻度図居て、ポッドを実行するに相応しいノードを制限できるようにします。

<!--
There are currently two types of node affinity, called `requiredDuringSchedulingIgnoredDuringExecution` and
`preferredDuringSchedulingIgnoredDuringExecution`. You can think of them as "hard" and "soft" respectively,
in the sense that the former specifies rules that *must* be met for a pod to be scheduled onto a node (just like
`nodeSelector` but using a more expressive syntax), while the latter specifies *preferences* that the scheduler
will try to enforce but will not guarantee. The "IgnoredDuringExecution" part of the names means that, similar
to how `nodeSelector` works, if labels on a node change at runtime such that the affinity rules on a pod are no longer
met, the pod will still continue to run on the node. In the future we plan to offer
`requiredDuringSchedulingRequiredDuringExecution` which will be just like `requiredDuringSchedulingIgnoredDuringExecution`
except that it will evict pods from nodes that cease to satisfy the pods' node affinity requirements.
-->
現時点では、ノード親和性には２つのタイプ（種類）があります。
`requiredDuringSchedulingIgnoredDuringExecution` と`preferredDuringSchedulingIgnoredDuringExecution` と呼びます。
これらは「ハード」と「ソフト」を表します。
原則的として仕様ルールでは、ノードに対するスケジュールは「必ず」実施します（`nodeSelector` のようですが、より表現豊かな構文です）。
*preferences* （望ましいの意味）の文字が示すのは、スケジューラに対して試みさせるものの、保証しません。
名前の *IgnoredDuringExecution* の部分が意味するのは、 `nodeSelector` の動作と似ており、ノード上のラベルが実行中に変更となり、ポッドに対する親和性のルールが一致しなくなった場合でも、ノード上のポッドは実行し続けるようにします。
将来的な `requiredDuringSchedulingRequiredDuringExecution` で計画しているのは、ノードを退去するために `requiredDuringSchedulingIgnoredDuringExecution` のノード親和性要件を満たすことです。

<!--
Thus an example of `requiredDuringSchedulingIgnoredDuringExecution` would be "only run the pod on nodes with Intel CPUs"
and an example `preferredDuringSchedulingIgnoredDuringExecution` would be "try to run this set of pods in availability
zone XYZ, but if it's not possible, then allow some to run elsewhere".
-->
従って例の `requiredDuringSchedulingIgnoredDuringExecution` で「intel GPUを持つノード上でのみポッドを実行する」を指定するには、例の `preferredDuringSchedulingIgnoredDuringExecution` による指定は「アベイラビリティ・ゾーン XYZ でポッド・セットの実行を試みるものの、不可能な場合はどこでも実行可能にする」となります。

<!--
Node affinity is specified as field `nodeAffinity` of field `affinity` in the PodSpec.
-->
ノード親和性を指定するのは、PodSpec 内の `affinity` （親和性）フィールド内の `nodeAffinity` （ノード親和性）です。

<!_-
Here's an example of a pod that uses node affinity:
-->
こちらにあるのはノード親和性を使うポッドの例です。

{{< codenew file="pods/pod-with-node-affinity.yaml" >}}

<!--
This node affinity rule says the pod can only be placed on a node with a label whose key is
`kubernetes.io/e2e-az-name` and whose value is either `e2e-az1` or `e2e-az2`. In addition,
among nodes that meet that criteria, nodes with a label whose key is `another-node-label-key` and whose
value is `another-node-label-value` should be preferred.
-->
このノード親和性ルールは、ポッドを置けるノードは、ラベルのキーが `kubernetes.io/e2e-az-name` であり、値が `e2e-az1` か `e2e-az2` のどちらかです。
さらに、基準を満たすのに望ましいノードは、 `another-node-label-key`  のキーと、`another-node-label-value` の値を持ちます。

<!--
You can see the operator `In` being used in the example. The new node affinity syntax supports the following operators: `In`, `NotIn`, `Exists`, `DoesNotExist`, `Gt`, `Lt`.
You can use `NotIn` and `DoesNotExist` to achieve node anti-affinity behavior, or use 
[node taints](/docs/concepts/configuration/taint-and-toleration/) to repel pods from specific nodes.
-->
サンプル中に `In` 演算子を使っているのがみえます。
新しいノード親和性構文がサポートするのは、次の演算子です：。`In`, `NotIn`, `Exists`、 `DoesNotExist`、 `Gt`、 `Lt` 。
ノード非親和性を達成するには `NotIn` と `DoesNotExist`が使えます。
あるいは、[ノード故障（node taint）](/jp/docs/concepts/configuration/taint-and-toleration/) で特定のノード上のポッドを再度ラベル付けします。

<!--
If you specify both `nodeSelector` and `nodeAffinity`, *both* must be satisfied for the pod
to be scheduled onto a candidate node.
-->
`nodeSelector` と `nodeAffinity` の両方を指定すると、  *両方* を満たすノード候補に対してポッドがスケジュールされます。

<!--
If you specify multiple `nodeSelectorTerms` associated with `nodeAffinity` types, then the pod can be scheduled onto a node **if one of** the `nodeSelectorTerms` is satisfied.
-->
`nodeAffinity` タイプとして `nodeSelectorTerms` を複数指定すると、`nodeSelectorTerms` を満たす **ノードの１つ** に対してポッドをスケジュールします。

<!_-
If you specify multiple `matchExpressions` associated with `nodeSelectorTerms`, then the pod can be scheduled onto a node **only if all** `matchExpressions` can be satisfied.
-->
`nodeSelectorTerms` に関連して `matchExpressions` を複数指定すると、 `matchExpressions` を満たす **すべてのノード** に対してポッドをスケジュールします。

<!--
If you remove or change the label of the node where the pod is scheduled, the pod won't be removed. In other words, the affinity selection works only at the time of scheduling the pod.
-->
ポッドがスケジュールされたノードに対するラベルを削除または変更しても、ポッドは削除されません。
言い換えれば、親和性による選択が機能するのは、ポッドのスケジュール時のみです。

<!--
The `weight` field in `preferredDuringSchedulingIgnoredDuringExecution` is in the range 1-100. For each node that meets all of the scheduling requirements (resource request, RequiredDuringScheduling affinity expressions, etc.), the scheduler will compute a sum by iterating through the elements of this field and adding "weight" to the sum if the node matches the corresponding MatchExpressions. This score is then combined with the scores of other priority functions for the node. The node(s) with the highest total score are the most preferred.
-->
`preferredDuringSchedulingIgnoredDuringExecution` フィールド内の `weight` の範囲は 1～100 です。
各ノードがスケジュール要件の全てを満たしている場合（リソース要求、RequiredDuringScheduling、親和性の指定、など）、スケジューラは一致する上限に適するノードがあれば、「weight」の各要素の合計値を計算します。
結果（スコア）は各ノードの優先度を計算するために連結されます。
最も高い結果を持つノードが最も適しています。

<!--
For more information on node affinity, see the 
[design doc](https://git.k8s.io/community/contributors/design-proposals/scheduling/nodeaffinity.md).
-->
ノード親和性に関する詳しい情報は、[設計ドキュメント](https://git.k8s.io/community/contributors/design-proposals/scheduling/nodeaffinity.md) をご覧ください。

<!--
### Inter-pod affinity and anti-affinity (beta feature)
-->
### ポッド間親和性と非親和性（ベータ機能） {#inter-pop-affinity-and-anti-affinity-beta-feature}

<!--
Inter-pod affinity and anti-affinity were introduced in Kubernetes 1.4.
Inter-pod affinity and anti-affinity allow you to constrain which nodes your pod is eligible to be scheduled *based on
labels on pods that are already running on the node* rather than based on labels on nodes. The rules are of the form "this pod should (or, in the case of
anti-affinity, should not) run in an X if that X is already running one or more pods that meet rule Y". Y is expressed
as a LabelSelector with an associated list of namespaces (or "all" namespaces); unlike nodes, because pods are namespaced
(and therefore the labels on pods are implicitly namespaced),
a label selector over pod labels must specify which namespaces the selector should apply to. Conceptually X is a topology domain
like node, rack, cloud provider zone, cloud provider region, etc. You express it using a `topologyKey` which is the
key for the node label that the system uses to denote such a topology domain, e.g. see the label keys listed above
in the section [Interlude: built-in node labels](#interlude-built-in-node-labels).
-->
ポッド間親和性（inter-pod affinity）と非親和性（anti-affinity）は Kubernetes 1.4 で導入されました。
ポッド間親和性と非親和性によって、ポッドをスケジュールするのに相応しいノードを制限するにあたり、ノード上のラベルに基づくのではなく、*既にノード上で実行しているポッドのラベルに基づいて行います* 。
このルールの形式は「このポッドは X 上実行すべき（場合によっては、実行すべきではない）です。たルール Y に基づくポッドが１つまたは複数が動いている場合」のようになります。
Y を表すのは、名前空間のリスト（あるは「すべての」名前空間）と関連付けられた LabelSelector（ラベル・セレクタ）です。
つまり、ノードではありません。なぜならば、ポッドは名前空間内にあり（つまり、ポッドは暗黙的に名前空間内にラベル付けされます）、ラベル・セレクタは、ポッド・ラベルよりも、セレクタが適用すべき名前空間を指定するために利用します。
概念としては X はノード、ラック、クラウド事業者のゾーン、クラウド事業者のリージョン等のようなトポロジー領域です。
`topologyKey` を使って表現するのは、ノード・ラベルのキーとして使うためであり、これはシステムにおけるトポロジー・ドメイン（領域）を表すためです。
[内蔵ノード・ラベル](#interlude-built-in-node-labels) のセクション内に、ラベル・キーに関する例があります。

<!--
**Note:** Inter-pod affinity and anti-affinity require substantial amount of 
processing which can slow down scheduling in large clusters significantly. We do
not recommend using them in clusters larger than several hundred nodes.
-->
**メモ：** ポッド間親和性と非親和性は、非常に大きなクラスタで使うと、十分な量の処理がなければ処理遅延を引き起こします。数百を越えるノードを持つクラスタ上での利用は推奨しません。


<!--
As with node affinity, there are currently two types of pod affinity and anti-affinity, called `requiredDuringSchedulingIgnoredDuringExecution` and
`preferredDuringSchedulingIgnoredDuringExecution` which denote "hard" vs. "soft" requirements.
See the description in the node affinity section earlier.
An example of `requiredDuringSchedulingIgnoredDuringExecution` affinity would be "co-locate the pods of service A and service B
in the same zone, since they communicate a lot with each other"
and an example `preferredDuringSchedulingIgnoredDuringExecution` anti-affinity would be "spread the pods from this service across zones"
(a hard requirement wouldn't make sense, since you probably have more pods than zones).
-->
ノード親和性と同様に、ポッド親和性と非親和性には２種類あります。
`requiredDuringSchedulingIgnoredDuringExecution` と `preferredDuringSchedulingIgnoredDuringExecution` は「ハード」と「ソフト」要求に相当します。
説明については前方のノード親和性のセクションをご覧ください。
`requiredDuringSchedulingIgnoredDuringExecution` 親和性の例は「サービス A とサービス B が同じゾーンにあり、ポッドが一緒に動きながら、お互いに頻繁に通信する」であり、 `preferredDuringSchedulingIgnoredDuringExecution` 非親和性は「サービスを横断するゾーンからは、ポッドを離す」（ポッドが複数のゾーンに存在しないため、ハード要求が一致しない）です。

<!--
Inter-pod affinity is specified as field `podAffinity` of field `affinity` in the PodSpec.
And inter-pod anti-affinity is specified as field `podAntiAffinity` of field `affinity` in the PodSpec.
-->
ポッド間親和性は PodSpec 内の `affinity` フィールドの `podAffinity` フィールドで指定します。
また、ポッド間非親和性はPodSpec 内の `affinity` フィールドの `podAntiAffinity` フィールドで指定します。

<!--
#### An example of a pod that uses pod affinity:
-->
#### ポッドがポッド親和性を使う例： {#an-example-of-a-pod-that-uses-pod-affinity}

{{< codenew file="pods/pod-with-pod-affinity.yaml" >}}

The affinity on this pod defines one pod affinity rule and one pod anti-affinity rule. In this example, the
`podAffinity` is `requiredDuringSchedulingIgnoredDuringExecution`
while the `podAntiAffinity` is `preferredDuringSchedulingIgnoredDuringExecution`. The
pod affinity rule says that the pod can be scheduled onto a node only if that node is in the same zone
as at least one already-running pod that has a label with key "security" and value "S1". (More precisely, the pod is eligible to run
on node N if node N has a label with key `failure-domain.beta.kubernetes.io/zone` and some value V
such that there is at least one node in the cluster with key `failure-domain.beta.kubernetes.io/zone` and
value V that is running a pod that has a label with key "security" and value "S1".) The pod anti-affinity
rule says that the pod prefers not to be scheduled onto a node if that node is already running a pod with label
having key "security" and value "S2". (If the `topologyKey` were `failure-domain.beta.kubernetes.io/zone` then
it would mean that the pod cannot be scheduled onto a node if that node is in the same zone as a pod with
label having key "security" and value "S2".) See the 
[design doc](https://git.k8s.io/community/contributors/design-proposals/scheduling/podaffinity.md)
for many more examples of pod affinity and anti-affinity, both the `requiredDuringSchedulingIgnoredDuringExecution`
flavor and the `preferredDuringSchedulingIgnoredDuringExecution` flavor.

The legal operators for pod affinity and anti-affinity are `In`, `NotIn`, `Exists`, `DoesNotExist`.

In principle, the `topologyKey` can be any legal label-key. However,
for performance and security reasons, there are some constraints on topologyKey:

1. For affinity and for `requiredDuringSchedulingIgnoredDuringExecution` pod anti-affinity,
empty `topologyKey` is not allowed.
2. For `requiredDuringSchedulingIgnoredDuringExecution` pod anti-affinity, the admission controller `LimitPodHardAntiAffinityTopology` was introduced to limit `topologyKey` to `kubernetes.io/hostname`. If you want to make it available for custom topologies, you may modify the admission controller, or simply disable it.
3. For `preferredDuringSchedulingIgnoredDuringExecution` pod anti-affinity, empty `topologyKey` is interpreted as "all topologies" ("all topologies" here is now limited to the combination of `kubernetes.io/hostname`, `failure-domain.beta.kubernetes.io/zone` and `failure-domain.beta.kubernetes.io/region`).
4. Except for the above cases, the `topologyKey` can be any legal label-key.

In addition to `labelSelector` and `topologyKey`, you can optionally specify a list `namespaces`
of namespaces which the `labelSelector` should match against (this goes at the same level of the definition as `labelSelector` and `topologyKey`).
If omitted, it defaults to the namespace of the pod where the affinity/anti-affinity definition appears.
If defined but empty, it means "all namespaces".

All `matchExpressions` associated with `requiredDuringSchedulingIgnoredDuringExecution` affinity and anti-affinity
must be satisfied for the pod to be scheduled onto a node.

#### More Practical Use-cases

Interpod Affinity and AntiAffinity can be even more useful when they are used with higher
level collections such as ReplicaSets, StatefulSets, Deployments, etc.  One can easily configure that a set of workloads should
be co-located in the same defined topology, eg., the same node.

##### Always co-located in the same node

In a three node cluster, a web application has in-memory cache such as redis. We want the web-servers to be co-located with the cache as much as possible.
Here is the yaml snippet of a simple redis deployment with three replicas and selector label `app=store`. The deployment has `PodAntiAffinity` configured to ensure the scheduler does not co-locate replicas on a single node.

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: redis-cache
spec:
  selector:
    matchLabels:
      app: store
  replicas: 3
  template:
    metadata:
      labels:
        app: store
    spec:
      affinity:
        podAntiAffinity:
          requiredDuringSchedulingIgnoredDuringExecution:
          - labelSelector:
              matchExpressions:
              - key: app
                operator: In
                values:
                - store
            topologyKey: "kubernetes.io/hostname"
      containers:
      - name: redis-server
        image: redis:3.2-alpine
```

The below yaml snippet of the webserver deployment has `podAntiAffinity` and `podAffinity` configured. This informs the scheduler that all its replicas are to be co-located with pods that have selector label `app=store`. This will also ensure that each web-server replica does not co-locate on a single node.

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: web-server
spec:
  selector:
    matchLabels:
      app: web-store
  replicas: 3
  template:
    metadata:
      labels:
        app: web-store
    spec:
      affinity:
        podAntiAffinity:
          requiredDuringSchedulingIgnoredDuringExecution:
          - labelSelector:
              matchExpressions:
              - key: app
                operator: In
                values:
                - web-store
            topologyKey: "kubernetes.io/hostname"
        podAffinity:
          requiredDuringSchedulingIgnoredDuringExecution:
          - labelSelector:
              matchExpressions:
              - key: app
                operator: In
                values:
                - store
            topologyKey: "kubernetes.io/hostname"
      containers:
      - name: web-app
        image: nginx:1.12-alpine
```

If we create the above two deployments, our three node cluster should look like below.

|       node-1         |       node-2        |       node-3       |
|:--------------------:|:-------------------:|:------------------:|
| *webserver-1*        |   *webserver-2*     |    *webserver-3*   |
|  *cache-1*           |     *cache-2*       |     *cache-3*      |

As you can see, all the 3 replicas of the `web-server` are automatically co-located with the cache as expected.

```
$ kubectl get pods -o wide
NAME                           READY     STATUS    RESTARTS   AGE       IP           NODE
redis-cache-1450370735-6dzlj   1/1       Running   0          8m        10.192.4.2   kube-node-3
redis-cache-1450370735-j2j96   1/1       Running   0          8m        10.192.2.2   kube-node-1
redis-cache-1450370735-z73mh   1/1       Running   0          8m        10.192.3.1   kube-node-2
web-server-1287567482-5d4dz    1/1       Running   0          7m        10.192.2.3   kube-node-1
web-server-1287567482-6f7v5    1/1       Running   0          7m        10.192.4.3   kube-node-3
web-server-1287567482-s330j    1/1       Running   0          7m        10.192.3.2   kube-node-2
```

##### Never co-located in the same node

The above example uses `PodAntiAffinity` rule with `topologyKey: "kubernetes.io/hostname"` to deploy the redis cluster so that 
no two instances are located on the same host. 
See [ZooKeeper tutorial](/docs/tutorials/stateful-application/zookeeper/#tolerating-node-failure) 
for an example of a StatefulSet configured with anti-affinity for high availability, using the same technique.

For more information on inter-pod affinity/anti-affinity, see the 
[design doc](https://git.k8s.io/community/contributors/design-proposals/scheduling/podaffinity.md).

You may want to check [Taints](/docs/concepts/configuration/taint-and-toleration/)
as well, which allow a *node* to *repel* a set of pods.

{{% /capture %}}

{{% capture whatsnext %}}

{{% /capture %}}
