---
reviewers:
- caesarxuchao
- dchen1107
title: ノード
content_template: templates/concept
weight: 10
---

{{% capture overview %}}

<!--
A `node` is a worker machine in Kubernetes, previously known as a `minion`. A node
may be a VM or physical machine, depending on the cluster. Each node has
the services necessary to run [pods](/docs/concepts/workloads/pods/pod/) and is managed by the master
components. The services on a node include Docker, kubelet and kube-proxy. See
[The Kubernetes Node](https://git.k8s.io/community/contributors/design-proposals/architecture/architecture.md#the-kubernetes-node) section in the
architecture design doc for more details.
-->
`node` （ノード）とは、 Kubernetes における作業マシン（worker machine）であり、以前は `minion` （ミニオン）として知られていました。
ノードは仮想マシンまたは物理マシンであり、クラスタに依存します。
各ノードでサービスに欠かせないのが [ポッド](/jp/docs/concepts/workloads/pods/pod/) の実行と、ポッドがマスタ・コンポーネント（構成要素）によって管理されることです。
ノード上のサービスとして含まれるのは Docker 、kubectl 、kube-proxy です。
アーキテクチャ設計ドキュメントの [Kubernetes Node](https://git.k8s.io/community/contributors/design-proposals/architecture/architecture.md#the-kubernetes-node) セクションに詳細があります。


{{% /capture %}}

{{< toc >}}

{{% capture body %}}

<!--
## Node Status
-->
## ノード状態（status）{#node-status}

<!--
A node's status contains the following information:
-->
ノードの状態には以下の情報を含みます：

<!--
* [Addresses](#addresses)
* [Condition](#condition)
* [Capacity](#capacity)
* [Info](#info)
-->
* [アドレス](#addresses)
* [状況](#condition)
* [キャパシティ](#capacity)
* [情報](#info)

<!--
Each section is described in detail below.
-->
以下の各セクションで詳細を説明します。

<!--
### Addresses
-->
### アドレス

<!--
The usage of these fields varies depending on your cloud provider or bare metal configuration.
-->
各フィールドの使い方は、クラウド事業者やベアメタル設定情報に依存して変わります。

<!--
* HostName: The hostname as reported by the node's kernel. Can be overridden via the kubelet `--hostname-override` parameter.
* ExternalIP: Typically the IP address of the node that is externally routable (available from outside the cluster).
* InternalIP: Typically the IP address of the node that is routable only within the cluster.
-->
* ホスト名（HostName）：ノードのカーネルが報告するホスト名（hostname）です。kubelet の `--hostname-override` パラメータいよって上書きできます。
* 外部 IP（ExternalIP）：ノードの典型的な IP アドレスであり、外部に径路付けが可能（ルーティング可能／routable）です（クラスタの外からも利用できます）。
* 内部 IP（InternalIP）：ノードの典型的な IP アドレスであり、クラスタ内でのみ径路付けが可能（ルーティング可能）です。

<!--
### Condition
-->
### 状況（Condition） {#condition}

<!--
The `conditions` field describes the status of all `Running` nodes.
-->
`conditions` （状況）フィールドは、全ての `Running`（実行中）ノードのステータス（状態）を説明します。

<!--
| Node Condition | Description |
|----------------|-------------|
| `OutOfDisk`    | `True` if there is insufficient free space on the node for adding new pods, otherwise `False` |
| `Ready`        | `True` if the node is healthy and ready to accept pods, `False` if the node is not healthy and is not accepting pods, and `Unknown` if the node controller has not heard from the node in the last `node-monitor-grace-period` (default is 40 seconds) |
| `MemoryPressure`    | `True` if pressure exists on the node memory -- that is, if the node memory is low; otherwise `False` |
| `PIDPressure`    | `True` if pressure exists on the processes -- that is, if there are too many processes on the node; otherwise `False` |
| `DiskPressure`    | `True` if pressure exists on the disk size -- that is, if the disk capacity is low; otherwise `False` |
| `NetworkUnavailable`    | `True` if the network for the node is not correctly configured, otherwise `False` |
| `ConfigOK`    | `True` if the kubelet is correctly configured, otherwise `False` |
-->
| ノード状況 | 説明 |
|----------------|-------------|
| `OutOfDisk`    | `True` の場合、新しいポッドをノードに追加するための十分な空きスペースが無い状態。そうでなければ `False` |
| `Ready`        | `True` の場合は、ノードが正常（healthy）かつポッドの受け入れ準備が整っている状態。 `False` はノードが正常ではなく、ポッドを受け入れられない状態。 `Unknows` （不明）はノード・コントローラがノードからの応答が直近の `node-monitor-grace-period` （ノード監視猶予間隔）(デフォルトは 40 秒) 以内に無かった状態。 |
| `MemoryPressure`    | `True`の場合、ノード・メモリ上に圧力（プレッシャー）が存在する。つまり、ノードのメモリが少ない。そうでなければ `False` |
| `PIDPressure`    | `True` の場合、プロセス上に圧力（プレッシャー）が存在する。つまり、ノード上に多くのプロセスが存在している。そうでなければ `False` |
| `DiskPressure`    | `True` の場合、ディスク容量上にプレッシャが存在する。つまり、ディスク空き容量（キャパシティ）が少ない。そうでなければ `False` |
| `NetworkUnavailable`    | `True` の場合、ノードのネットワークが適切に設定されていない。そうでなければ `False` |
| `ConfigOK`    | `True`の場合、kubeletは適切に設定されている。そうでなければ `False` |

<!--
The node condition is represented as a JSON object. For example, the following response describes a healthy node.
-->
ノード設定は JSON オブジェクトとして表示されます。
たおとえば、以下は正常な（healthy）ノードの応答です。

```json
"conditions": [
  {
    "type": "Ready",
    "status": "True"
  }
]
```
r
<!--
If the Status of the Ready condition is "Unknown" or "False" for longe than the `pod-eviction-timeout`, an argument is passed to the [kube-controller-manager](/docs/admin/kube-controller-manager/) and all of the Pods on the node are scheduled for deletion by the Node Controller. The default eviction timeout duration is **five minutes**. In some cases when the node is unreachable, the apiserver is unable to communicate with the kubelet on it. The decision to delete the pods cannot be communicated to the kubelet until it re-establishes communication with the apiserver. In the meantime, the pods which are scheduled for deletion may continue to run on the partitioned node.
-->
待機状態が "Unknown"（不明）もしくは "False" が `pod-eviction-timeout` （ポッド退避タイムアウト）よりも長くなれば、
引数（argument）が [kube-controller-manager](/jp/docs/admin/kube-controller-manager/) に渡され、
ノード・コントローラによって、ノード上にあるポッド全ての削除がスケジュールされます。
デフォルトの退避タイムアウト期間は **５分間** です。
場合によっては、ノードとの疎通がとれなくなり、apiserver は対象ノード上の kubelet とが通信できなくなります。
apiserver との通信が再確立できないようであれば、 kubelet と通信できないポッドの削除を検討します。
そうこうしているうちに、隔たれたノード上で実行しているポッドの削除がスケジュールされてしまいます。

<!--
In versions of Kubernetes prior to 1.5, the node controller would [force delete](/docs/concepts/workloads/pods/pod/#force-deletion-of-pods)
these unreachable pods from the apiserver. However, in 1.5 and higher, the node controller does not force delete pods until it is
confirmed that they have stopped running in the cluster. One can see these pods which may be running on an unreachable node as being in
the "Terminating" or "Unknown" states. In cases where Kubernetes cannot deduce from the underlying infrastructure if a node has
permanently left a cluster, the cluster administrator may need to delete the node object by hand.  Deleting the node object from
Kubernetes causes all the Pod objects running on it to be deleted from the apiserver, freeing up their names.
-->
Kubernetes 1.5 未満のバージョンでは、apiserver から到達しないポッドに対して、ノード・コントローラは [強制削除（force delete）](/jp/docs/concepts/workloads/pods/pod/#force-deletion-of-pods) を行います。
しかしながら、1.5 および以降のバージョンでは、ノード・コントローラは確認のない強制削除は行わず、クラスタ上での実行が止まっているものとします。
これにより、到達できないノード上で実行している可能性のあるポッドは、「Terminating」（終了中）もしくは「Unknown」（不明）状態として表示されます。
念のため、Kubernetes は基盤を支えるインフラ上にあるノードが、クラスタから永久に切り離されたとは想定しません。
クラスタの管理者が、ノード・オブジェクトを手動で削除する必要があります。
Kubernetes からノード・オブジェクトを削除すると、そこで動いている全てのポッド・オブジェクトも apiserver によって削除されます。これは、名前による制限を解かれるからです。

<!--
Version 1.8 introduced an alpha feature that automatically creates
[taints](/docs/concepts/configuration/taint-and-toleration/) that represent conditions.
To enable this behavior, pass an additional feature gate flag `--feature-gates=...,TaintNodesByCondition=true`
to the API server, controller manager, and scheduler.
When `TaintNodesByCondition` is enabled, the scheduler ignores conditions when considering a Node; instead
it looks at the Node's taints and a Pod's tolerations.
-->
バージョン 1.8 では状況を表す [taint（テイント：故障、汚染）](/jp/docs/concepts/configuration/taint-and-toleration/)  を自動的に作成するアルファ機能が導入されました。
この挙動を有効にすると、API サーバ、コントローラ、マネージャ、スケジューラに対して、追加機能ゲート・フラグ `--feature-gates=...,TaintNodesByCondition=true` を渡します。
もしも `TaintNodesByCondition` が有効であれば、スケジューラはノードの状況考慮を無視します。つまり、そのかわりにノードのテイントとポッドの耐性が得られます。

<!--
Now users can choose between the old scheduling model and a new, more flexible scheduling model.
A Pod that does not have any tolerations gets scheduled according to the old model. But a Pod that 
tolerates the taints of a particular Node can be scheduled on that Node.
-->
これでユーザは古いスケジューリング方式か、新しいより柔軟なスケジューリング方式かを選択できるようになりました。
ポッドは古いモデルで言うところのスケジュールに対する耐性を持ちません。
しかし、ポッドの耐性とはノードでテイント（故障）が起こったとしても、別のノードでスケジュールされうるからです。

<!--
Note that because of small delay, usually less than one second, between time when condition is observed and a taint
is created, it's possible that enabling this feature will slightly increase number of Pods that are successfully
scheduled but rejected by the kubelet.
-->
状況が発見されてテイント（taint）が作成されるまでは、常に１秒以下のわずかな遅延が起こるためご注意ください。
これにより、この機能を有効化すると作成されるポッドの数がわずかに増える可能性があります。
これはスケジュールに成功しても、kubelet によって拒否（rejected）されたものがあるからです。

<!--
### Capacity
-->
### キャパシティ（Capacity） {#capacity}

<!--
Describes the resources available on the node: CPU, memory and the maximum
number of pods that can be scheduled onto the node.
-->
ノード上で利用可能なリソースを示します。
たとえば、CPU、メモリ、ノード上にスケジュールできるポッドの最大数です。


<!--
### Info
-->
### 情報 {#info}

<!--
General information about the node, such as kernel version, Kubernetes version
(kubelet and kube-proxy version), Docker version (if used), OS name.
The information is gathered by Kubelet from the node.
-->
ノードに関する一般的な情報を表示します。例えば、カーネルのバージョン、Kubernetes バージョン（kubelet および kube-proxy バージョン）、Docker バージョン（もし使っていれば）、OS 名などです。
情報は kubelet によってノードから集められます。

<!--
## Management
-->
## 管理 {#management}

<!--
Unlike [pods](/docs/concepts/workloads/pods/pod/) and [services](/docs/concepts/services-networking/service/),
a node is not inherently created by Kubernetes: it is created externally by cloud
providers like Google Compute Engine, or exists in your pool of physical or virtual
machines. What this means is that when Kubernetes creates a node, it is really
just creating an object that represents the node. After creation, Kubernetes
will check whether the node is valid or not. For example, if you try to create
a node from the following content:
-->
[ポッド](/jp/docs/concepts/workloads/pods/pod/) や [サービス](/jp/docs/concepts/services-networking/service/) とは異なり、ノードは本質的に Kubernetes によって作成されるものではありません。
つまり、ノードとは Google Compute Engine のようなクラウド事業者によって外部で作成されるか、既存の物理または仮想マシンのプール上にあります。
これが意味するのは、Kubernetes でノードを作成する時に、実際に作成するオブジェクトとはノードに相当する（意味を持つ）ものです。
作成後、Kubernetes はノードが有効かどうかの確認をします。
たとえば、以下の内容のノードを作成しようとします：

```json
{
  "kind": "Node",
  "apiVersion": "v1",
  "metadata": {
    "name": "10.240.79.157",
    "labels": {
      "name": "my-first-k8s-node"
    }
  }
}
```

<!--
Kubernetes will create a node object internally (the representation), and
validate the node by health checking based on the `metadata.name` field (we
assume `metadata.name` can be resolved). If the node is valid, i.e. all necessary
services are running, it is eligible to run a pod; otherwise, it will be
ignored for any cluster activity until it becomes valid. Note that Kubernetes
will keep the object for the invalid node unless it is explicitly deleted by
the client, and it will keep checking to see if it becomes valid.
-->
Kubernetes はノード・オブジェクトを内部に（表面上は）作成します。
そして、ノードが有効かどうかを、`metadata.name` フィールドをベースとした正常性チェック（health check）で行います（ `metadata.name` で名前解決できると想定します）。
ノードが有効であれば、例えば、全ての必要なサービスが実行中であれば、ポッドを実行する資格があります。
もしも有効でなければ、有効になるまでは、あらゆるクラスタ活動が無視されます。
クライアントによってノード削除を明示しない限り、Kubernetes は無効なノードのオブジェクトを維持しますのでご注意ください。
また、有効になったとしてもチェックは継続します。

<!--
Currently, there are three components that interact with the Kubernetes node
interface: node controller, kubelet, and kubectl.
-->
現時点では３つの構成要素（コンポーネント）が Kubernetes ノード・インターフェースと通信します。ノード・コントローラ、kubelet、kubectl です。

<!--
### Node Controller
-->
### ノード・コントローラ（Node Controller）{#node-controller}

<!--
The node controller is a Kubernetes master component which manages various
aspects of nodes.
-->
ノード・コントローラは Kubernetes の主要な構成要素（コンポーネント）であり、ノードの様々な面を管理します。

<!--
The node controller has multiple roles in a node's life. The first is assigning a
CIDR block to the node when it is registered (if CIDR assignment is turned on).
-->
ノード・コントローラはノードの寿命（life）において複数の役割を持ちます。
はじめに割り当てられるのは、ノードが登録された時、ノードに対する CIDR ブロックです（CIDR 割り当てが有効な場合）。

<!--
The second is keeping the node controller's internal list of nodes up to date with
the cloud provider's list of available machines. When running in a cloud
environment, whenever a node is unhealthy, the node controller asks the cloud
provider if the VM for that node is still available. If not, the node
controller deletes the node from its list of nodes.
-->
２つめは、ノード・コントローラが内部に維持するノード一覧を、クラウド・プロバイダで利用可能なマシン一覧に更新することです。
クラウド環境で実行すると、ノードが障害（unhealthy）でなければ、ノード・コントローラはクラウド・プロバイダに対して、ノードが利用可能な VM があるかどうかを訊ねます。
もしそうでなければ、ノード・コントローラはノード一覧から、この対象となるノードを削除します。

<!--
The third is monitoring the nodes' health. The node controller is
responsible for updating the NodeReady condition of NodeStatus to
ConditionUnknown when a node becomes unreachable (i.e. the node controller stops
receiving heartbeats for some reason, e.g. due to the node being down), and then later evicting
all the pods from the node (using graceful termination) if the node continues
to be unreachable. (The default timeouts are 40s to start reporting
ConditionUnknown and 5m after that to start evicting pods.) The node controller
checks the state of each node every `--node-monitor-period` seconds.
-->
３つめは、ノード正常性の監視です。
ノード・コントローラはノードに到達不能になり（例：何らかの理由によってハードビードを受信してノード・コントローラ停止、あるいは、ノードの停止が始まったことにより）、 NodeStatus（ノード状態）が ConditionUnknown （状況不明）であれば、  NodeReady（ノード待機）状態に更新する責任を持ちます。
また、その後もノードに到達不能の状態が継続する場合は、（丁寧な停止を使い）ノード上の全てのポッドを退避します（デフォルトのタイムアウトは ConditionUnknown の報告後 40 秒で、5分経過するとポッドの退避を開始）。
ノード・コントローラは  `--node-monitor-period` 秒ごとに状態を確認します。

<!--
In Kubernetes 1.4, we updated the logic of the node controller to better handle
cases when a large number of nodes have problems with reaching the master
(e.g. because the master has networking problem). Starting with 1.4, the node
controller will look at the state of all nodes in the cluster when making a
decision about pod eviction.
-->
Kubernetes 1.4 では、私たちはノード・コントローラのロジックを更新し、多くのノードがあるときもマスタに到達できない問題がおこらず、上手に扱えるようにしました（例：マスタにネットワーク上の問題がある場合）。
1.4 からは、ポッドの退避を決めるにあたり、ノード・コントローラがクラスタ内の全てのノード状態を観察します。

<!--
In most cases, node controller limits the eviction rate to
`--node-eviction-rate` (default 0.1) per second, meaning it won't evict pods
from more than 1 node per 10 seconds.
-->
ほとんどの場合、ノード・コントローラは1秒あたりの退避レートを `--node-eviction-rate` で指定しています（デフォルトは 0.1）。
つまり、１ノードあたり 10 秒を越えて、ポッドを退避しません。

<!--
The node eviction behavior changes when a node in a given availability zone
becomes unhealthy. The node controller checks what percentage of nodes in the zone
are unhealthy (NodeReady condition is ConditionUnknown or ConditionFalse) at
the same time. If the fraction of unhealthy nodes is at least
`--unhealthy-zone-threshold` (default 0.55) then the eviction rate is reduced:
if the cluster is small (i.e. has less than or equal to
`--large-cluster-size-threshold` nodes - default 50) then evictions are
stopped, otherwise the eviction rate is reduced to
`--secondary-node-eviction-rate` (default 0.01) per second. The reason these
policies are implemented per availability zone is because one availability zone
might become partitioned from the master while the others remain connected. If
your cluster does not span multiple cloud provider availability zones, then
there is only one availability zone (the whole cluster).
-->
ノード退避の挙動が変わるのは、ノードが稼働している availability ゾーンが異常（unhealthy）になった時です。
ノード・コントローラはゾーン内の何パーセントのノードが同時に異常（NodeReady 状態が ConditionUnknown もしくは ConditionFalse）になっているかどうかを調べます。
もし、異常ノードがごく少量のノードで、`--unhealthy-zone-threshold` (デフォルト 0.55)  以下であれば、退避レートを減少します。たとえば、クラスタが小さければ（例：`--large-cluster-size-threshold` で指定した値よりもノードが等しいか少なくなるように。デフォルトは 50 ） 退避は停止します。
そうでなければ、退避レートは１秒ごとに `--secondary-node-eviction-rate` ずつ（デフォルトは 0.01）減少します。
これらの方針（ポリシー）をアベイラビリティ・ゾーンごとに実装されているのは、１つのアベイラビリティ・ゾーンが他と通信中にもかかわらずマスタから切り離されてしまう可能性があるためです。
もしクラスタがクラウド事業者にある複数のアベイラビリティ・ゾーンを横断しないのであれば、アベイラビリティ・ゾーン（クラスタ全体）は１つしかないことになります。

<!--
A key reason for spreading your nodes across availability zones is so that the
workload can be shifted to healthy zones when one entire zone goes down.
Therefore, if all nodes in a zone are unhealthy then node controller evicts at
the normal rate `--node-eviction-rate`.  The corner case is when all zones are
completely unhealthy (i.e. there are no healthy nodes in the cluster). In such
case, the node controller assumes that there's some problem with master
connectivity and stops all evictions until some connectivity is restored.
-->
アベイラビリティ・ゾーンを横断してノードが拡散する主な理由は、あるノード全体で障害が発生（ダウン）した時、正常なゾーンにワークロードが移行するためです。
そのため、ゾーン内の全てのノードが異常（unhealthy）になれば、通常はレート（比率）が `--node-eviction-rate` を越えると、ノード・コントローラは退避します。
このような場合、ノード・コントローラはマスタとの疎通に何らかの問題が発生したと見なし、どこかとの接続性が回復するまで、全ての退避は停止したままにします。

<!--
Starting in Kubernetes 1.6, the NodeController is also responsible for evicting
pods that are running on nodes with `NoExecute` taints, when the pods do not tolerate
the taints. Additionally, as an alpha feature that is disabled by default, the
NodeController is responsible for adding taints corresponding to node problems like
node unreachable or not ready. See [this documentation](/docs/concepts/configuration/taint-and-toleration/)
for details about `NoExecute` taints and the alpha feature.
-->
Kubernetes 1.6 以降、NodeController もポッドの退避に責任を持つようになりました。
ポッドが汚染（taint）に対する耐性（tolerate）がなければ、 `NoExecute` （実行でいない）テイントを持つノード上で実行しているポッドを NodeController が退避します。
なお、これはアルファ機能のため、デフォルトでは無効化されています。
NodeController が責任を持つのは、ノードに到達できない場合や準備ができていないなどノードの問題に対して、適切に汚染（taint）を追加することです。

<!--
Starting in version 1.8, the node controller can be made responsible for creating taints that represent
Node conditions. This is an alpha feature of version 1.8.
-->
バージョン 1.8 からは、ノード・コントローラはノード状態を表すテイント（taint）の作成に責任を持つことになりました。
これはバージョン 1.8 のアルファ機能です。

<!--
### Self-Registration of Nodes
-->
### ノードの自己登録 {#self-registration-of-nodes}

<!--
When the kubelet flag `--register-node` is true (the default), the kubelet will attempt to
register itself with the API server.  This is the preferred pattern, used by most distros.
-->
kubelet のフラグ `--register-node` が true であれば（デフォルトです）、kubelet は自分自身を API サーバへの登録を試みます。
これが望ましい形態（パターン）であり、多くのディストリビューションで使われています。

<!--
For self-registration, the kubelet is started with the following options:
-->
自分で登録する（self-registration）には、以下のオプションで kubelet を起動します。

<!--
  - `--kubeconfig` - Path to credentials to authenticate itself to the apiserver.
  - `--cloud-provider` - How to talk to a cloud provider to read metadata about itself.
  - `--register-node` - Automatically register with the API server.
  - `--register-with-taints` - Register the node with the given list of taints (comma separated `<key>=<value>:<effect>`). No-op if `register-node` is false.
  - `--node-ip` - IP address of the node.
  - `--node-labels` - Labels to add when registering the node in the cluster.
  - `--node-status-update-frequency` - Specifies how often kubelet posts node status to master.
-->
  - `--kubeconfig` - apiserver に自分自身を登録するために使う信用証明（credential）のパス。
  - `--cloud-provider` - クラウド事業者と通信するために使う、自分自身に関するメタデータ。
  - `--register-node` - API サーバの自動的に登録。
  - `--register-with-taints` - ノードを指定したテイント（taint）の一覧に基づき登録（カンマ区切りの `<キー>=<値>:<効果>`）。もし `register-node` が false であれば何も行いません。
  - `--node-ip` - ノードの IP アドレス。
  - `--node-labels` - クラスタにノードを登録するときのラベルを追加。
  - `--node-status-update-frequency` - kubelet がノードのステータスを master に何回ポストするかを指定

<!--
Currently, any kubelet is authorized to create/modify any node resource, but in practice it only creates/modifies
its own. (In the future, we plan to only allow a kubelet to modify its own node resource.)
-->
現在の所、どのようなノード・リソースに対しても kubelet は作成・変更する権限を持っています。
しかし、現実的に破作成・変更は自分自身に留めます（将来的には、kubelet は自分自身のノード・リソースのみ変更できるようにするのを計画しています）。

<!--
#### Manual Node Administration
-->
#### 手動のノード管理 {#manual-node-administration}

<!--
A cluster administrator can create and modify node objects.
-->
クラスタの管理者はノード・オブジェクトの作成と変更を行えます。

<!--
If the administrator wishes to create node objects manually, set the kubelet flag
`--register-node=false`.
-->
管理者がノード・オブジェクトを手動で作りたい場合は、kubelet にフラグ `--register-node=false`  を指定します。

<!--
The administrator can modify node resources (regardless of the setting of `--register-node`).
Modifications include setting labels on the node and marking it unschedulable.
-->
管理者はノード・リソースの変更が可能です（ `--register-node` の設定を無視します）。

<!--
Labels on nodes can be used in conjunction with node selectors on pods to control scheduling,
e.g. to constrain a pod to only be eligible to run on a subset of the nodes.
-->
ノード上のラベルは、ポッド上のノード・セレクタとスケジューリング管理を連結（結び付ける）ために使います。
例えば、適切なノードのサブセット上でのみ実行するポッドを制限する場合です。

<!--
Marking a node as unschedulable will prevent new pods from being scheduled to that
node, but will not affect any existing pods on the node. This is useful as a
preparatory step before a node reboot, etc. For example, to mark a node
unschedulable, run this command:
-->
スケジュール不可能（unschedulable）とマーク（印付け）されたノードには、新しいポッドがスケジュールされるのを阻止します。
ですが、これはノードが対象であり、ノード上にある既存のポッドに対しては何ら影響はありません。
これはノードの再起動など、事前の準備段階に使うのが便利です。
たとえば、ノードをスケジュール不可能と印を付けるには、次のコマンドを実行します。

```shell
kubectl cordon $NODENAME
```

<!--
Note that pods which are created by a DaemonSet controller bypass the Kubernetes scheduler,
and do not respect the unschedulable attribute on a node.  The assumption is that daemons belong on
the machine even if it is being drained of applications in preparation for a reboot.
-->
DaemonSet コントローラが Kubernetes スケジューラをバイパスして作成されたポッドは、ノードのスケジュール不可能な属性を一切考慮しません。
これは、デーモンが対象マシン上に存在していると仮定しているためで、たとえ再起動の準備のためにアプリケーションのドレイン（排出）を行っているとしてもです。

<!--
### Node capacity
-->
### ノードの収容能力（キャパシティ） {#node-capacity}

<!--
The capacity of the node (number of cpus and amount of memory) is part of the node object.
Normally, nodes register themselves and report their capacity when creating the node object. If
you are doing [manual node administration](#manual-node-administration), then you need to set node
capacity when adding a node.
-->
ノードの許容能力（CPU 数、メモリ容量）とはノード・オブジェクトの一部です。
通常、ノード・オブジェクトの作成時、ノードは自分自身を登録し、自身の許容能力（キャパシティ）を報告します。
もしも [ノードの手動管理](#manual-node-administration) を考えている場合は、ノードの追加時にノード許容容量を設定する必要があります。

<!--
The Kubernetes scheduler ensures that there are enough resources for all the pods on a node.  It
checks that the sum of the requests of containers on the node is no greater than the node capacity.  It
includes all containers started by the kubelet, but not containers started directly by Docker nor
processes not in containers.
-->
Kubernetes スケジューラは、ノード上の全てのポッドに対して十分なリソースがあるのを確保します。
スケジューラはノード上にあるコンテナの要求を合計し、ノードの許容容量（キャパシティ）を越えないようにチェックします。
これには kubelet によって作成された全てのコンテナを含みます。
しかし、Docker が直接開始したコンテナや、コンテナ内に入っていないプロセスは含みません。

<!--
If you want to explicitly reserve resources for non-pod processes, you can create a placeholder
pod. Use the following template:
-->
ポッド以外のプロセスに対しても、明示的にリソースを予約したい場合は、 代替ポッド（placeholder pod）を作成できます。 
以下のテンプレートをお使いください。


```yaml
apiVersion: v1
kind: Pod
metadata:
  name: resource-reserver
spec:
  containers:
  - name: sleep-forever
    image: k8s.gcr.io/pause:0.8.0
    resources:
      requests:
        cpu: 100m
        memory: 100Mi
```

<!--
Set the `cpu` and `memory` values to the amount of resources you want to reserve.
Place the file in the manifest directory (`--config=DIR` flag of kubelet).  Do this
on each kubelet where you want to reserve resources.
-->
予約したいリソース量の `cpu` と `memory` を設定します。
マニフェスト・ディレクトリ内（ kubelet の `--config=DIR` フラグ）にこのファイルを置きます。
この作業を、リソースを予約したい各 kubelet 上で行います。

<!--
## API Object
-->
## API オブジェクト {#api-object}

<!--
Node is a top-level resource in the Kubernetes REST API. More details about the
API object can be found at:
[Node API object](/docs/reference/generated/kubernetes-api/{{< param "version" >}}/#node-v1-core).
-->
ノードは Kubernetes REST API の中でトップ・レベルのリソースです。
API オブジェクトの詳細については、 [Node API object](/jp/docs/reference/generated/kubernetes-api/{{< param "version" >}}/#node-v1-core) にあります。


{{% /capture %}}
