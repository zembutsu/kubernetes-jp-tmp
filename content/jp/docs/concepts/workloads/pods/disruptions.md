---
reviewers:
- erictune
- foxish
- davidopp
title: 破壊（disruptions）
content_template: templates/concept
weight: 60
---

{{% capture overview %}}
<!--
This guide is for application owners who want to build
highly available applications, and thus need to understand
what types of Disruptions can happen to Pods.
-->
このガイドは可用性が高いアプリケーションを構築したいアプリケーション所有者のためのものです。また、そのために、ポッドに対してどのような破壊影響を与えるか理解する必要があります。

<!--
It is also for Cluster Administrators who want to perform automated
cluster actions, like upgrading and autoscaling clusters.
-->
また、クラスタの更新やオートスケーリング（自動規模変更）のように、クラスタに対する行動を自動処理したいクラスタ管理者のためでもあります。

{{% /capture %}}

{{< toc >}}

{{% capture body %}}

<!--
## Voluntary and Involuntary Disruptions
-->
### 自発的および不本意な破壊 {#voluntary-and-involuntary-disruptions}

<!--
Pods do not disappear until someone (a person or a controller) destroys them, or
there is an unavoidable hardware or system software error.
-->
ポッドは誰か（人またはコントローラ）が破棄しない限り消えません。あるいは、避けられないハードウェアやシステム・ソフトウェアのエラーによっても消失します。

<!--
We call these unavoidable cases *involuntary disruptions* to
an application.  Examples are:
-->
私たちはアプリケーションが避けられない状態を「不本意な破壊（involuntary disruption）」と呼びます。たとえば：

<!--
- a hardware failure of the physical machine backing the node
- cluster administrator deletes VM (instance) by mistake
- cloud provider or hypervisor failure makes VM disappear
- a kernel panic
- the node disappears from the cluster due to cluster network partition
- eviction of a pod due to the node being [out-of-resources](/docs/tasks/administer-cluster/out-of-resource/).
-->
- ノードを支える物理マシンのハードウェア故障
- クラスタ/管理者が仮想マシン（インスタンス）を誤って削除
- クラウド事業者やハイパーバイザの障害で発生する仮想マシン消失
- カーネルパニック
- クラスタのネトワーク分断（partition）によって、クラスタからノードが見えなくなる（消失する）
- ノードの[リソース不足（out-of-resources）](/jp/docs/tasks/administer-cluster/out-of-resource/) によるポッド撤去

<!--
Except for the out-of-resources condition, all these conditions
should be familiar to most users; they are not specific
to Kubernetes.
-->
リソース不足の状態を除き、これらすべての状態は多くのユーザにとって見慣れたものでしょう。これらは Kubernetes に特化したものではありません。

<!--
We call other cases *voluntary disruptions*.  These include both
actions initiated by the application owner and those initiated by a Cluster
Administrator.  Typical application owner actions include:
-->
私たちは他のケースを *自発的な破壊（voluntary disruption）* と呼びます。ここにはアプリケーション所有者による初期化とクラスタ管理者による初期化の両方を含みます。典型的なアプリケーション所有者による対応に含まれるのは：

<!--
- deleting the deployment or other controller that manages the pod
- updating a deployment's pod template causing a restart
- directly deleting a pod (e.g. by accident)
-->
- ポッドが管理するデプロイメントや他のコントローラを削除
- デプロイメントのポッド・テンプレート更新に伴う再起動
- ポッドを直接削除する（例：不意の事故によって）

<!--
Cluster Administrator actions include:
-->
クラスタ管理者の対応に含まれるのは：

