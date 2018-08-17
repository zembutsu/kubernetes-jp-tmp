---
title: コンテナ用の計算資源を管理
content_template: templates/concept
weight: 20
---

{{% capture overview %}}
<!---
When you specify a [Pod](/docs/concepts/workloads/pods/pod/), you can optionally specify how
much CPU and memory (RAM) each Container needs. When Containers have resource
requests specified, the scheduler can make better decisions about which nodes to
place Pods on. And when Containers have their limits specified, contention for
resources on a node can be handled in a specified manner. For more details about
the difference between requests and limits, see
[Resource QoS](https://git.k8s.io/community/contributors/design-proposals/node/resource-qos.md).
-->
[ポッド](/jp/docs/concepts/workloads/pods/pod/) の指定時、オプションでコンテナが必要とする CPU とメモリ（RAM）を指定できます。
コンテナでリソース要求（resource request）指定があれば、スケジューラはノードをどのポッドに置くかを決定するのにあたり、より良い判断ができます。
また、コンテナに制限（limit）の指定があれば、ノード上でリソースの競合が起こったとしても、指定した方式に従って処理できます。
要求と制限の違いについての詳細は [リソース QoS](https://git.k8s.io/community/contributors/design-proposals/node/resource-qos.md) をご覧ください。

{{% /capture %}}


{{% capture body %}}

<!--
## Resource types
-->
## リソース型 {#resource-type}

<!--
*CPU* and *memory* are each a *resource type*. A resource type has a base unit.
CPU is specified in units of cores, and memory is specified in units of bytes.
-->
*CPU* と *メモリ*  はどちらも *リソース型* です。
リソース型は基本単位（base unit）があります。
CPU はコア単位で指定子、メモリはバイト（byte）単位で指定します。

<!--
CPU and memory are collectively referred to as *compute resources*, or just
*resources*. Compute
resources are measurable quantities that can be requested, allocated, and
consumed. They are distinct from
[API resources](/docs/concepts/overview/kubernetes-api/). API resources, such as Pods and
[Services](/docs/concepts/services-networking/service/) are objects that can be read and modified
through the Kubernetes API server.
-->
CPU とメモリは *計算資源（compute resource）* や単なる *資源（リソース：resource）* として一緒に参照されます。
計算資源は測定可能な値のため、要求、割り当て、消費できます。
これらは [API リソース](/jp/docs/concepts/overview/kubernetes-api) とは異なります。
ポッドや [サービス](/jp/docs/concepts/services-networking/service/) のような API リソースはオブジェクト（object）であり、Kubernetes API サーバを通して読み込みや変更できます。

<!--
## Resource requests and limits of Pod and Container
-->
## ポッドとコンテナのリソース要求と制限 {#resource-requests-and-limits-of-pod-and-container}

<!--
Each Container of a Pod can specify one or more of the following:
-->
ポッドの各コントローラでは、以下の１つまたは複数を指定できます：

* `spec.containers[].resources.limits.cpu`
* `spec.containers[].resources.limits.memory`
* `spec.containers[].resources.requests.cpu`
* `spec.containers[].resources.requests.memory`

<!--
Although requests and limits can only be specified on individual Containers, it
is convenient to talk about Pod resource requests and limits. A
*Pod resource request/limit* for a particular resource type is the sum of the
resource requests/limits of that type for each Container in the Pod.
-->
１つ１つのコンテナに対して要求と制限をするのではなく、ポッドに対するリソース要求と制限が便利です。
個々のリソース型に対する *ポッド・リソース要求/制限* は、ポッド内にある各コンテナ用の型に対するリソース要求/制限の合計です。

<!--
## Meaning of CPU
-->
## CPU の意味

<!--
Limits and requests for CPU resources are measured in *cpu* units.
One cpu, in Kubernetes, is equivalent to:
-->
CPU リソースに対する制限と要求は、 *cpu*  単位で測ります。
Kubernetes における１つの CPU とは、以下と同等です：

<!--
- 1 AWS vCPU
- 1 GCP Core
- 1 Azure vCore
- 1 *Hyperthread* on a bare-metal Intel processor with Hyperthreading
-->
- 1 AWS vCPU
- 1 GCP コア
- 1 Azure vCore
- ベアメタルのハイパースレッディング対応インテル・プロセッサ上の 1 *ハイパースレッド*

<!--
Fractional requests are allowed. A Container with
`spec.containers[].resources.requests.cpu` of `0.5` is guaranteed half as much
CPU as one that asks for 1 CPU.  The expression `0.1` is equivalent to the
expression `100m`, which can be read as "one hundred millicpu". Some people say
"one hundred millicores", and this is understood to mean the same thing. A
request with a decimal point, like `0.1`, is converted to `100m` by the API, and
precision finer than `1m` is not allowed. For this reason, the form `100m` might
be preferred.
-->
わずかなリクエストでも可能です。
コンテナに対して `spec.containers[].resources.requests.cpu` を `0.5` で指定すると、１つの CPU に対して、CPU の半分を保証するように要求します。
`0.1` の表記は `100m` と表記するのと同じです。 呼び方は「100 ミリ CPU」です。
人によっては「100 ミリコア」と呼ぶ場合もありますが、どちらも同じだと理解されています。
`0.1` のような小数点での要求は 、API によって `100m` に変換されます。
また、`1m` よりも細かい値は許可されていません。
そのため、 `100m` と指定する方が望ましいでしょう。

<!--
CPU is always requested as an absolute quantity, never as a relative quantity;
0.1 is the same amount of CPU on a single-core, dual-core, or 48-core machine.
-->
CPU は相対値ではなく、常に絶対値で要求します。
つまり、１コア、２コア、あるいは48コアのマシンでも、  0.1 は CPU 量が同じです。

<!--
## Meaning of memory
-->
## メモリの意味 {#meaning-memory}

<!--
Limits and requests for `memory` are measured in bytes. You can express memory as
a plain integer or as a fixed-point integer using one of these suffixes:
E, P, T, G, M, K. You can also use the power-of-two equivalents: Ei, Pi, Ti, Gi,
Mi, Ki. For example, the following represent roughly the same value:
-->
`memory`（メモリ）に対する制限と要求はバイトで指定します。
メモリは単なる整数値か固定小数点で、末尾に E、P、T、G、M、K を付けます。
また、２の冪乗を表す Ei、Pi、Gi、Mi、Ki も使えます。
たとえば、以下はどれも同じ値を表しています：

```shell
128974848, 129e6, 129M, 123Mi
```

<!--
Here's an example.
The following Pod has two Containers. Each Container has a request of 0.25 cpu
and 64MiB (2<sup>26</sup> bytes) of memory. Each Container has a limit of 0.5
cpu and 128MiB of memory. You can say the Pod has a request of 0.5 cpu and 128
MiB of memory, and a limit of 1 cpu and 256MiB of memory.
-->
例を考えましょう。
以下のポッドは２つのコンテナがあります。
各コンテナは 0.25 cpu と 64MiB (2<sup>26</sup> バイト) のメモリを要求します。
各コンテナは 0.5 cpu と 128MiB のメモリ上限があります。
これで、ポッドには 0.5 cpu と 128MiB メモリの要求と、1 cpu と 256MiB メモリの上限があると言えます。

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: frontend
spec:
  containers:
  - name: db
    image: mysql
    env:
    - name: MYSQL_ROOT_PASSWORD
      value: "password"
    resources:
      requests:
        memory: "64Mi"
        cpu: "250m"
      limits:
        memory: "128Mi"
        cpu: "500m"
  - name: wp
    image: wordpress
    resources:
      requests:
        memory: "64Mi"
        cpu: "250m"
      limits:
        memory: "128Mi"
        cpu: "500m"
```

<!--
## How Pods with resource requests are scheduled
-->
## どのようにしてリソース要求があるポッドをスケジュールするか {#how-pods-with-resource-requests-are-scheduled}

<!--
When you create a Pod, the Kubernetes scheduler selects a node for the Pod to
run on. Each node has a maximum capacity for each of the resource types: the
amount of CPU and memory it can provide for Pods. The scheduler ensures that,
for each resource type, the sum of the resource requests of the scheduled
Containers is less than the capacity of the node. Note that although actual memory
or CPU resource usage on nodes is very low, the scheduler still refuses to place
a Pod on a node if the capacity check fails. This protects against a resource
shortage on a node when resource usage later increases, for example, during a
daily peak in request rate.
-->
ポッドの作成時、Kubernetes スケジューラはポッドを実行するためのスケジューラを選択します。
各ノードは各リソース型ごとに最大のキャパシティ（許容量）があります。
そのために、ポッドに対して提供可能な CPU とメモリの容量を提供します。
スケジューラが確保するのは、各リソース型ごとに、コンテナをスケジュールするためのリソース要求の合計が、ノードの許容量を下回るようにします。
ノード上で実際のメモリや CPU が少なくても、許容量の確認に失敗したら、スケジューラは対象ノードへのポッドを配置しません。
これは後からリソース使用量が増加しても、ノード上のリソース不足を防止するためです。
たとえば、要求比率（request rate）のピークが毎日ある場合です。

<!--
## How Pods with resource limits are run
-->
## どのようにしてポッドの実行時にリソース制限をするのか {#how-pods-with-resource-limits-are-run}

<!--
When the kubelet starts a Container of a Pod, it passes the CPU and memory limits
to the container runtime.
-->
ポッドのコンテナを kubelete が開始するとき、コンテナのランタイムに対して CPU とメモリ制限を渡します。


<!--
When using Docker:
-->
Docker の使用時：

<!--
- The `spec.containers[].resources.requests.cpu` is converted to its core value,
  which is potentially fractional, and multiplied by 1024. The greater of this number
  or 2 is used as the value of the
  [`--cpu-shares`](https://docs.docker.com/engine/reference/run/#/cpu-share-constraint)
  flag in the `docker run` command.

- The `spec.containers[].resources.limits.cpu` is converted to its millicore value and
  multiplied by 100. The resulting value is the total amount of CPU time that a container can use
  every 100ms. A container cannot use more than its share of CPU time during this interval.
-->
- `spec.containers[].resources.requests.cpu` はコアの値に変換します。1024 を割った数か倍数の可能性があります。
`docker run` コマンドの [`--cpu-shares`](https://docs.docker.com/engine/reference/run/#/cpu-share-constraint) フラグで指定する値として、この数より大きいか、あるいは 2 を指定します。

- `spec.containers[].resources.limits.cpu` はミリコアの値に変換され、100 を掛けます。結果の値は CPU 時間の合計となり、これはコンテナが 100ms ごとに使えるようになります。コンテナはこの間、指定した CPU 共有時間を越えて利用できません。

<!--
  {{< note >}}**Note**: The default quota period is 100ms. The minimum resolution of CPU quota is 1ms.{{</ note >}}
-->
  {{< note >}}**メモ**: デフォルトのクォータ期間は 100ms です。CPU クォータの最小単位は 1ms です。{{</ note >}}

<!--
- The `spec.containers[].resources.limits.memory` is converted to an integer, and
  used as the value of the
  [`--memory`](https://docs.docker.com/engine/reference/run/#/user-memory-constraints)
  flag in the `docker run` command.
-->
- `spec.containers[].resources.limits.memory` は整数に変換され、 `docker run` コマンドの [`--memory`](https://docs.docker.com/engine/reference/run/#/user-memory-constraints) フラグで指定する値に使います。

<!--
If a Container exceeds its memory limit, it might be terminated. If it is
restartable, the kubelet will restart it, as with any other type of runtime
failure.
-->
もしもコンテナがメモリ上限に到達すると、コンテナは終了させられるでしょう。
コンテナが再起動可能であれば、他のランタイム障害と同様に、 kubelet がコンテナを再起動します。

<!--
If a Container exceeds its memory request, it is likely that its Pod will
be evicted whenever the node runs out of memory.
-->
もしもコンテナがメモリ要求に到達すると、メモリ不足ではないノードにポッドが退避される場合があります。

<!--
A Container might or might not be allowed to exceed its CPU limit for extended
periods of time. However, it will not be killed for excessive CPU usage.
-->
コンテナは CPU 制限を一定期間に超過する場合があるかもしれません。
しかし、極度の CPU 使用がなければ強制停止しません。


<!--
To determine whether a Container cannot be scheduled or is being killed due to
resource limits, see the
[Troubleshooting](#troubleshooting) section.
-->
リソース制限によって、コンテナをスケジュールできないか、あるいは強制停するかを決定する方法は、[トラブルシューティング](#troubleshooting) セクションをご覧ください。

<!--
## Monitoring compute resource usage
-->
## 計算資源の使用を監視

<!--
The resource usage of a Pod is reported as part of the Pod status.
-->

<!--
If [optional monitoring](http://releases.k8s.io/{{< param "githubbranch" >}}/cluster/addons/cluster-monitoring/README.md)
is configured for your cluster, then Pod resource usage can be retrieved from
the monitoring system.
-->

<!--
## Troubleshooting
-->
## トラブルシューティング {#troubleshooting}

<!--
### My Pods are pending with event message failedScheduling
-->
### ポッドのイベント・メッセージが failedScheduling のまま待機中です {#my-pods-are-pending-with-event-message-failedscheduling}

<!--
If the scheduler cannot find any node where a Pod can fit, the Pod remains
unscheduled until a place can be found. An event is produced each time the
scheduler fails to find a place for the Pod, like this:
-->
スケジューラがポッドに適合するノードを見つけられなければ、場所が見つかるまでポッドはスケジュール不可能（unscheduled）のままです。以下のように、ポッドに適した場所をポッドが見つけられなければ、毎回イベントを生成します。

```shell
$ kubectl describe pod frontend | grep -A 3 Events
Events:
  FirstSeen LastSeen   Count  From          Subobject   PathReason      Message
  36s   5s     6      {scheduler }              FailedScheduling  Failed for reason PodExceedsFreeCPU and possibly others
```

<!--
In the preceding example, the Pod named "frontend" fails to be scheduled due to
insufficient CPU resource on the node. Similar error messages can also suggest
failure due to insufficient memory (PodExceedsFreeMemory). In general, if a Pod
is pending with a message of this type, there are several things to try:
-->
こちらの例では、ポッド名「frontend」はノード上で CPU リソースが不足しているため、スケジュールに失敗しています。
また、メモリ不足によっても同様のエラーメッセージが出る場合もあります（PodExceedsFreeMemory）。
通常、ポッドが保留中（Pending）のメッセージが出る場合には、試すことがいくつかあります：

<!--
- Add more nodes to the cluster.
- Terminate unneeded Pods to make room for pending Pods.
- Check that the Pod is not larger than all the nodes. For example, if all the
  nodes have a capacity of `cpu: 1`, then a Pod with a request of `cpu: 1.1` will
  never be scheduled.
-->
- クラスタに更にノードを追加する。
- 保留中ポッドに空きをつくるために、不要なポッドを終了する。
- ポッドがノード全体よりも大きくないかどうか確認する。たとえば、全ノードの許容量が `cpu: 1` なのに、ポッドに対する要求が `cpu: 1.1`の場合、決してスケジュールされません。

<!--
You can check node capacities and amounts allocated with the
`kubectl describe nodes` command. For example:
-->
`kubectl describe nodes` コマンドで、ノードの許容量と割当量を確認できます。
サンプル：

```shell
$ kubectl describe nodes e2e-test-minion-group-4lw4
Name:            e2e-test-minion-group-4lw4
[ ... lines removed for clarity ...]
Capacity:
 cpu:                               2
 memory:                            7679792Ki
 pods:                              110
Allocatable:
 cpu:                               1800m
 memory:                            7474992Ki
 pods:                              110
[ ... lines removed for clarity ...]
Non-terminated Pods:        (5 in total)
  Namespace    Name                                  CPU Requests  CPU Limits  Memory Requests  Memory Limits
  ---------    ----                                  ------------  ----------  ---------------  -------------
  kube-system  fluentd-gcp-v1.38-28bv1               100m (5%)     0 (0%)      200Mi (2%)       200Mi (2%)
  kube-system  kube-dns-3297075139-61lj3             260m (13%)    0 (0%)      100Mi (1%)       170Mi (2%)
  kube-system  kube-proxy-e2e-test-...               100m (5%)     0 (0%)      0 (0%)           0 (0%)
  kube-system  monitoring-influxdb-grafana-v4-z1m12  200m (10%)    200m (10%)  600Mi (8%)       600Mi (8%)
  kube-system  node-problem-detector-v0.1-fj7m3      20m (1%)      200m (10%)  20Mi (0%)        100Mi (1%)
Allocated resources:
  (Total limits may be over 100 percent, i.e., overcommitted.)
  CPU Requests    CPU Limits    Memory Requests    Memory Limits
  ------------    ----------    ---------------    -------------
  680m (34%)      400m (20%)    920Mi (12%)        1070Mi (14%)
```

<!--
In the preceding output, you can see that if a Pod requests more than 1120m
CPUs or 6.23Gi of memory, it will not fit on the node.
-->
先ほどの出力では、ポッドの要求が 1120m CPU と 6.23Gi メモリですが、該当するノードがありません。

<!--
By looking at the `Pods` section, you can see which Pods are taking up space on
the node.
-->
`Pods` （ポッド）のセクションを見ますと、どのノード上で空きを確保するかが分かります。

<!--
The amount of resources available to Pods is less than the node capacity, because
system daemons use a portion of the available resources. The `allocatable` field
[NodeStatus](/docs/reference/generated/kubernetes-api/{{< param "version" >}}/#nodestatus-v1-core)
gives the amount of resources that are available to Pods. For more information, see
[Node Allocatable Resources](https://git.k8s.io/community/contributors/design-proposals/node/node-allocatable.md).
-->
ポッドが利用可能なリソース量がノード許容量よりも少ないのは、利用可能なリソースの一部をシステム・デーモンが使うからです。
`allocatable` （割り当て可能）フィールドは [NodeStatus](/docs/reference/generated/kubernetes-api/{{< param "version" >}}/#nodestatus-v1-core) がポッドで利用可能なリソース量を提供します。
詳しい情報は [Node に割り当て可能なリソース](https://git.k8s.io/community/contributors/design-proposals/node/node-allocatable.md) をご覧ください。


<!--
The [resource quota](/docs/concepts/policy/resource-quotas/) feature can be configured
to limit the total amount of resources that can be consumed. If used in conjunction
with namespaces, it can prevent one team from hogging all the resources.
-->
[リソース・クォータ（resource quota）](/jp/docs/concepts/policy/resource-quotas/) 機能によって、消費する可能性のあるリソースの合計量を制限する設定ができます。
名前空間と一緒に使うと、あるチームによって全てのリソースを消費され尽くすのを防止します。

<!--
### My Container is terminated
-->
### コンテナが終了させられる {#my-container-is-terminated}

<!--
Your Container might get terminated because it is resource-starved. To check
whether a Container is being killed because it is hitting a resource limit, call
`kubectl describe pod` on the Pod of interest:
-->
リソースの欠乏によって、コンテナが終了させられる場合があります。
コンテナがリソース上限によって強制停止させられたかどうかを調べるには、対象のポッドに対して `kubectl describe pod` を呼び出します。


```shell
[12:54:41] $ kubectl describe pod simmemleak-hra99
Name:                           simmemleak-hra99
Namespace:                      default
Image(s):                       saadali/simmemleak
Node:                           kubernetes-node-tf0f/10.240.216.66
Labels:                         name=simmemleak
Status:                         Running
Reason:
Message:
IP:                             10.244.2.75
Replication Controllers:        simmemleak (1/1 replicas created)
Containers:
  simmemleak:
    Image:  saadali/simmemleak
    Limits:
      cpu:                      100m
      memory:                   50Mi
    State:                      Running
      Started:                  Tue, 07 Jul 2015 12:54:41 -0700
    Last Termination State:     Terminated
      Exit Code:                1
      Started:                  Fri, 07 Jul 2015 12:54:30 -0700
      Finished:                 Fri, 07 Jul 2015 12:54:33 -0700
    Ready:                      False
    Restart Count:              5
Conditions:
  Type      Status
  Ready     False
Events:
  FirstSeen                         LastSeen                         Count  From                              SubobjectPath                       Reason      Message
  Tue, 07 Jul 2015 12:53:51 -0700   Tue, 07 Jul 2015 12:53:51 -0700  1      {scheduler }                                                          scheduled   Successfully assigned simmemleak-hra99 to kubernetes-node-tf0f
  Tue, 07 Jul 2015 12:53:51 -0700   Tue, 07 Jul 2015 12:53:51 -0700  1      {kubelet kubernetes-node-tf0f}    implicitly required container POD   pulled      Pod container image "k8s.gcr.io/pause:0.8.0" already present on machine
  Tue, 07 Jul 2015 12:53:51 -0700   Tue, 07 Jul 2015 12:53:51 -0700  1      {kubelet kubernetes-node-tf0f}    implicitly required container POD   created     Created with docker id 6a41280f516d
  Tue, 07 Jul 2015 12:53:51 -0700   Tue, 07 Jul 2015 12:53:51 -0700  1      {kubelet kubernetes-node-tf0f}    implicitly required container POD   started     Started with docker id 6a41280f516d
  Tue, 07 Jul 2015 12:53:51 -0700   Tue, 07 Jul 2015 12:53:51 -0700  1      {kubelet kubernetes-node-tf0f}    spec.containers{simmemleak}         created     Created with docker id 87348f12526a
```

<!--
In the preceding example, the `Restart Count:  5` indicates that the `simmemleak`
Container in the Pod was terminated and restarted five times.
-->
こちらの例では、 `Restart Cound: 5`が示すのは、ポッド内の `simmemleak` コンテナが停止と再起動を５回行いました。

<!--
You can call `kubectl get pod` with the `-o go-template=...` option to fetch the status
of previously terminated Containers:
-->
`kubectl get pod` with the `-o go-template=...` オプションで、停止した状態になっている以前のコンテナの状態を確認できます。


```shell
[13:59:01] $ kubectl get pod -o go-template='{{range.status.containerStatuses}}{{"Container Name: "}}{{.name}}{{"\r\nLastState: "}}{{.lastState}}{{end}}'  simmemleak-hra99
Container Name: simmemleak
LastState: map[terminated:map[exitCode:137 reason:OOM Killed startedAt:2015-07-07T20:58:43Z finishedAt:2015-07-07T20:58:43Z containerID:docker://0e4095bba1feccdfe7ef9fb6ebffe972b4b14285d5acdec6f0d3ae8a22fad8b2]]{% endraw %}
```

<!--
You can see that the Container was terminated because of `reason:OOM Killed`,
where `OOM` stands for Out Of Memory.
-->
`OOM` と書かれている場所は Out Of Memory （メモリ不足）のためであり、 `reason:OOM Killed` によってコンテナが停止されたのが分かります。

<!--
## Local ephemeral storage
-->
## ローカル短期的ストレージ（Local ephemeral storage）

{{< feature-state state="beta" >}}

<!--
Kubernetes version 1.8 introduces a new resource, _ephemeral-storage_ for managing local ephemeral storage. In each Kubernetes node, kubelet's root directory (/var/lib/kubelet by default) and log directory (/var/log) are stored on the root partition of the node. This partition is also shared and consumed by pods via EmptyDir volumes, container logs, image layers and container writable layers.
-->
Kubernetes バージョン 1.8 で新しいリソース、 _ephemeral-storage（短期的ストレージ）_ が導入されました。
これはローカルの短期的なストレージを管理するためです。
各 Kubernetes ノードでは、kubelet のルート・ディレクトリ（ /var/lib/kubelet がデフォルト）とログ・ディレクトリ（/var/log）はノードのルート・パーティションに格納されます。
このパーティションは、ポッドを通して共有および消費されます。用途は  EmptyDir ボリューム、コンテナのログ、イメージ・レイヤ、コンテナから書き込み可能なレイヤです。

<!--
This partition is “ephemeral” and applications cannot expect any performance SLAs (Disk IOPS for example) from this partition. Local ephemeral storage management only applies for the root partition; the optional partition for image layer and writable layer is out of scope.
-->
このパーティションは「ephemeral」（エフェメラル、短命）であり、アプリケーションはこのパーティションからはあらゆる性能  SLA（たとえばディスクの IOPS） を期待できません。
ローカル短期的ストレージの管理が適用できるのは、ルート・パーティションのみです。
つまり、イメージ・レイヤと書き込み可能なレイヤ用途のオプションのパーティションは範囲外です。

{{< note >}}
<!--
**Note:** If an optional runtime partition is used, root partition will not hold any image layer or writable layers.
-->
**メモ:** オプションのランタイム・パーティションが使われている場合、ルート・パーティションはイメージ・レイヤや書き込み可能なレイヤを一切保持しません。
{{< /note >}}

<!--
### Requests and limits setting for local ephemeral storage
-->
### ローカル短期的ストレージの要求と制限を設定 {#requests-and-limits-setting-for-local-ephemeral-storage}

<!--
Each Container of a Pod can specify one or more of the following:
-->
ポッドの各コンテナに対して、以下の１つまたは複数を指定できます。

* `spec.containers[].resources.limits.ephemeral-storage`
* `spec.containers[].resources.requests.ephemeral-storage`

<!--
Limits and requests for `ephemeral-storage` are measured in bytes. You can express storage as
a plain integer or as a fixed-point integer using one of these suffixes:
E, P, T, G, M, K. You can also use the power-of-two equivalents: Ei, Pi, Ti, Gi,
Mi, Ki. For example, the following represent roughly the same value:
-->
`ephemeral-storage`（短期的ストレジ）に対する制限と要求はバイトで指定します。
ストレージを表すのはは単なる整数値か固定小数点で、末尾に E、P、T、G、M、K を付けます。
また、２の冪乗を表す Ei、Pi、Gi、Mi、Ki も使えます。
たとえば、以下はどれも同じ値を表しています：

```shell
128974848, 129e6, 129M, 123Mi
```

<!--
For example, the following Pod has two Containers. Each Container has a request of 2GiB of local ephemeral storage. Each Container has a limit of 4GiB of local ephemeral storage. Therefore, the Pod has a request of 4GiB of local ephemeral storage, and a limit of 8GiB of storage.
-->
たとえば、以下のポッドには２つのコンテナがあります。
各コンテナは 2GiB のローカル短期的ストレージを要求します。
各コンテナは 4GiB のローカル短期的ストレージの上限があります。
そのため、ポッドは 4GiB のローカル短期的ストレージを要求し、ストレージの上限は 8GiB です。


```yaml
apiVersion: v1
kind: Pod
metadata:
  name: frontend
spec:
  containers:
  - name: db
    image: mysql
    env:
    - name: MYSQL_ROOT_PASSWORD
      value: "password"
    resources:
      requests:
        ephemeral-storage: "2Gi"
      limits:
        ephemeral-storage: "4Gi"
  - name: wp
    image: wordpress
    resources:
      requests:
        ephemeral-storage: "2Gi"
      limits:
        ephemeral-storage: "4Gi"
```

<!--
### How Pods with ephemeral-storage requests are scheduled
-->

### ポッドはどのようにして ephemeral-storage リクエストをするのか {#how-pods-with-ephemeral-storage-requests-are-scheduled}

<!--
When you create a Pod, the Kubernetes scheduler selects a node for the Pod to
run on. Each node has a maximum amount of local ephemeral storage it can provide for Pods. (For more information, see ["Node Allocatable"](/docs/tasks/administer-cluster/reserve-compute-resources/#node-allocatable) The scheduler ensures that the sum of the resource requests of the scheduled Containers is less than the capacity of the node.
-->
ポッドを作成する時、Kubernetes スケジューラは、ポッドを実行するノードを選択します。
各ノードには、ポッドのローカル短期的ストレージに割り当て可能な最大量があります
（詳しい情報は ["ノード割り当て"](/jp/docs/tasks/administer-cluster/reserve-compute-resources/#node-allocatable）をご覧ください。）。
スケジューラはスケジュールされるコンテナのリソース要求の合計が、ノードの許容量よりも低くなる状況を確保します。

<!--
### How Pods with ephemeral-storage limits run
-->
### どのようにしてポッドに対して ephemeral-storage の制限をするか {#how-pods-with-ephemeral-storage-limits-run}

<!--
For container-level isolation, if a Container's writable layer and logs usage exceeds its storage limit, the pod will be evicted. For pod-level isolation, if the sum of the local ephemeral storage usage from all containers and also the pod's EmptyDir volumes exceeds the limit, the pod will be evicted.
-->
コンテナ・レベルの分離（isolation）では、もしもコンテナの書き込み可能なレイヤとログがストレージ上限に到達すると、ポッドは退避されます。
ポッド・レベルの分離（isolation）では、もしも全てのコンテナのローカル短期的ストレージ容量の合計と、ポッドの EmptyDir ボリュームが上限に達しても、ポッドが退避されます。

<!--
## Extended Resources
-->
## 拡張リソース {#extended-resources}

<!--
Extended Resources are fully-qualified resource names outside the
`kubernetes.io` domain. They allow cluster operators to advertise and users to
consume the non-Kubernetes-built-in resources.
-->
拡張されたリソースは、 `kubernetes.io` の完全修飾（省略されていない）リソース名の外にあります。
そのため、クラスタの運用担当者（オペレータ）は Kubernetes で作成されていないリソースを通知（advertise）する必要があります。

<!--
There are two steps required to use Extended Resources. First, the cluster
operator must advertise an Extended Resource. Second, users must request the
Extended Resource in Pods.
-->
拡張リソースを使うためには２つの手順が必要です。
まず、クラスタ運用担当者は拡張リソースを通知（advertise）します。
次に、ユーザはポッドの中で拡張リソースを要求する必要があります。

<!--
### Managing extended resources
-->
### 拡張リソースの管理 {#managing-extended-resources}

<!--
#### Node-level extended resources
-->
#### ノード・レベル拡張リソース {#node-level-extended-resoruces}

<!--
Node-level extended resources are tied to nodes.
-->
ノード・レベルの拡張リソースはノードに紐付きます。

<!--
##### Device plugin managed resources
-->
##### デバイス・プラグインの管理リソース {#device-plugin-managed-resoruces}

<!--
See [Device
Plugin](/docs/concepts/extend-kubernetes/compute-storage-net/device-plugins/)
for how to advertise device plugin managed resources on each node.
-->
各ノード上でデバイス・プラグインが管理するリソースを通知（advertise）する方法は、[デバイス・プラグイン](/jp/docs/concepts/extend-kubernetes/compute-storage-net/device-plugins/) をご覧ください。

<!--
##### Other resources
--> 
##### 他のリソース {#other-resources}

<!--
To advertise a new node-level extended resource, the cluster operator can
submit a `PATCH` HTTP request to the API server to specify the available
quantity in the `status.capacity` for a node in the cluster. After this
operation, the node's `status.capacity` will include a new resource. The
`status.allocatable` field is updated automatically with the new resource
asynchronously by the kubelet. Note that because the scheduler uses the	node
`status.allocatable` value when evaluating Pod fitness, there may be a short
delay between patching the node capacity with a new resource and the first pod
that requests the resource to be scheduled on that node.
-->
新しいノード・レベル拡張リソースを通知（advertise）するには、クラスタ運用担当者はクラスタ内のノードに対して、 `status.capacity` で利用可能な量を指定し、API サーバに対して `PATCH`HTTP リクエストを送信できます。
この作業の後、ノードの `status.capacity` に新しいリソースが含まれます。
`status.allocatable` フィールドは kubelete によって非同期で新しいリソースが自動的に更新されます。
スケジューラはノードの `status.allocatable` 値をポッドが一致するかどうかの評価時に使うため、新しいノードの許容量（キャパシティ）が適用されるのと、ノード上に新しいリソースと１つめのポッドを要求したリソースが利用可能になるまでは、時間差（タイムラグ）がありますのでご注意ください。

<!--
**Example:**
-->
**例：**

<!--
Here is an example showing how to use `curl` to form an HTTP request that
advertises five "example.com/foo" resources on node `k8s-node-1` whose master
is `k8s-master`.
-->
ここにある例は `curl` を使って HTTP リクエストをする方法です。５つの `example.com/foo` リソースがノード `k8s-node-1` 上にあるのを、マスタ `k8s-master` に通知（advertise）します。


```shell
curl --header "Content-Type: application/json-patch+json" \
--request PATCH \
--data '[{"op": "add", "path": "/status/capacity/example.com~1foo", "value": "5"}]' \
http://k8s-master:8080/api/v1/nodes/k8s-node-1/status
```

{{< note >}}
<!--
**Note**: In the preceding request, `~1` is the encoding for the character `/`
in the patch path. The operation path value in JSON-Patch is interpreted as a
JSON-Pointer. For more details, see
[IETF RFC 6901, section 3](https://tools.ietf.org/html/rfc6901#section-3).
-->
**メモ**: 前述のリクエストで、パッチ・パスにおける `~1` は、文字列 `/` のエンコーディングです。
JSON-PATCH における操作パスの値は、JSON-Pointer としても解釈されます。
詳細は [IETF RFC 6901, section 3](https://tools.ietf.org/html/rfc6901#section-3) をご覧ください。
{{< /note >}}

<!--
#### Cluster-level extended resources
-->
#### クラスタ・レベル拡張リソース {#cluster-level-extended-resources}

<!--
Cluster-level extended resources are not tied to nodes. They are usually managed
by scheduler extenders, which handle the resource comsumption, quota and so on.
-->
クラスタ・レベルの拡張リソースはノードと紐付きません。
これらは通常、スケジューラ拡張（scheduler extenders）によって管理され、リソースの消費やクォータ（制限）などが扱われます。

<!--
You can specify the extended resources that are handled by scheduler extenders
in [scheduler policy
configuration](https://github.com/kubernetes/kubernetes/blob/release-1.10/pkg/scheduler/api/v1/types.go#L31).
-->
拡張リソースを使うには、スケジューラ拡張によって扱われるようにするため、 [スケジューラ方針設定（scheduler policy
configuration）](https://github.com/kubernetes/kubernetes/blob/release-1.10/pkg/scheduler/api/v1/types.go#L31)で行います。


<!--
**Example:**
-->
**例：**

<!--
The following configuration for a scheduler policy indicates that the
cluster-level extended resource "example.com/foo" is handled by scheduler
extender.
-->
以下のスケジューラ方針用設定では、クラスタ・レベル拡張リソース「example.com/foo」がスケジューラ extender によって扱われます。

<!--
 - The scheduler sends a pod to the scheduler extender only if the pod requests
   "example.com/foo".
 - The `ignoredByScheduler` field specifies that the scheduler does not check
   the "example.com/foo" resource in its `PodFitsResources` predicate.
-->
 - ポッドが「example.com/foo」を要求する時のみ、スケジューラはポッドに対してスケジューラ extender を送る。
 - `ignoredByScheduler` フィールドの指定があれば、「example.com/foo」リソースの基礎となる `PodFitsResources` を確認しません。

```json
{
  "kind": "Policy",
  "apiVersion": "v1",
  "extenders": [
    {
      "urlPrefix":"<extender-endpoint>",
      "bindVerb": "bind",
      "managedResources": [
        {
          "name": "example.com/foo",
          "ignoredByScheduler": true
        }
      ]
    }
  ]
}
```

<!--
### Consuming extended resources
-->
### 拡張リソースの確保 {#consuming-extended-resources}

<!--
Users can consume Extended Resources in Pod specs just like CPU and memory.
The scheduler takes care of the resource accounting so that no more than the
available amount is simultaneously allocated to Pods.
-->
ユーザはポッドの spec で、CPU やメモリのように拡張リソースを確保できます。
スケジューラはリソースの容量計算を扱いますが、利用可能な量をポッドに擬似的に割り当てるにすぎません。

<!--
The API server restricts quantities of Extended Resources to whole numbers.
Examples of _valid_ quantities are `3`, `3000m` and `3Ki`. Examples of
_invalid_ quantities are `0.5` and `1500m`.
-->
API サーバは拡張リソースの量の指定を、数値のみに制限しています。
例えば、 _有効な_ 量は  `3`、 `3000m`、 `3Ki` です。
_無効な_ 量の例は `0.5` と `1500m` です。

{{< note >}}
<!--
**Note:** Extended Resources replace Opaque Integer Resources.
Users can use any domain name prefix other than "`kubernetes.io`" which is reserved.
-->
**メモ：** 拡張リソースは不透明な整数リソースを置き換えます。
ユーザは予約されている「kubernetes.io」以外のあらゆるドメイン名を使えます。
{{< /note >}}

<!--
To consume an Extended Resource in a Pod, include the resource name as a key
in the `spec.containers[].resources.limits` map in the container spec.
-->
ポッドで拡張リソースを確保するには、コンテナの spec で `spec.containers[].resources.limits`  のキーをリソース名に含めます。

{{< note >}}
<!--
**Note:** Extended resources cannot be overcommitted, so request and limit
must be equal if both are present in a container spec.
-->
**メモ：** 拡張リソースはオーバーコミットできません。
そのため、コンテナ spec 内にある要求と制限は、どちらも同じにする必要があります。
{{< /note >}}

<!--
A Pod is scheduled only if all of the resource requests are satisfied, including
CPU, memory and any Extended Resources. The Pod remains in the `PENDING` state
as long as the resource request cannot be satisfied.
-->
全てのリソース要求が満たされた場合のみ、ポッドはスケジュールされます。
リソース要求には CPU、メモリ、拡張リソースを含みます。
リソース要求が満たされない場合は、ポッドは `PENDING`（保留中）の状態のままです。

<!--
**Example:**
-->
**例：**

<!--
The Pod below requests 2 CPUs and 1 "example.com/foo" (an extended resource).
-->
以下のポッドは 2 CPU と「example.com/foo」（拡張リソース）を要求します。

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: my-pod
spec:
  containers:
  - name: my-container
    image: myimage
    resources:
      requests:
        cpu: 2
        example.com/foo: 1
      limits:
        example.com/foo: 1
```

<!--
## Planned Improvements
-->
## 計画済みの改良 {#planned-improvements}

<!--
Kubernetes version 1.5 only allows resource quantities to be specified on a
Container. It is planned to improve accounting for resources that are shared by
all Containers in a Pod, such as
[emptyDir volumes](/docs/concepts/storage/volumes/#emptydir).
-->
Kubernetes バージョン 1.5 では、コンテナに対してのみリソース量を指定できます。
[emptyDir ボリューム](/jp/docs/concepts/storage/volumes/#emptydir) など、ポッド内の全てのコンテナが共有できるような、他のリソースも扱えるように改良を計画しています。

<!--
Kubernetes version 1.5 only supports Container requests and limits for CPU and
memory. It is planned to add new resource types, including a node disk space
resource, and a framework for adding custom
[resource types](https://github.com/kubernetes/community/blob/{{< param "githubbranch" >}}/contributors/design-proposals/scheduling/resources.md).
-->
Kubernetes バージョン 1.5 でサポートしているのはコンテナに対する CPU とメモリに対する要求と制限です。
ディスク容量リソースを含む新しいリソースタイプと、カスタム [リソース・タイプ](https://github.com/kubernetes/community/blob/{{< param "githubbranch" >}}/contributors/design-proposals/scheduling/resources.md) の追加が計画されています。

<!--
Kubernetes supports overcommitment of resources by supporting multiple levels of
[Quality of Service](http://issue.k8s.io/168).
-->
Kubernetes は、複数レベルの [サービス品質（Quality of Service）](http://issue.k8s.io/168) による、リソースのオーバーコミットをサポートします。

<!--
In Kubernetes version 1.5, one unit of CPU means different things on different
cloud providers, and on different machine types within the same cloud providers.
For example, on AWS, the capacity of a node is reported in
[ECUs](http://aws.amazon.com/ec2/faqs/), while in GCE it is reported in logical
cores. We plan to revise the definition of the cpu resource to allow for more
consistency across providers and platforms.
-->
Kubernetes バージョン 1.5 では、CPU の１単位がクラウド事業者によって異なります。
また、同じクラウド事業者にも様々な種類のマシンがあります。
たとえば、AWS では、ノードの許容量は [ECU](http://aws.amazon.com/ec2/faqs/) で表されますが、GCE は論理コアです。
私達は CPU リソースの定義を改定し、プロバイダやプラットフォームを横断して一貫性を保てるようにします。
{{% /capture %}}


{{% capture whatsnext %}}

<!--
* Get hands-on experience [assigning Memory resources to containers and pods](/docs/tasks/configure-pod-container/assign-memory-resource/).

* Get hands-on experience [assigning CPU resources to containers and pods](/docs/tasks/configure-pod-container/assign-cpu-resource/).

* [Container](/docs/reference/generated/kubernetes-api/{{< param "version" >}}/#container-v1-core)

* [ResourceRequirements](/docs/reference/generated/kubernetes-api/{{< param "version" >}}/#resourcerequirements-v1-core)
-->
* [メモリ・リソースをコンテナとポッドに割り当てる](/jp/docs/tasks/configure-pod-container/assign-memory-resource/) ハンズオン経験をする。
* [CPU リソースをコンテナとポッドに割り当てる](/jp/docs/tasks/configure-pod-container/assign-cpu-resource/) ハンズオン経験をする。
* [Container](/jp/docs/reference/generated/kubernetes-api/{{< param "version" >}}/#container-v1-core)
* [ResourceRequirements](/jp/docs/reference/generated/kubernetes-api/{{< param "version" >}}/#resourcerequirements-v1-core)

{{% /capture %}}
