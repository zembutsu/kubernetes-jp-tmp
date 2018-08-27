---
reviewers:
- davidopp
- kevin-wangzefeng
- bsalamat
title: 劣化（テイント）と許容（トレレイト）
content_template: templates/concept
weight: 40
---

{{< toc >}}

{{% capture overview %}}
<!--
Node affinity, described [here](/docs/concepts/configuration/assign-pod-node/#node-affinity-beta-feature),
is a property of *pods* that *attracts* them to a set of nodes (either as a
preference or a hard requirement). Taints are the opposite -- they allow a
*node* to *repel* a set of pods.
-->
[こちら](/jp/docs/concepts/configuration/assign-pod-node/#node-affinity-beta-feature)で説明しているノード親和性とは、 *ポッド* を割り当てるにあたり、指定するノードをどのように *集めるか* です（あるいは、優先度や、厳密な要求）。
テイント（taint）は正反対であり、 *ノード* に対してポッドが集まるのを *拒絶（repel）* します。

<!--
Taints and tolerations work together to ensure that pods are not scheduled
onto inappropriate nodes. One or more taints are applied to a node; this
marks that the node should not accept any pods that do not tolerate the taints.
Tolerations are applied to pods, and allow (but do not require) the pods to schedule
onto nodes with matching taints.
-->
ポッドを不適切なノードにスケジュールされるのを阻止するために、劣化（taint）と許容（toleration）を一緒に使います。
ノードに対してテイントを適用すると、対象ノードはポッドを受け付けるべきではなく、劣化（テイント）を許容しません。
許容（toleration）はポッドに対して適用するもので、劣化（テイント）に一致するノードに対してポッドをスケジュールできるようにします（必須ではありません）。


{{% /capture %}}

{{% capture body %}}

<!--
## Concepts
-->
## 概念

<!--
You add a taint to a node using [kubectl taint](/docs/reference/generated/kubectl/kubectl-commands#taint).
For example,
-->
ノードに対してテイントを追加するには [kubectl taint](/jp/docs/reference/generated/kubectl/kubectl-commands#taint) を使います。

```shell
kubectl taint nodes node1 key=value:NoSchedule
```

<!--
places a taint on node `node1`. The taint has key `key`, value `value`, and taint effect `NoSchedule`.
This means that no pod will be able to schedule onto `node1` unless it has a matching toleration.
-->
ノード `node1` にテイントを付けます。
テイントは `key` キーと `value` 値とテイント効果 `NoSchedule` を持ちます。
つまり、 `node`` には一致する許容（toleration）が無い限り一切のポッドをスケジュールできません。

<!--
To remove the taint added by the command above, you can run:
```shell
kubectl taint nodes node1 key:NoSchedule-
```
-->
先ほど追加したテイントの削除は、次のように実行します。
```shell
kubectl taint nodes node1 key:NoSchedule-
```

<!--
You specify a toleration for a pod in the PodSpec. Both of the following tolerations "match" the
taint created by the `kubectl taint` line above, and thus a pod with either toleration would be able
to schedule onto `node1`:
-->
PodSpec の中でポッドに対する許容（toleration）を指定できます。
以下の許容は、いずれも先ほど `kubectl taint` で作成したテイントに「一致する」ものです。


```yaml
tolerations:
- key: "key"
  operator: "Equal"
  value: "value"
  effect: "NoSchedule"
```

```yaml
tolerations:
- key: "key"
  operator: "Exists"
  effect: "NoSchedule"
```

<!--
A toleration "matches" a taint if the keys are the same and the effects are the same, and:
-->
許容（toleration）に「一致する」テイントとは、キーが同じかつ効果が同じであり、さらに以下の条件です。

<!--
* the `operator` is `Exists` (in which case no `value` should be specified), or
* the `operator` is `Equal` and the `value`s are equal
-->
* `operator` が  `Exists` （この場合、 `value` は指定すべきではない ） 、または、
* `operator` が `Equal` かつ `value` が同じ

<!--
`Operator` defaults to `Equal` if not specified.
-->
`Operator` を指定しなければ、デフォルトで `Equal` になります。

<!--
**NOTE:** There are two special cases:
-->
**メモ：** ２つの例外があります。

<!--
* An empty `key` with operator `Exists` matches all keys, values and effects which means this
will tolerate everything.
-->
* 以下は空っぽの `key` とオペレータ `Exists` に一致する全てのキーと値と効果を現します。

```yaml
tolerations:
- operator: "Exists"
```

<!--
* An empty `effect` matches all effects with key `key`.
-->
* 空っぽの `effect` は、全ての効果のキーが `key` と一致します。

```yaml
tolerations:
- key: "key"
  operator: "Exists"
```

<!--
The above example used `effect` of `NoSchedule`. Alternatively, you can use `effect` of `PreferNoSchedule`.
This is a "preference" or "soft" version of `NoSchedule` -- the system will *try* to avoid placing a
pod that does not tolerate the taint on the node, but it is not required. The third kind of `effect` is
`NoExecute`, described later.
-->
先ほどの例では `NoSchedule` の `effect` を使いました。
別の方法として、`PreferNoSchedule` の `effect` も使えます。
これは `NoSchedule` の「優先」または「ソフト」です。
つまり、システムはポッドを指定した場所を避けるのを *試み*  ノード上のテイントを許容しまｓねんが、必須ではありません。
`effect` の３つめは `NoExecute` であり、後述します。

<!--
You can put multiple taints on the same node and multiple tolerations on the same pod.
The way Kubernetes processes multiple taints and tolerations is like a filter: start
with all of a node's taints, then ignore the ones for which the pod has a matching toleration; the
remaining un-ignored taints have the indicated effects on the pod. In particular,
-->
同じノードに対して複数のテイントを指定できますし、同じポッドに対する複数の許容を指定できます。
Kubernetes は複数のテイントと許容をフィルタのように処理します。
つまり、全てのノードのテイントで始めた後、ポッドの許容に一致するノードがあれば除外します。
これにより、無視されなかったテイントがポッド上で示した効果を持ち続けます。
具体的には次の通りです。

<!--
* if there is at least one un-ignored taint with effect `NoSchedule` then Kubernetes will not schedule
the pod onto that node
* if there is no un-ignored taint with effect `NoSchedule` but there is at least one un-ignored taint with
effect `PreferNoSchedule` then Kubernetes will *try* to not schedule the pod onto the node
* if there is at least one un-ignored taint with effect `NoExecute` then the pod will be evicted from
the node (if it is already running on the node), and will not be
scheduled onto the node (if it is not yet running on the node).
-->
* もしも効果が `NoSchedule` で無視できないテイントが少なくとも１つあれば、 Kubernetes は対象ノード上にポッドをスケジュールしない
* もしも効果が `NoSchedule` で無視できないテイントがなくても、効果が `PreferNoSchedule` で無視できないテイントが少なくとも１つあれば、Kubernetes は対象ノード上へスケジュールしないよう *試み*  る
* もしも効果が `NoSchedule` で無視できないテイントが少なくとも１つあれば、ポッドはノード～退避し（既にノード上で実行していたとしても）、 ノード上へのスケジュールは行わない（ノード上でまだ実行していなくても）

<!--
For example, imagine you taint a node like this
-->
たとえば、ノードに次のようなテイントがあるとします。

```shell
kubectl taint nodes node1 key1=value1:NoSchedule
kubectl taint nodes node1 key1=value1:NoExecute
kubectl taint nodes node1 key2=value2:NoSchedule
```

<!--
And a pod has two tolerations:
-->
そして、ポッドには２つの許容（toleration）があります。

```yaml
tolerations:
- key: "key1"
  operator: "Equal"
  value: "value1"
  effect: "NoSchedule"
- key: "key1"
  operator: "Equal"
  value: "value1"
  effect: "NoExecute"
```

<!--
In this case, the pod will not be able to schedule onto the node, because there is no
toleration matching the third taint. But it will be able to continue running if it is
already running on the node when the taint is added, because the third taint is the only
one of the three that is not tolerated by the pod.
-->
この場合、ポッドはノードに対してスケジュールできません。
なぜなら、３つのテイントと一致する許容がないからです。
しかし、テイントの追加時に、既にノード上で実行している場合は、処理を継続できます。
なぜなら３つめのテイントは、３つのテイントの中で唯一ポッドによる許容がないからです。

<!--
Normally, if a taint with effect `NoExecute` is added to a node, then any pods that do
not tolerate the taint will be evicted immediately, and any pods that do tolerate the
taint will never be evicted. However, a toleration with `NoExecute` effect can specify
an optional `tolerationSeconds` field that dictates how long the pod will stay bound
to the node after the taint is added. For example,
-->
通常、ノードに対して `NoExecute` 効果のあるテイントを追加すると、テイントを許容しないあらゆるポッドが直ちに退去されます。
そして、テイントを許容しているポッドは、決して退去させられることはありません。
しかしながら、 `NoExecute` 効果を持つ許容の場合は、オプションで `tolerationSeconds` フィールドを指定できます。
これはノードに対してテイントを追加した後も、ポッドをどれくらいの間に保持しているか指定します。
例えば、


```yaml
tolerations:
- key: "key1"
  operator: "Equal"
  value: "value1"
  effect: "NoExecute"
  tolerationSeconds: 3600
```

<!--
means that if this pod is running and a matching taint is added to the node, then
the pod will stay bound to the node for 3600 seconds, and then be evicted. If the
taint is removed before that time, the pod will not be evicted.
-->
こちらが意味するのは、ポッドが実行中で、ノードに対してテイントの一致を追加した後、ポッドは3600秒まで待った後に退去します。
もしテイントが指定時間よりも早く削除されれば、ポッドは退去させられません。


<!--
## Example Use Cases
-->
## 利用例 {#example-use-cases}

Taints and tolerations are a flexible way to steer pods *away* from nodes or evict
pods that shouldn't be running. A few of the use cases are

* **Dedicated Nodes**: If you want to dedicate a set of nodes for exclusive use by
a particular set of users, you can add a taint to those nodes (say,
`kubectl taint nodes nodename dedicated=groupName:NoSchedule`) and then add a corresponding
toleration to their pods (this would be done most easily by writing a custom
[admission controller](/docs/admin/admission-controllers/)).
The pods with the tolerations will then be allowed to use the tainted (dedicated) nodes as
well as any other nodes in the cluster. If you want to dedicate the nodes to them *and*
ensure they *only* use the dedicated nodes, then you should additionally add a label similar
to the taint to the same set of nodes (e.g. `dedicated=groupName`), and the admission
controller should additionally add a node affinity to require that the pods can only schedule
onto nodes labeled with `dedicated=groupName`.

* **Nodes with Special Hardware**: In a cluster where a small subset of nodes have specialized
hardware (for example GPUs), it is desirable to keep pods that don't need the specialized
hardware off of those nodes, thus leaving room for later-arriving pods that do need the
specialized hardware. This can be done by tainting the nodes that have the specialized
hardware (e.g. `kubectl taint nodes nodename special=true:NoSchedule` or
`kubectl taint nodes nodename special=true:PreferNoSchedule`) and adding a corresponding
toleration to pods that use the special hardware. As in the dedicated nodes use case,
it is probably easiest to apply the tolerations using a custom
[admission controller](/docs/admin/admission-controllers/)).
For example, it is recommended to use [Extended
Resources](/docs/concepts/configuration/manage-compute-resources-container/#extended-resources)
to represent the special hardware, taint your special hardware nodes with the
extended resource name and run the
[ExtendedResourceToleration](/docs/admin/admission-controllers/#extendedresourcetoleration)
admission controller. Now, because the nodes are tainted, no pods without the
toleration will schedule on them. But when you submit a pod that requests the
extended resource, the `ExtendedResourceToleration` admission controller will
automatically add the correct toleration to the pod and that pod will schedule
on the special hardware nodes. This will make sure that these special hardware
nodes are dedicated for pods requesting such hardware and you don't have to
manually add tolerations to your pods.

* **Taint based Evictions (alpha feature)**: A per-pod-configurable eviction behavior
when there are node problems, which is described in the next section.

## Taint based Evictions

Earlier we mentioned the `NoExecute` taint effect, which affects pods that are already
running on the node as follows

 * pods that do not tolerate the taint are evicted immediately
 * pods that tolerate the taint without specifying `tolerationSeconds` in
   their toleration specification remain bound forever
 * pods that tolerate the taint with a specified `tolerationSeconds` remain
   bound for the specified amount of time

In addition, Kubernetes 1.6 introduced alpha support for representing node
problems. In other words, the node controller automatically taints a node when
certain condition is true. The following taints are built in:

 * `node.kubernetes.io/not-ready`: Node is not ready. This corresponds to
   the NodeCondition `Ready` being "`False`".
 * `node.kubernetes.io/unreachable`: Node is unreachable from the node
   controller. This corresponds to the NodeCondition `Ready` being "`Unknown`".
 * `node.kubernetes.io/out-of-disk`: Node becomes out of disk.
 * `node.kubernetes.io/memory-pressure`: Node has memory pressure.
 * `node.kubernetes.io/disk-pressure`: Node has disk pressure.
 * `node.kubernetes.io/network-unavailable`: Node's network is unavailable.
 * `node.kubernetes.io/unschedulable`: Node is unschedulable.
 * `node.cloudprovider.kubernetes.io/uninitialized`: When the kubelet is started
    with "external" cloud provider, this taint is set on a node to mark it
    as unusable. After a controller from the cloud-controller-manager initializes
    this node, the kubelet removes this taint.

When the `TaintBasedEvictions` alpha feature is enabled (you can do this by
including `TaintBasedEvictions=true` in `--feature-gates` for Kubernetes controller manager,
such as `--feature-gates=FooBar=true,TaintBasedEvictions=true`), the taints are automatically
added by the NodeController (or kubelet) and the normal logic for evicting pods from nodes
based on the Ready NodeCondition is disabled.
(Note: To maintain the existing [rate limiting](/docs/concepts/architecture/nodes/)
behavior of pod evictions due to node problems, the system actually adds the taints
in a rate-limited way. This prevents massive pod evictions in scenarios such
as the master becoming partitioned from the nodes.)
This alpha feature, in combination with `tolerationSeconds`, allows a pod
to specify how long it should stay bound to a node that has one or both of these problems.

For example, an application with a lot of local state might want to stay
bound to node for a long time in the event of network partition, in the hope
that the partition will recover and thus the pod eviction can be avoided.
The toleration the pod would use in that case would look like

```yaml
tolerations:
- key: "node.alpha.kubernetes.io/unreachable"
  operator: "Exists"
  effect: "NoExecute"
  tolerationSeconds: 6000
```

Note that Kubernetes automatically adds a toleration for
`node.kubernetes.io/not-ready` with `tolerationSeconds=300`
unless the pod configuration provided
by the user already has a toleration for `node.kubernetes.io/not-ready`.
Likewise it adds a toleration for
`node.alpha.kubernetes.io/unreachable` with `tolerationSeconds=300`
unless the pod configuration provided
by the user already has a toleration for `node.alpha.kubernetes.io/unreachable`.

These automatically-added tolerations ensure that
the default pod behavior of remaining bound for 5 minutes after one of these
problems is detected is maintained.
The two default tolerations are added by the [DefaultTolerationSeconds
admission controller](https://git.k8s.io/kubernetes/plugin/pkg/admission/defaulttolerationseconds).

[DaemonSet](/docs/concepts/workloads/controllers/daemonset/) pods are created with
`NoExecute` tolerations for the following taints with no `tolerationSeconds`:

  * `node.alpha.kubernetes.io/unreachable`
  * `node.kubernetes.io/not-ready`

This ensures that DaemonSet pods are never evicted due to these problems,
which matches the behavior when this feature is disabled.

## Taint Nodes by Condition

Version 1.8 introduces an alpha feature that causes the node controller to create taints corresponding to
Node conditions. When this feature is enabled (you can do this by including `TaintNodesByCondition=true` in the `--feature-gates` command line flag to the scheduler, such as
`--feature-gates=FooBar=true,TaintNodesByCondition=true`), the scheduler does not check Node conditions; instead the scheduler checks taints. This assures that Node conditions don't affect what's scheduled onto the Node. The user can choose to ignore some of the Node's problems (represented as Node conditions) by adding appropriate Pod tolerations.

Starting in Kubernetes 1.8, the DaemonSet controller automatically adds the
following `NoSchedule` tolerations to all daemons, to prevent DaemonSets from
breaking.

  * `node.kubernetes.io/memory-pressure`
  * `node.kubernetes.io/disk-pressure`
  * `node.kubernetes.io/out-of-disk` (*only for critical pods*)
  * `node.kubernetes.io/unschedulable` (1.10 or later)
  * `node.kubernetes.io/network-unavailable` (*host network only*)

Adding these tolerations ensures backward compatibility. You can also add
arbitrary tolerations to DaemonSets.