<!--
- [Draining a node](/docs/tasks/administer-cluster/safely-drain-node/) for repair or upgrade.
- Draining a node from a cluster to scale the cluster down (learn about
[Cluster Autoscaling](/docs/tasks/administer-cluster/cluster-management/#cluster-autoscaler)
).
- Removing a pod from a node to permit something else to fit on that node.
-->
- 修復もしくは更新のために[ノード排出（ドレイニング：draining）](/jp/docs/tasks/administer-cluster/safely-drain-node/)
- クラスタ縮小のために、クラスタの規模を減らすノードを排出（[クラスタ・自動規模変更](/jp/docs/tasks/administer-cluster/cluster-management/#cluster-autoscaler)について学ぶ）
- ノードに何かを取り付けられるようにするため、ノードからポッドを取り外す

<!--
These actions might be taken directly by the cluster administrator, or by automation
run by the cluster administrator, or by your cluster hosting provider.
-->
これらの対応は、クラスタ管理者によって直接行われるかもしれませんし、あるいはクラスタ管理者によって自動的に実行さえるかもしれませんし、あるいは、クラスタをホスティングする事業者によって行われる場合があります。

<!--
Ask your cluster administrator or consult your cloud provider or distribution documentation
to determine if any sources of voluntary disruptions are enabled for your cluster.
If none are enabled, you can skip creating Pod Disruption Budgets.
-->
クラスタの管理者に尋ねるか、クラウド事業者と相談するか、またはディストリビューションのドキュメントを読むかなど、クラスタ上で起こりうる自発的破壊について検討します。何もできないのであれば、ポッドの破壊予測を省略できます。

<!--
## Dealing with Disruptions
-->
## 破壊への対応 {#dealing-with-disruption}

<!--
Here are some ways to mitigate involuntary disruptions:
-->
不本意な破壊に至る道がいくつかあります：

<!--
- Ensure your pod [requests the resources](/docs/tasks/configure-pod-container/assign-cpu-ram-container) it needs.
- Replicate your application if you need higher availability.  (Learn about running replicated
[stateless](/docs/tasks/run-application/run-stateless-application-deployment/)
and [stateful](/docs/tasks/run-application/run-replicated-stateful-application/) applications.)
- For even higher availability when running replicated applications,
spread applications across racks (using
[anti-affinity](/docs/user-guide/node-selection/#inter-pod-affinity-and-anti-affinity-beta-feature))
or across zones (if using a
[multi-zone cluster](/docs/setup/multiple-zones).)
-->
- ポッドが確実な [リソースを要求](/jp/docs/tasks/configure-pod-container/assign-cpu-ram-container) 場合
- 高可用性のためにアプリケーションを複製（レプリケート）する場合（
[ステートレス（状態を持たない：stateless）](/jp/docs/tasks/run-application/run-stateless-application-deployment/)
および [ステートフル（状態を持つ：stateful）](/jp/docs/tasks/run-application/run-replicated-stateful-application/) なアプリケーションの複製について学ぶ)
- 複製したアプリケーションを実行して可用性を高めていたとしても、アプリケーションがラック（ [anti-affinity（アンチ・アフィニティ／非親和性）](/jp/docs/user-guide/node-selection/#inter-pod-affinity-and-anti-affinity-beta-feature))
 を津各）やゾーンを横断している場合

<!--
The frequency of voluntary disruptions varies.  On a basic Kubernetes cluster, there are
no voluntary disruptions at all.  However, your cluster administrator or hosting provider
may run some additional services which cause voluntary disruptions. For example,
rolling out node software updates can cause voluntary disruptions. Also, some implementations
of cluster (node) autoscaling may cause voluntary disruptions to defragment and compact nodes.
Your cluster administrator or hosting provider should have documented what level of voluntary
disruptions, if any, to expect.
-->
様々な自主的破壊は頻繁に起こります。基本的な Kubernetes クラスタでは自主的破壊はほとんど発生しません。しかしながら、クラスタ管理者やホスティング事業者であれば、いくつかのサービスを追加するために、自主的破壊を引き起こします。たとえば、ノードに対してソフトウェア更新を展開すると、自主的破壊を引き起こします。また、いくつかのクラスタ（ノード）オートスケーリング（自動規模変更）の実装によっては、ノードの最適化（デフラグ：defragment）と小型化のために自主的破壊を引き起こします。クラスタ管理者またはホスティング事業者は、あらゆる自主的破壊の段階（レベル）を例外なく文章化すべきでしょう。

<!--
Kubernetes offers features to help run highly available applications at the same
time as frequent voluntary disruptions.  We call this set of features
*Disruption Budgets*.
-->
Kubernetes は定期的な自主的破壊が同時に発生したとしても、高可用性アプリケーションの実行に役立つ機能を提供します。私たちはこの機能群を *破壊予測（Disruption Budgets）* と呼びます。

<!--
## How Disruption Budgets Work
-->
## 破壊予測の挙動 {#how-disruption-budgets-work}

<!--
An Application Owner can create a `PodDisruptionBudget` object (PDB) for each application.
A PDB limits the number pods of a replicated application that are down simultaneously from
voluntary disruptions.  For example, a quorum-based application would
like to ensure that the number of replicas running is never brought below the
number needed for a quorum. A web front end might want to
ensure that the number of replicas serving load never falls below a certain
percentage of the total.
-->
アプリケーション所有者は `PodDisruptionBudget` （ポッド破壊予測）オブジェクト（PDB）をアプリケーションごとに作成できます。PDB の上限とは、自主的破壊によって同時にダウン可能な複製アプリケーションのポッド数です。たとえば、クォーラム（quorum）をベースとするアプリケーションであれば、確実な複製（レプリカ）を実行できる数はクォーラム数を必ず下回ります。ウェブ・フロントエンドであれば複製（レプリカ）が負荷分散を途切れず提供するために、全体から一定の割合を確保する必要があるでしょう。

<!--
Cluster managers and hosting providers should use tools which
respect Pod Disruption Budgets by calling the [Eviction API](/docs/tasks/administer-cluster/safely-drain-node/#the-eviction-api)
instead of directly deleting pods.  Examples are the `kubectl drain` command
and the Kubernetes-on-GCE cluster upgrade script (`cluster/gce/upgrade.sh`).
-->
クラスタ管理者とホスティング事業者は、ポッド破壊予測を尊重するツールを使うべきです。そのためには、ポッドを直接削除せず、[Eviction API（退避 API ）](/jp/docs/tasks/administer-cluster/safely-drain-node/#the-eviction-api) を使います。`kubectl drain` コマンドの例であれば、 GCE 上の Kubernetes クラスタで使う更新用のスクリプト（ `cluster/gce/upgrade.sh` ）です。

<!--
When a cluster administrator wants to drain a node
they use the `kubectl drain` command.  That tool tries to evict all
the pods on the machine.  The eviction request may be temporarily rejected,
and the tool periodically retries all failed requests until all pods
are terminated, or until a configurable timeout is reached.
-->
クラスタ管理者がノードを抜き出したい（drain）場合は、 `kubectl drain` コマンドを使います。このツールはマシン上の全てのポッドの退避（evict）を試みます。退去要求は一時的な除去であり、ツールは要求に失敗しても、ポッドが終了するまで定期的に再試行します。これは設定変更可能なタイムアウト時間に到達するまで続きます。

<!--
A PDB specifies the number of replicas that an application can tolerate having, relative to how
many it is intended to have.  For example, a Deployment which has a `.spec.replicas: 5` is
supposed to have 5 pods at any given time.  If its PDB allows for there to be 4 at a time,
then the Eviction API will allow voluntary disruption of one, but not two pods, at a time.
-->
ポッド破壊予測（PDB）で指定するのは複製数（レプリカ数）です。これはアプリケーションが可用性を維持するためには、いくつの数を確保するかです。たとえば、デプロイメントが `.spec.replicas: 5` であれば、常時 5 つのポッドを持てるようにします。ポッド破壊予測（PDB）が 4 であれば、退避（Eviction）API は同時に１つの（ポッドの）自主的破壊を許可しますが、２つは許可しません。

<!--
The group of pods that comprise the application is specified using a label selector, the same
as the one used by the application's controller (deployment, stateful-set, etc).
-->
ポッドのグループを構成するのは、アプリケーションが使うために指定されているラベル・セレクタを含みますが、これはアプリケーションのコントローラ（デプロイメント、ステートフルセット、など）によって使われるものとも同じです。

<!--
The "intended" number of pods is computed from the `.spec.replicas` of the pods controller.
The controller is discovered from the pods using the `.metadata.ownerReferences` of the object.
-->
ポッドの「意図した」（intended）数は、ポッド・コントローラの `.spec.replicas` から算出されます。コントローラはポッドを見つけるのに `.metadata.ownerReferences` にあるオブジェクトを使います。

<!--
PDBs cannot prevent [involuntary disruptions](#voluntary-and-involuntary-disruptions) from
occurring, but they do count against the budget.
-->
ポッド破壊予測（PDB）は、[不本意な破壊](#voluntary-and-involuntary-disruptions) を阻止できませんが、予測に対するカウントはできます。

<!--
Pods which are deleted or unavailable due to a rolling upgrade to an application do count
against the disruption budget, but controllers (like deployment and stateful-set)
are not limited by PDBs when doing rolling upgrades -- the handling of failures
during application updates is configured in the controller spec.
(Learn about [updating a deployment](/docs/concepts/workloads/controllers/deployment/#updating-a-deployment).)
-->
ローリング・アップデートの実施は、プリケーションが破壊予測数に達するまで数えながら、ポッドが削除または利用不可能になります。しかし、ローリング・アップデート中のコントローラ（デプロイメントやステートフル・セット）はポッド破壊予測（PDB）による制限を受けません。つまり、アプリケーション更新中の障害を扱うのは、コントローラ spec に記述がある設定です。
（詳細は[デプロイメントの更新](/jp/docs/concepts/workloads/controllers/deployment/#updating-a-deployment)で学べます。）

<!--
When a pod is evicted using the eviction API, it is gracefully terminated (see
`terminationGracePeriodSeconds` in [PodSpec](/docs/reference/generated/kubernetes-api/{{< param "version" >}}/#podspec-v1-core).)
-->
ポッドが退避に退避（eviction）API を使う場合、ポッドは丁寧に（gracefully）停止します（
詳細は [PodSpec](/docs/reference/generated/kubernetes-api/{{< param "version" >}}/#podspec-v1-core) にある `terminationGracePeriodSeconds` をご覧ください。
）。

<!--
## PDB Example
-->
## ポッド破壊予測の例 {#pdb-example}

<!--
Consider a cluster with 3 nodes, `node-1` through `node-3`.
The cluster is running several applications.  One of them has 3 replicas initially called
`pod-a`, `pod-b`, and `pod-c`.  Another, unrelated pod without a PDB, called `pod-x`, is also shown.
Initially, the pods are laid out as follows:
-->
クラスタには３つのノード、 `node-1` から `node-3` まであるものと想定します。クラスタは複数のアプリケーションを実行しています。初期状態のアプリケーションは `pod-a` 、 `pod-b`、 `pod-c` と呼ぶ３つのレプリカがあります。他にも、ポッド破壊予測（PDB）とは関係の無い `pod-x` もあるものとします。初期状態では、ポッドは以下のように配置されています。

<!--
|       node-1         |       node-2        |       node-3       |
|:--------------------:|:-------------------:|:------------------:|
| pod-a  *available*   | pod-b *available*   | pod-c *available*  |
| pod-x  *available*   |                     |                    |
-->
|       node-1         |       node-2        |       node-3       |
|:--------------------:|:-------------------:|:------------------:|
| pod-a  *利用可*   | pod-b *利用可*   | pod-c *利用可*  |
| pod-x  *利用可*   |                     |                    |

<!--
All 3 pods are part of a deployment, and they collectively have a PDB which requires
there be at least 2 of the 3 pods to be available at all times.
-->
デプロイメントの全てを構成するのは３つのポッドです。また、全体にはポッド破壊予測（PDB）も含むため、３つのポッドうち最低２つのポッドが常時利用可能な状態になっている必要があります。

<!--
For example, assume the cluster administrator wants to reboot into a new kernel version to fix a bug in the kernel.
The cluster administrator first tries to drain `node-1` using the `kubectl drain` command.
That tool tries to evict `pod-a` and `pod-x`.  This succeeds immediately.
Both pods go into the `terminating` state at the same time.
This puts the cluster in this state:
-->
たとえば、クラスタ管理者が再起動を行いたいとします。これは、新しいカーネルを導入して、カーネルのバグ修正に対応するためです。クラスタ管理者はまず、 `node-1` を排出（drain）するために `kubectl drain` コマンドを使います。ツールは `pod-a` と `pod-x` の退避処理を開始します。この処理は直ちに成功します。同時に、どちらのポッドも `terminating` （終了中）の状態となります。この時点でクラスタは以下の状態です。

<!--
|   node-1 *draining*  |       node-2        |       node-3       |
|:--------------------:|:-------------------:|:------------------:|
| pod-a  *terminating* | pod-b *available*   | pod-c *available*  |
| pod-x  *terminating* |                     |                    |
-->
|   node-1 *draining  (排出中)*  |       node-2        |       node-3       |
|:--------------------:|:-------------------:|:------------------:|
| pod-a  *terminating (終了中)* | pod-b *available (利用可)*   | pod-c *available (利用可)*  |
| pod-x  *terminating (終了中)* |                     |                    |

<!--
The deployment notices that one of the pods is terminating, so it creates a replacement
called `pod-d`.  Since `node-1` is cordoned, it lands on another node.  Something has
also created `pod-y` as a replacement for `pod-x`.
-->
デプロイメントにおける注意点としては、ポッドが停止中（terminating）のため、 `pod-d` と呼ぶ代替ポッド作成します。`node-1` を遮断すると、別のノードに切り替わります。また、 `pod-x` を代替する `pod-y` も作成します。

<!--
(Note: for a StatefulSet, `pod-a`, which would be called something like `pod-1`, would need
to terminate completely before its replacement, which is also called `pod-1` but has a
different UID, could be created.  Otherwise, the example applies to a StatefulSet as well.)
-->
（メモ：StatefulSet の場合、`pod-a` は `pod-1` のように呼ばれるかもしれませんが、これは置換前に完全に終了する必要があります。また、 `pod-1`  と名前があるものの、異なる UID を持つポッドが作成されます。また、この例は StatefulSet でも同様に適用します。）

<!--
Now the cluster is in this state:
-->
これでクラスタは次の状態になります：

<!--
|   node-1 *draining*  |       node-2        |       node-3       |
|:--------------------:|:-------------------:|:------------------:|
| pod-a  *terminating* | pod-b *available*   | pod-c *available*  |
| pod-x  *terminating* | pod-d *starting*    | pod-y              |
-->
|   node-1 *draining (排出中)*  |       node-2        |       node-3       |
|:--------------------:|:-------------------:|:------------------:|
| pod-a  *terminating (終了中) * | pod-b *available (利用可)*   | pod-c *available (利用可)*  |
| pod-x  *terminating (終了中)* | pod-d *starting (開始中)*    | pod-y              |

<!--
At some point, the pods terminate, and the cluster looks like this:
-->
もう少しするとポッドは終了し、クラスタは次の状態になります：

<!--
|    node-1 *drained*  |       node-2        |       node-3       |
|:--------------------:|:-------------------:|:------------------:|
|                      | pod-b *available*   | pod-c *available*  |
|                      | pod-d *starting*    | pod-y              |
-->
|    node-1 *drained (排出済)*  |       node-2        |       node-3       |
|:--------------------:|:-------------------:|:------------------:|
|                      | pod-b *available (利用可)*   | pod-c *available (利用可)*  |
|                      | pod-d *starting (開始中)*    | pod-y              |

<!--
At this point, if an impatient cluster administrator tries to drain `node-2` or
`node-3`, the drain command will block, because there are only 2 available
pods for the deployment, and its PDB requires at least 2.  After some time passes, `pod-d` becomes available.
-->
この時点で、せっかちな（我慢できない）クラスタ管理者が `node-2` か `node-3`  の排出（ドレイン）を試みる可能性がありますが、 drain コマンドはブロック（遮断）されます。なぜなら、このデプロイメントにおいては２つのポッドが利用可能な状態であり、ポッド破壊予測（PDB）によって少なくとも２つのポッドが必要だからです。もうしばらく時間が経つと、 `pod-d` が利用可能になります。

<!--
The cluster state now looks like this:
-->
クラスタは次の状態になります：

<!--
|    node-1 *drained*  |       node-2        |       node-3       |
|:--------------------:|:-------------------:|:------------------:|
|                      | pod-b *available*   | pod-c *available*  |
|                      | pod-d *available*   | pod-y              |
-->
|    node-1 *drained (排出済)*  |       node-2        |       node-3       |
|:--------------------:|:-------------------:|:------------------:|
|                      | pod-b *available (利用可)*   | pod-c *available (利用可)*  |
|                      | pod-d *available (利用可)*   | pod-y              |

<!--
Now, the cluster administrator tries to drain `node-2`.
The drain command will try to evict the two pods in some order, say
`pod-b` first and then `pod-d`.  It will succeed at evicting `pod-b`.
But, when it tries to evict `pod-d`, it will be refused because that would leave only
one pod available for the deployment.
-->
これで、クラスタ管理者は `node-2` の排出（drain）を試みます。drain コマンドは２つのポッドを同じ順番で退避をはじめます。まずは `pod-b` を行い、それから `pod-d` です。（ここで、仮に） `pod-b` の退去が成功したとすると、 `pod-d` の退去を試みても拒否されます。なぜならデプロイメント上で離れられるポッドは１つ（に設定している）だからです。

<!--
The deployment creates a replacement for `pod-b` called `pod-e`.
Because there are not enough resources in the cluster to schedule
`pod-e` the drain will again block.  The cluster may end up in this
state:
-->
デプロイメントは `pod-b` を置換する `pod-e` を作成します。これはクラスタ上に `pod-e` をスケジュールするための十分なリソースがないためで、排出は再びブロックされます。クラスタは最終的に次の状態となります。

<!--
|    node-1 *drained*  |       node-2        |       node-3       | *no node*          |
|:--------------------:|:-------------------:|:------------------:|:------------------:|
|                      | pod-b *available*   | pod-c *available*  | pod-e *pending*    |
|                      | pod-d *available*   | pod-y              |                    |
-->
|    node-1 *drained (排出済み)*  |       node-2        |       node-3       | *ノードが無い*          |
|:--------------------:|:-------------------:|:------------------:|:------------------:|
|                      | pod-b *available (利用可)*   | pod-c *available (利用可)*  | pod-e *pending (保留中)*    |
|                      | pod-d *available (利用可)*   | pod-y              |                    |

<!--
At this point, the cluster administrator needs to
add a node back to the cluster to proceed with the upgrade.
-->
この時点で、クラスタが更新を継続を復帰できるようにするには、クラスタ管理者はノードを追加する必要があります。

<!--
You can see how Kubernetes varies the rate at which disruptions
can happen, according to:
-->
このようにしても、破壊が発生しても、以下の条件を考慮しながらであれば  Kubernetes が対応できるのが分かりました：

<!--
- how many replicas an application needs
- how long it takes to gracefully shutdown an instance
- how long it takes a new instance to start up
- the type of controller
- the cluster's resource capacity
-->
- アプリケーションが必要な複製数はいくつか
- インスタンスの丁寧な停止にかかる時間はどれくらいか
- 新しいインスタンスの起動にかかる時間はどれくらいか
- コントローラの種類
- クラスタのリソース許容量

<!--
## Separating Cluster Owner and Application Owner Roles
-->
## クラスタの所有者とアプリケーション所有者の役割を分離 {#separating-cluster-owner-and-application-owner-roles}

<!--
Often, it is useful to think of the Cluster Manager
and Application Owner as separate roles with limited knowledge
of each other.   This separation of responsibilities
may make sense in these scenarios:
-->
クラスタの管理者とアプリケーション所有者を考えるにあたり、お互いの限られた知識ごとに役割分割するのは便利です。責任の分離は、以下のシナリオ時に役立つでしょう。

<!--
- when there are many application teams sharing a Kubernetes cluster, and
  there is natural specialization of roles
- when third-party tools or services are used to automate cluster management
-->
- 多くのアプリケーション・チームで Kubernetes クラスタを共有し、特別な役割を持つ担当者がいる場合
- サードパーティ製ツールやサービスを自動化クラスタ管理に使う場合

<!--
Pod Disruption Budgets support this separation of roles by providing an
interface between the roles.
-->
ポッド破壊予測は、役割に応じて異なる作用をもたらします。

<!--
If you do not have such a separation of responsibilities in your organization,
you may not need to use Pod Disruption Budgets.
-->
もしもあなたが組織で責任上の役割分割がなければ、ポッド破壊予測を使う必要性はないかもしれません。

<!--
## How to perform Disruptive Actions on your Cluster
-->
## クラスタ破壊への対処方法 {#how-to-perform-disruptive-actions-on-your-cluster}

<!--
If you are a Cluster Administrator, and you need to perform a disruptive action on all
the nodes in your cluster, such as a node or system software upgrade, here are some options:
-->
クラスタ管理者であれば、クラスタ上の全てのノードにおける破壊的な出来事への対処が必要です。たとえば、ノードやシステム・ソフトウェアの更新では、以下の選択肢があります：

<!--
- Accept downtime during the upgrade.
- Fail over to another complete replica cluster.
   -  No downtime, but may be costly both for the duplicated nodes,
     and for human effort to orchestrate the switchover.
- Write disruption tolerant applications and use PDBs.
   - No downtime.
   - Minimal resource duplication.
   - Allows more automation of cluster administration.
   - Writing disruption-tolerant applications is tricky, but the work to tolerate voluntary
     disruptions largely overlaps with work to support autoscaling and tolerating
     involuntary disruptions.
-->
- アップグレード期間中の停止時間（ダウンタイム）を許容する
- 他の計算複製（レプリカ）クラスタへフェイルオーバーする
   -  停止時間はないが、ノードの複製コストと、切替を調整（オーケストレート）するために人間の努力コストが必要
- 分断に耐えうる（disruption tolerant）アプリケーションを書き、ポッド破壊予測（PDB）を使う
   -  停止時間なし
   -  リソース重複は最小
   -  クラスタ管理をより自動化できる
   - 分断に耐えうるアプリケーションを書くのはトリッキーですが、自発的破壊を許容するのと大部分が許容しており、オートスケーリング（自動規模変更）や不意の破壊に対する許容もサポートする


{{% /capture %}}


{{% capture whatsnext %}}

<!--
* Follow steps to protect your application by [configuring a Pod Disruption Budget](/docs/tasks/run-application/configure-pdb/).

* Learn more about [draining nodes](/docs/tasks/administer-cluster/safely-drain-node/)
-->
* アプリケーションを [ポッド破壊予測設定](/jp/docs/tasks/run-application/configure-pdb/) に対応する手順を進める。
* [ノード排出（ドレイン）](/jp/docs/tasks/administer-cluster/safely-drain-node/) について学ぶ。

{{% /capture %}}



