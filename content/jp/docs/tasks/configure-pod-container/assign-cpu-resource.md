---
title: コンテナごとに CPU リソースを割り当て
content_template: templates/task
weight: 20
---

{{% capture overview %}}
<!--
This page shows how to assign a CPU *request* and a CPU *limit* to
a Container. A Container is guaranteed to have as much CPU as it requests,
but is not allowed to use more CPU than its limit.
-->
このページはコンテナに対して CPU 要求（ *request* ）と CPU 上限（ *limit* ）を割り当てる方法を説明します。コンテナは要求があった CPU 保持を保証します。しかし、上限を超える CPU 使用を許可しません。


{{% /capture %}}


{{% capture prerequisites %}}

{{< include "task-tutorial-prereqs.md" >}} {{< version-check >}}

<!--
Each node in your cluster must have at least 1 cpu.
-->
クラスタの各ノードでは少なくとも１つの CPU が必要です。

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
kubectl create namespace cpu-example
```

<!--
## Specify a CPU request and a CPU limit
-->
## メモリ要求とメモリ上限の指定 {#spacify-a-cpu-request-and-a-cpu-limit}

<!--
To specify a CPU request for a Container, include the `resources:requests` field
in the Container's resource manifest. To specify a CPU limit, include `resources:limits`.
-->
コンテナに対して CPU 要求（request）を指定するには、コンテナのリソース・マニフェストを `resources:request` に含めます。CPU 上限（limit）を指定するには `resources:imits` に含めます。


<!--
In this exercise, you create a Pod that has one Container. The Container has a CPU
request of 0.5 cpu and a CPU limit of 1 cpu. Here's the configuration file
for the Pod:
-->
この演習では１つのコンテナがあるポッドを作成します。コンテナは CPU 要求が 0.5 cpu、CPU 上限が 1 cpu です。こちらはポッド用の設定ファイルです。

{{< code file="cpu-request-limit.yaml" >}}

<!--
In the configuration file, the `args` section provides arguments for the Container when it starts.
The `-cpus "2"` argument tells the Container to attempt to use 2 cpus.
-->
設定ファイル中で、 `args` の部分で指定する引数はコンテナの起動時に使います。 `-cpus "2"` の引数は、コンテナに対して 2 cpu の割り当てを試みていると伝えます。


<!--
Create the Pod:
-->
ポッドを作成します：


```shell
kubectl create -f https://k8s.io/docs/tasks/configure-pod-container/cpu-request-limit.yaml --namespace=cpu-example
```

<!--
Verify that the Pod's Container is running:
-->
ポッドのコンテナが稼働中なのを確認します：

```shell
kubectl get pod cpu-demo --namespace=cpu-example
```

<!--
View detailed information about the Pod:
-->
ポッドに関する詳細情報を表示します。

```shell
kubectl get pod cpu-demo --output=yaml --namespace=cpu-example
```

<!--
The output shows that the one Container in the Pod has a CPU request of 500 millicpu
and a CPU limit of 1 cpu.
-->
出力結果から、ポッド内のコンテナには 500 ミリcpu の CPU 要求と 1 cpu の CPU 上限があるのが分かります。


```shell
resources:
  limits:
    cpu: "1"
  requests:
    cpu: 500m
