---
title: コンテナとポッドにメモリ・リソースを割り当て
content_template: templates/task
weight: 10
---

{{% capture overview %}}
<!--
This page shows how to assign a memory *request* and a memory *limit* to a
Container. A Container is guaranteed to have as much memory as it requests,
but is not allowed to use more memory than its limit.
-->
このページはコンテナに対してメモリ要求（ *request* ）とメモリ上限（ *limit* ）を割り当てる方法を説明します。コンテナは要求があったメモリ保持を保証します。しかし、上限を超えるメモリ使用を許可しません。

{{% /capture %}}


{{% capture prerequisites %}}

{{< include "task-tutorial-prereqs.md" >}} {{< version-check >}}

<!--
Each node in your cluster must have at least 300 MiB of memory.
-->
クラスタの各ノードでは、最小 300 MiB のメモリが必要です。

<!--
A few of the steps on this page require you to run the
[metrics-server](https://github.com/kubernetes-incubator/metrics-server)
service in your cluster. If you don't have metrics-server
+running, you can skip those steps.
-->
このページのいくつかのステップで [metrics-server](https://github.com/kubernetes-incubator/metrics-server) サービスをクラスタでの実行が必要になります。もしも metrics-server を持っておらず実行中でもなければ、各ステップを省略（スキップ）できます。

<!--
If you are running minikube, run the following command to enable
metrics-server:
-->
minikube を実行中であれば、以下のコマンドを実行して metrics-server を有効化します。


```shell
minikube addons enable metrics-server
```

<!--
To see whether metrics-server (or another provider of the resource metrics
API, `metrics.k8s.io`) is running, enter this command:
-->
metrics-server（あるいは他のリソース・メトリクス API `metrics.k8s.io` を提供するもの ）が事項中かどうかを調べるには、こちらのコマンドを入力します：

```shell
kubectl get apiservices
```

<!--
If the resource metrics API is available, the output will include a
reference to `metrics.k8s.io`.
-->
リソース・メトリクス API が利用可能であれば、出力結果に `metrics.k8s.io` が含まれているのが分かります。


```shell
NAME      
v1beta1.metrics.k8s.io
```

{{% /capture %}}


{{% capture steps %}}

<!--
## Create a namespace
-->
## 名前空間の作成 {#create-a-namespace}

<!--
Create a namespace so that the resources you create in this exercise are
isolated from the rest of your cluster.
-->
この演習で作成するリソースがクラスタの他へ影響を与えないよう隔てるために、名前空間を作成します。


```shell
kubectl create namespace mem-example
```

<!--
## Specify a memory request and a memory limit
-->
## メモリ要求とメモリ上限の指定 {#specify-a-memory-request-and-a-memory-limit}

<!--
To specify a memory request for a Container, include the `resources:requests` field
in the Container's resource manifest. To specify a memory limit, include `resources:limits`.
-->
コンテナに対してメモリ要求（request）を指定するには、コンテナのリソース・マニフェストを `resources:request`に含めます。メモリ上限（limit）を指定するには `resources:imits` に含めます。

<!--
In this exercise, you create a Pod that has one Container. The Container has a memory
request of 100 MiB and a memory limit of 200 MiB. Here's the configuration file
for the Pod:
-->
この演習では１つのコンテナがあるポッドを作成します。コンテナはメモリ要求が 100 MiB、メモリ上限が 200 MiB です。こちらはポッド用の設定ファイルです。

{{< code file="memory-request-limit.yaml" >}}

<!--
In the configuration file, the `args` section provides arguments for the Container when it starts.
The `"--vm-bytes", "150M"` arguments tell the Container to attempt to allocate 150 MiB of memory.
-->
設定ファイル中で、 `args` の部分で指定する引数はコンテナの起動時に使います。`"--vm-bytes", "150M"` の引数は、コンテナに対してメモリ 150 MiB の割り当てを試みていると伝えます。

<!--
Create the Pod:
-->
ポッドを作成します：

```shell
kubectl create -f https://k8s.io/docs/tasks/configure-pod-container/memory-request-limit.yaml --namespace=mem-example
```

<!--
Verify that the Pod's Container is running:
-->
ポッドのコンテナが稼働中なのを確認します：

```shell
kubectl get pod memory-demo --namespace=mem-example
```

<!--
View detailed information about the Pod:
-->
ポッドに関する詳細情報を表示します。

```shell
kubectl get pod memory-demo --output=yaml --namespace=mem-example
```

<!--
The output shows that the one Container in the Pod has a memory request of 100 MiB
and a memory limit of 200 MiB.
-->
出力結果から、ポッド内のコンテナには 100MiB のメモリ要求と 200 Mib のメモリ上限があるのが分かります。


```yaml
...
resources:
  limits:
    memory: 200Mi
  requests:
    memory: 100Mi
...
```

<!--
Use `kubectl top` to fetch the metrics for the pod:
-->
`kubectl top` を使い、ポッドに対する数値（メトリクス）を取得します：


```shell
kubectl top pod memory-demo
```

<!--
The output shows that the Pod is using about 162,900,000 bytes of memory, which
is about 150 MiB. This is greater than the Pod's 100 MiB request, but within the
Pod's 200 MiB limit.
-->
出力結果からポッドは 162,900,000 バイトのメモリを使用中と表示しており、おおよそ 150 MiB です。これはポッドに対する 100 MiB 要求よりは大きいですが、ポッドの 200 MiB 上限内です。

```
NAME                        CPU(cores)   MEMORY(bytes)
memory-demo                 <something>  162856960
```

<!--
Delete your Pod:
-->
ポッドを削除します：

```shell
kubectl delete pod memory-demo --namespace=mem-example
```

<!--
## Exceed a Container's memory limit
-->
## コンテナのメモリ上限の超過 {#exceed-a-containers-memory-limit}

<!--
A Container can exceed its memory request if the Node has memory available. But a Container
is not allowed to use more than its memory limit. If a Container allocates more memory than
its limit, the Container becomes a candidate for termination. If the Container continues to
consume memory beyond its limit, the Container is terminated. If a terminated Container is
restartable, the kubelet will restart it, as with any other type of runtime failure.
-->
ノードでメモリを利用可能であれば、コンテナはメモリ要求よりも多く利用できます。しかし、メモリ上限よりも多くの利用は許可されません。メモリ上限を超えてコンテナにメモリが割り当てられると、コンテナは終了候補（candidate for termination）となります。もしも、コンテナが上限を超えてメモリを使用し続けると、コンテナは終了させられます。終了したコンテナが再起動可能であれば、他の何らかのランタイム障害と同様、kubelet はコンテナ再起動します。

<!---
In this exercise, you create a Pod that attempts to allocate more memory than its limit.
Here is the configuration file for a Pod that has one Container. The Container has a
memory request of 50 MiB and a memory limit of 100 MiB.
-->
この演習では、作成したポッドに対して上限を超えてメモリの割り当てを試みます。ここにあるポッド用の設定ファイルは１つのコンテナを持ちます。コンテナはメモリ要求 50MiB とメモリ上限 100MB です。

{{< code file="memory-request-limit-2.yaml" >}}

<!--
In the configuration file, in the `args` section, you can see that the Container
will attempt to allocate 250 MiB of memory, which is well above the 100 MiB limit.
-->
設定ファイル中で、 `args` の部分で指定する引数はコンテナの起動時に使います。`"--vm-bytes", "250M"` の引数は、コンテナに対してメモリ 250 MiB の割り当てを試みていると伝えます。

<!--
Create the Pod:
-->
ポッドを作成します：

```shell
kubectl create -f https://k8s.io/docs/tasks/configure-pod-container/memory-request-limit-2.yaml --namespace=mem-example
```

<!--
View detailed information about the Pod:
-->
ポッドに対する詳細情報を表示します：

```shell
kubectl get pod memory-demo-2 --namespace=mem-example
```

<!--
At this point, the Container might be running, or it might have been killed. If the
Container has not yet been killed, repeat the preceding command until you see that
the Container has been killed:
-->
この時点で、コンテナは実行しているかもしれませんが、停止（killl）させられているでしょう：

```shell
NAME            READY     STATUS      RESTARTS   AGE
memory-demo-2   0/1       OOMKilled   1          24s
```

<!--
Get a more detailed view of the Container's status:
-->
コンテナの詳しい状態を表示します：

```shell
kubectl get pod memory-demo-2 --output=yaml --namespace=mem-example
```

<!--
The output shows that the Container has been killed because it is out of memory (OOM).
-->
出力結果からはメモリ不足（out of memory：OOM）のため、コンテナが停止済（killed）であるのが分かります。

```shell
lastState:
   terminated:
     containerID: docker://65183c1877aaec2e8427bc95609cc52677a454b56fcb24340dbd22917c23b10f
     exitCode: 137
     finishedAt: 2017-06-20T20:52:19Z
     reason: OOMKilled
     startedAt: null
```

<!--
The Container in this exercise is restartable, so the kubelet will restart it. Enter
this command several times to see that the Container gets repeatedly killed and restarted:
-->
この演習のコンテナは再起動可能なので、kubelet は再起動します。次のコマンドを数回実行し、コンテナが停止と再起動を繰り返すのを確認します。

```shell
kubectl get pod memory-demo-2 --namespace=mem-example
```

<!--
The output shows that the Container gets killed, restarted, killed again, restarted again, and so on:
-->
出力結果からはコンテナが停止、再起動、再び停止、再び再起動、の繰り返しが分かります：

```
stevepe@sperry-1:~/steveperry-53.github.io$ kubectl get pod memory-demo-2 --namespace=mem-example
NAME            READY     STATUS      RESTARTS   AGE
memory-demo-2   0/1       OOMKilled   1          37s
stevepe@sperry-1:~/steveperry-53.github.io$ kubectl get pod memory-demo-2 --namespace=mem-example
NAME            READY     STATUS    RESTARTS   AGE
memory-demo-2   1/1       Running   2          40s
```

<!--
View detailed information about the Pod's history:
-->
ポッドの履歴から詳細情報を表示します：

```
kubectl describe pod memory-demo-2 --namespace=mem-example
```

<!--
The output shows that the Container starts and fails repeatedly:
-->
出力結果からコンテナは起動と失敗を繰り返しているのが分かります。

```
... Normal  Created   Created container with id 66a3a20aa7980e61be4922780bf9d24d1a1d8b7395c09861225b0eba1b1f8511
... Warning BackOff   Back-off restarting failed container
```

<!--
View detailed information about your cluster's Nodes:
-->
クラスタのノードから詳細情報を表示します：

```
kubectl describe nodes
```

<!--
The output includes a record of the Container being killed because of an out-of-memory condition:
-->
出力結果にはコンテナがメモリ不足状態のため停止した記録が含まれています。

```
Warning OOMKilling  Memory cgroup out of memory: Kill process 4481 (stress) score 1994 or sacrifice child
```

<!--
Delete your Pod:
-->
ポッドの削除：

```shell
kubectl delete pod memory-demo-2 --namespace=mem-example
```

<!--
## Specify a memory request that is too big for your Nodes
-->
## ノードに対して大きすぎるメモリ要求を指定 {#specify-a-mmeory-request-that-is-too-big-for-your-nodes}

<!--
Memory requests and limits are associated with Containers, but it is useful to think
of a Pod as having a memory request and limit. The memory request for the Pod is the
sum of the memory requests for all the Containers in the Pod. Likewise, the memory
limit for the Pod is the sum of the limits of all the Containers in the Pod.
-->
メモリ要求と上限はコンテナに関連付けられたものですが、ポッドに対するメモリ要求と上限の検討も便利です。ポッドに対するメモり要求とは、ポッド内の全てのコンテナに対するメモリ要求の合計です。同様に、ポッドのメモリ上限とは、ポッド内の全てのコンテナに対するメモリ上限の合計です。

<!--
Pod scheduling is based on requests. A Pod is scheduled to run on a Node only if the Node
has enough available memory to satisfy the Pod's memory request.
-->
ポッドのスケジューリングはリクエストに基づきます。ポッドがノード上での実行をスケジュールするのは、ノードがポッドのメモリ要求を満たすために十分なメモリがある時です。

<!--
In this exercise, you create a Pod that has a memory request so big that it exceeds the
capacity of any Node in your cluster. Here is the configuration file for a Pod that has one
Container. The Container requests 1000 GiB of memory, which is likely to exceed the capacity
of any Node in your cluster.
-->
この演習では、クラスタ内にあるノードの受け入れ能力を超えるメモリ要求をするポッドを作成します。この設定ファイルは１つのコンテナを持つポッドを作成します。コンテナは 1000 GiB のメモリを要求します。これはクラスタ上のノードの受け入れ能力を超えるものです。


{{< code file="memory-request-limit-3.yaml" >}}

<!--
Create the Pod:
-->
ポッドを作成します：

```shell
kubectl create -f https://k8s.io/docs/tasks/configure-pod-container/memory-request-limit-3.yaml --namespace=mem-example
```

<!--
View the Pod's status:
-->
ポッドの状態を確認します：

```shell
kubectl get pod memory-demo-3 --namespace=mem-example
```

<!--
The output shows that the Pod's status is PENDING. That is, the Pod has not been
scheduled to run on any Node, and it will remain in the PENDING state indefinitely:
-->
出力結果からポッドの状態が「待機中」(Pending)になっているのが分かります。これはポッドを実行するためのノードが存在しておらず、ポッドのスケジュールができないからです。そのため、いつまでもずっと「待機中」のままです。

```
kubectl get pod memory-demo-3 --namespace=mem-example
NAME            READY     STATUS    RESTARTS   AGE
memory-demo-3   0/1       Pending   0          25s
```

<!--
View detailed information about the Pod, including events:
-->
イベントを含むポッドの詳細な情報を表示します：


```shell
kubectl describe pod memory-demo-3 --namespace=mem-example
```

<!--
The output shows that the Container cannot be scheduled because of insufficient memory on the Nodes:
-->
出力結果からコンテナは十分なメモリ容量を持つノードがないため、スケジュールできないのが分かります。


```shell
Events:
  ...  Reason            Message
       ------            -------
  ...  FailedScheduling  No nodes are available that match all of the following predicates:: Insufficient memory (3).
```

<!--
## Memory units
-->
## メモリ単位 {#memory-units}

<!--
The memory resource is measured in bytes. You can express memory as a plain integer or a
fixed-point integer with one of these suffixes: E, P, T, G, M, K, Ei, Pi, Ti, Gi, Mi, Ki.
For example, the following represent approximately the same value:
-->
メモリ・リソースはバイトで計算します。メモリは単純な整数や固定小数点と末尾の添え字（サフィックス） E、 P、 T、 G、 M、 K、 Ei、 Pi、 Ti、 Gi、 Mi、 Ki で表現します。

```shell
128974848, 129e6, 129M , 123Mi
```

<!--
Delete your Pod:
-->
ポッドを削除します：

```shell
kubectl delete pod memory-demo-3 --namespace=mem-example
```

<!--
## If you don’t specify a memory limit
-->
##  メモリ上限を指定しなければ {#if-you-dont-specify-a-memory-limit}

<!--
If you don’t specify a memory limit for a Container, then one of these situations applies:
-->
コンテナに対するメモリ上限を指定しなければ、以下のいずれかの状態が適用されます：

<!--
* The Container has no upper bound on the amount of memory it uses. The Container
could use all of the memory available on the Node where it is running.

* The Container is running in a namespace that has a default memory limit, and the
Container is automatically assigned the default limit. Cluster administrators can use a
[LimitRange](/docs/reference/generated/kubernetes-api/{{< param "version" >}}/#limitrange-v1-core)
to specify a default value for the memory limit.
-->
* コンテナは上限無くメモリ容量を使います。コンテナは起動したノード上で利用可能なメモリのすべてを使います。
* コンテナを名前空間内で実行している場合は、デフォルトのメモリ上限があるため、コンテナには自動的にデフォルトの上限値が自動的に割り当てられます。クラスタの管理者は [LimitRange](/docs/reference/generated/kubernetes-api/{{< param "version" >}}/#limitrange-v1-core) でメモリ上限のデフォルト値を指定できます。

<!--
## Motivation for memory requests and limits
-->
## メモリ要求と上限の動機 {#motivation-for-memory-requests-and-limits}

<!--
By configuring memory requests and limits for the Containers that run in your
cluster, you can make efficient use of the memory resources available on your cluster's
Nodes. By keeping a Pod's memory request low, you give the Pod a good chance of being
scheduled. By having a memory limit that is greater than the memory request, you accomplish two things:
-->
クラスタを動かすにあたり、クラスタのノード上でメモリ・リソース活用を効率的にするために、コンテナに対するメモリ要求と上限を設定します。ポッドのメモリ要求を低くし続けると、ポッドにスケジュールできる機会を与えます。メモリ要求を上回るメモリ上限を設けると、２つの点を達成します：

<!--
* The Pod can have bursts of activity where it makes use of memory that happens to be available.
* The amount of memory a Pod can use during a burst is limited to some reasonable amount.
-->
* ポッドはメモリを利用できる範囲内でバースト（訳者注：突発的なリソース利用が）できる。
* ポッドがバーストの間中に大量のメモリを 使ったとしても、適切な利用量に制限できる。

<!--
## Clean up
-->
## 後片付け {#clean-up}

<!--
Delete your namespace. This deletes all the Pods that you created for this task:
-->
名前空間を削除します。このタスクで作成したすべてのポッドを削除します：

```shell
kubectl delete namespace mem-example
```

{{% /capture %}}

{{% capture whatsnext %}}

<!--
### For app developers
-->
### アプリケーション開発者向け {for-app-developers}

<!--
* [Assign CPU Resources to Containers and Pods](/docs/tasks/configure-pod-container/assign-cpu-resource/)

* [Configure Quality of Service for Pods](/docs/tasks/configure-pod-container/quality-service-pod/)
-->
* [コンテナとポッドに CPU リソースを割り当て](/jp/docs/tasks/configure-pod-container/assign-cpu-resource/)
* [ポッドにサービス品質を設定](/jp/docs/tasks/configure-pod-container/quality-service-pod/)


<!--
### For cluster administrators
-->
### クラスタ管理者向け {#for-cluster-administrator}

<!--
* [Configure Default Memory Requests and Limits for a Namespace](/docs/tasks/administer-cluster/memory-default-namespace/)

* [Configure Default CPU Requests and Limits for a Namespace](/docs/tasks/administer-cluster/cpu-default-namespace/)

* [Configure Minimum and Maximum Memory Constraints for a Namespace](/docs/tasks/administer-cluster/memory-constraint-namespace/)

* [Configure Minimum and Maximum CPU Constraints for a Namespace](/docs/tasks/administer-cluster/cpu-constraint-namespace/)

* [Configure Memory and CPU Quotas for a Namespace](/docs/tasks/administer-cluster/quota-memory-cpu-namespace/)

* [Configure a Pod Quota for a Namespace](/docs/tasks/administer-cluster/quota-pod-namespace/)

* [Configure Quotas for API Objects](/docs/tasks/administer-cluster/quota-api-object/)
-->
* [名前空間にデフォルトのメモリ要求と上限を設定](/jp/docs/tasks/administer-cluster/memory-default-namespace/)
* [名前空間にデフォルトの CPU 要求と上限を設定](/jp/docs/tasks/administer-cluster/cpu-default-namespace/)
* [名前空間に最小と最大のメモリ制限を設定](/jp/docs/tasks/administer-cluster/memory-constraint-namespace/)
* [名前空間に最小と最大のメモリ制限を設定](/jp/docs/tasks/administer-cluster/cpu-constraint-namespace/)
* [名前空間にメモリと CPU の割り当て制限を設定](/jp/docs/tasks/administer-cluster/quota-memory-cpu-namespace/)
* [名前空間にポッドの割り当て制限を設定](/jp/docs/tasks/administer-cluster/quota-pod-namespace/)
* [API オブジェクトに対する割り当て制限を設定](/jp/docs/tasks/administer-cluster/quota-api-object/)

{{% /capture %}}