```

<!--
Use `kubectl top` to fetch the metrics for the pod:
-->
`kubectl top` を使い、ポッドに対する数値（メトリクス）を取得します：

```shell
kubectl top pod memory-demo
```

<!--
The output shows that the Pod is using 974 millicpu, which is just a bit less than
the limit of 1 cpu specified in the Pod's configuration file.
-->
出力結果からポッドは 974 ミリcpu を使用中と表示しています。これはポッド設定ファイル上で指定している 1 cpu の上限に収まっています。

```
NAME                        CPU(cores)   MEMORY(bytes)
memory-demo                 794m         <something>
```

<!--
Recall that by setting `-cpu "2"`, you configured the Container to attempt to use 2 cpus.
But the Container is only being allowed to use about 1 cpu. The Container's CPU use is being
throttled, because the Container is attempting to use more CPU resources than its limit.
-->
`-cpu "2"` を指定したのを思い出してください。コンテナに対して 2 cpu を使うように設定しました。しかし、コンテナが利用を許可されているのは 1 cpu のみです。コンテナの CPU 使用は絞り込まれています。なぜなら、コンテナは上限を超えて CPU リソースの利用を試みるからです。

{{< note >}}
<!--
**Note:** There's another possible explanation for the CPU throttling. The Node might not have
enough CPU resources available. Recall that the prerequisites for this exercise require that each of
your Nodes has at least 1 cpu. If your Container is running on a Node that has only 1 cpu, the Container
cannot use more than 1 cpu regardless of the CPU limit specified for the Container.
-->
**メモ：** CPU 絞り込みに関しては他の解釈の場合もあります。ノードは十分利用可能な CPU リソースがあるかもしれません。この演習のために準備した環境を思い出すと、各ノードは最小 1 cpu を指定しました。コンテナを実行するノードが 1 cpu のみであれば、コンテナに対して指定した CPU 上限に関係なく、１ cpu を越えて利用できません。
{{< /note >}}

<!--
## CPU units
-->
## CPU 単位 {#cpu-units}

<!--
The CPU resource is measured in *cpu* units. One cpu, in Kubernetes, is equivalent to:
-->
CPU リソースは *cpu* 単位で計測されます。１ CPU とは、Kubernetes では以下と同等です：

* 1 AWS vCPU
* 1 GCP Core
* 1 Azure vCore
* 1 ハイパースレッド（ハイパースレッディングを使うベアメタル・インテル・プロセッサ）

<!--
Fractional values are allowed. A Container that requests 0.5 cpu is guaranteed half as much
CPU as a Container that requests 1 cpu. You can use the suffix m to mean milli. For example
100m cpu, 100 millicpu, and 0.1 cpu are all the same. Precision finer than 1m is not allowed.
-->
小数値も指定できます。0.5 cpu を要求するコンテナには、1 cpu を要求するコンテナの半分を保証します。

<!--
CPU is always requested as an absolute quantity, never as a relative quantity; 0.1 is the same
amount of CPU on a single-core, dual-core, or 48-core machine.
-->
CPU の指定は常に絶対的な値であり、相対値ではありません。つまり、 0.1 はシングル・コア、デュアル・コア、あるいは 48 コアのマシンでも同じ値です。

<!--
Delete your Pod:
-->
ポッドを削除します：

```shell
kubectl delete pod cpu-demo --namespace=cpu-example
```

<!--
## Specify a CPU request that is too big for your Nodes
-->
## ノードに対して大きすぎる CPU 要求を指定 {#specify-a-cpu-request-that-is-too-big-for-your-nodes}

<!--
CPU requests and limits are associated with Containers, but it is useful to think
of a Pod as having a CPU request and limit. The CPU request for a Pod is the sum
of the CPU requests for all the Containers in the Pod. Likewise, the CPU limit for
a Pod is the sum of the CPU limits for all the Containers in the Pod.
-->
CPU 要求と上限はコンテナに関連付けられたものですが、CPU に対するメモリ要求と上限の検討も便利です。ポッドに対する CPU 要求とは、ポッド内の全てのコンテナに対する CPU 要求の合計です。同様に、ポッドの CPU 上限とは、ポッド内の全てのコンテナに対するCPU 上限の合計です。

<!--
Pod scheduling is based on requests. A Pod is scheduled to run on a Node only if
the Node has enough CPU resources available to satisfy the Pod’s CPU request.
-->
ポッドのスケジューリングはリクエストに基づきます。ポッドがノード上での実行をスケジュールするのは、ノードがポッドの CPU 要求を満たすために十分な CPU がある時です。

<!--
In this exercise, you create a Pod that has a CPU request so big that it exceeds
the capacity of any Node in your cluster. Here is the configuration file for a Pod
that has one Container. The Container requests 100 cpu, which is likely to exceed the
capacity of any Node in your cluster.
-->
この演習では、クラスタ内にあるノードの受け入れ能力を超える CPU 要求をするポッドを作成します。この設定ファイルは１つのコンテナを持つポッドを作成します。コンテナは 100 cpu を要求します。これはクラスタ上のノードの受け入れ能力を超えるものです。



{{< code file="cpu-request-limit-2.yaml" >}}

<!--
Create the Pod:
-->
ポッドを作成します：

```shell
kubectl create -f https://k8s.io/docs/tasks/configure-pod-container/cpu-request-limit-2.yaml --namespace=cpu-example
```

<!--
View the Pod's status:
-->
ポッドの状態を確認します：

```shell
kubectl get pod cpu-demo-2 --namespace=cpu-example
```

<!--
The output shows that the Pod's status is Pending. That is, the Pod has not been
scheduled to run on any Node, and it will remain in the Pending state indefinitely:
-->
出力結果からポッドの状態が待機中(Pending)になっているのが分かります。これはポッドを実行するためのノードが存在しておらず、ポッドのスケジュールができないからです。そのため、いつまでもずっと待機中のままです。


```
kubectl get pod cpu-demo-2 --namespace=cpu-example
NAME         READY     STATUS    RESTARTS   AGE
cpu-demo-2   0/1       Pending   0          7m
```

<!--
View detailed information about the Pod, including events:
-->
イベントを含むポッドの詳細な情報を表示します：

```shell
kubectl describe pod cpu-demo-2 --namespace=cpu-example
```

<!--
The output shows that the Container cannot be scheduled because of insufficient
CPU resources on the Nodes:
-->
出力結果からコンテナは十分な CPU 容量を持つノードがないため、スケジュールできないのが分かります。


```shell
Events:
  Reason			Message
  ------			-------
  FailedScheduling	No nodes are available that match all of the following predicates:: Insufficient cpu (3).
```

<!--
Delete your Pod:
-->
ポッドを削除します：

```shell
kubectl delete pod cpu-demo-2 --namespace=cpu-example
```

<!--
## If you don’t specify a CPU limit
-->
##  CPU 上限を指定しなければ {#if-you-dont-specify-a-cpu-limit}

<!--
If you don’t specify a CPU limit for a Container, then one of these situations applies:
-->
コンテナに対する CPU 上限を指定しなければ、以下のいずれかの状態が適用されます：

<!--
* The Container has no upper bound on the CPU resources it can use. The Container
could use all of the CPU resources available on the Node where it is running.

* The Container is running in a namespace that has a default CPU limit, and the
Container is automatically assigned the default limit. Cluster administrators can use a
[LimitRange](/docs/reference/generated/kubernetes-api/{{< param "version" >}}/#limitrange-v1-core/)
to specify a default value for the CPU limit.
-->
* コンテナは上限無く CPU リソースを使います。コンテナは起動したノード上で利用可能な CPU リソースのすべてを使います。
* コンテナを名前空間内で実行している場合は、デフォルトの CPU 上限があるため、コンテナには自動的にデフォルトの上限値が自動的に割り当てられます。クラスタの管理者は [LimitRange](/docs/reference/generated/kubernetes-api/{{< param "version" >}}/#limitrange-v1-core) で CPU 上限のデフォルト値を指定できます。

<!--
## Motivation for CPU requests and limits
-->
## CPU 要求と上限の動機 {#motivation-for-memory-requests-and-limits}

<!--
By configuring the CPU requests and limits of the Containers that run in your
cluster, you can make efficient use of the CPU resources available on your cluster's
Nodes. By keeping a Pod's CPU request low, you give the Pod a good chance of being
scheduled. By having a CPU limit that is greater than the CPU request, you accomplish two things:
-->
クラスタを動かすにあたり、クラスタのノード上でメモリ・リソース活用を効率的にするために、コンテナに対する CPU 要求と上限を設定します。ポッドの CPU 要求を低くし続けると、ポッドにスケジュールできる機会を与えます。CPU 要求を上回るメモリ上限を設けると、２つの点を達成します：


<!--
* The Pod can have bursts of activity where it makes use of CPU resources that happen to be available.
* The amount of CPU resources a Pod can use during a burst is limited to some reasonable amount.
-->
* ポッドは CPU リソースを利用できる範囲内でバースト（訳者注：突発的なリソース利用が）できる。
* ポッドがバーストの間中に大量の CPU リソースを 使ったとしても、適切な利用量に制限できる。

<!--
## Clean up
-->
## 後片付け {#clean-up}

<!--
Delete your namespace:
-->
名前空間を削除します：

```shell
kubectl delete namespace cpu-example
```

{{% /capture %}}

{{% capture whatsnext %}}

<!--
### For app developers
-->
### アプリケーション開発者向け {for-app-developers}

<!--
* [Assign Memory Resources to Containers and Pods](/docs/tasks/configure-pod-container/assign-memory-resource/)

* [Configure Quality of Service for Pods](/docs/tasks/configure-pod-container/quality-service-pod/)
-->
* [コンテナとポッドにメモリリソースを割り当て](/jp/docs/tasks/configure-pod-container/assign-memory-resource/)
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



