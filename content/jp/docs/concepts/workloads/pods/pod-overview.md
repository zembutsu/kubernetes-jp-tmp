---
reviewers:
- erictune
title: ポッド概要
content_template: templates/concept
weight: 10
---

{{% capture overview %}}
<!--
This page provides an overview of `Pod`, the smallest deployable object in the Kubernetes object model.
-->
このページは `Pod` （ポッド）の概要を紹介します。ポッドとは Kubernetes オブジェクト・モデルにおいて最小の展開可能なオブジェクトです。

{{% /capture %}}

{{< toc >}}

{{% capture body %}}

<!--
## Understanding Pods
-->
## ポッドの理解 {#understanding-pods}

<!--
A *Pod* is the basic building block of Kubernetes--the smallest and simplest unit in the Kubernetes object model that you create or deploy. A Pod represents a running process on your cluster.
-->
*ポッド（Pod）* は Kubernetes における基本的な構築要素です。Kubernetes オブジェクト・モデルで作成・展開できる、最小かつ最もシンプルな単位（ユニット）です。ポッドはクラスタ上で実行中のプロセスに相当します。

<!--
A Pod encapsulates an application container (or, in some cases, multiple containers), storage resources, a unique network IP, and options that govern how the container(s) should run. A Pod represents a unit of deployment: *a single instance of an application in Kubernetes*, which might consist of either a single container or a small number of containers that are tightly coupled and that share resources.
-->
ポッドが収容するのは、アプリケーション・コンテナ（または、時として複数のコンテナ）、ストレージ・リソース、ユニークなネットワーク IP 、そして、どのようにコンテナを実行すべきかを管理するオプションです。ポッドは展開（デプロイ）の単位を表します。すなわち、 *Kubernetes におけるアプリケーション１つのインスタンス（実体）* です。ポッドに含まれるのは、１つのコンテナに限りません。密結合の小数コンテナがリソースを共有する場合もあります。

<!--
> [Docker](https://www.docker.com) is the most common container runtime used in a Kubernetes Pod, but Pods support other container runtimes as well.
-->
[Docker](https://www.docker.com) は Kubernetes ポッドとして最も一般的に使われるコンテナ・ランタイムです。しかし、ポッドは他のコンテナ・ランタイムも同様にサポートします。

<!--
Pods in a Kubernetes cluster can be used in two main ways:
-->
Kubernetes クラスタにおいて、ポッドは主に２つの方法で使われます：

<!--
* **Pods that run a single container**. The "one-container-per-Pod" model is the most common Kubernetes use case; in this case, you can think of a Pod as a wrapper around a single container, and Kubernetes manages the Pods rather than the containers directly.
* **Pods that run multiple containers that need to work together**. A Pod might encapsulate an application composed of multiple co-located containers that are tightly coupled and need to share resources. These co-located containers might form a single cohesive unit of service--one container serving files from a shared volume to the public, while a separate "sidecar" container refreshes or updates those files. The Pod wraps these containers and storage resources together as a single manageable entity.
-->
* **１つのコンテナを実行するポッド** 。 "ポッドごとに１つのコンテナ" モデルは、Kubernetes で最も共通する使い方です。この場合、１つのコンテナを囲むラッパーとして、ポッドを考えられます。そして、Kubernetes はコンテナの管理を直接ではなく、ポッドを通して行います。
* **同時に動かす必要のある複数のコンテナを実行するポッド** 。ポッドは、同一の場所にある複数のコンテナによって構成されるアプリケーションをカプセル化している場合があります。ポッド内のコンテナは密結合であり、リソースを共有する必要があります。これら同一の場所にあるコンテナは、１つの密接なサービス単位です。たとえば、あるコンテナが提供する共有ボリューム上のファイルを、別の「サイドカー」（sidecar）コンテナが各ファイルを再読み込みまたは更新して公開します。ポッドはこれらのコンテナとストレージ・リソースを１つの管理可能な存在としてラップします（包みます）。

<!--
The [Kubernetes Blog](http://blog.kubernetes.io) has some additional information on Pod use cases. For more information, see:
-->
[Kubernetes Blog](http://blog.kubernetes.io) にPod 利用例の追加情報があります。詳しい情報はこちらをご覧ください：

<!--
* [The Distributed System Toolkit: Patterns for Composite Containers](https://kubernetes.io/blog/2015/06/the-distributed-system-toolkit-patterns)
* [Container Design Patterns](https://kubernetes.io/blog/2016/06/container-design-patterns)
-->
* [分散システム・ツールキット：複数のコンテナ用パターン](https://kubernetes.io/blog/2015/06/the-distributed-system-toolkit-patterns)
* [コンテナ・デザイン・パターン](https://kubernetes.io/blog/2016/06/container-design-patterns)

<!---
Each Pod is meant to run a single instance of a given application. If you want to scale your application horizontally (e.g., run multiple instances), you should use multiple Pods, one for each instance. In Kubernetes, this is generally referred to as _replication_. Replicated Pods are usually created and managed as a group by an abstraction called a Controller. See [Pods and Controllers](#pods-and-controllers) for more information.
-->
各ポッドが意図しているのは、あるアプリケーション実体（インスタンス）１つの実行です。もしもアプリケーションを水平にスケールしたい場合は（例：複数のインスタンスを実行したい場合）、インスタンスを１つ１つ使うのではなく、ポッドを使うべきです。Kubernetes では、通常これを *レプリケーション（replication）* としてみなします。複製されたポッドの作成と管理は、コントローラ（controller）と呼ばれる抽象化されたグループによって行われます。詳しい情報は [ポッドとコントローラ](#pods-and-controllers) をご覧ください。

<!--
### How Pods manage multiple Containers
-->
### ポッドは複数のコンテナをどのように管理するか {#how-pods-manage-multiple-containers}

<!--
Pods are designed to support multiple cooperating processes (as containers) that form a cohesive unit of service. The containers in a Pod are automatically co-located and co-scheduled on the same physical or virtual machine in the cluster. The containers can share resources and dependencies, communicate with one another, and coordinate when and how they are terminated.
-->
ポッドは、密接なサービス単位からなる複数のプロセス（をコンテナとして）の協調をサポートするように設計されています。ポッド内のコンテナは、クラスタ内の同じ物理または仮想マシン上へ自動的に配置（co-located）され、また、同時にスケジュール（co-scheduled）されます。コンテナはリソースと依存関係を共有でき、他のコンテナとも通信可能であり、いつどのようにコンテナが停止したとしても調整を行います。

<!--
Note that grouping multiple co-located and co-managed containers in a single Pod is a relatively advanced use case. You should use this pattern only in specific instances in which your containers are tightly coupled. For example, you might have a container that acts as a web server for files in a shared volume, and a separate "sidecar" container that updates those files from a remote source, as in the following diagram:
-->
１つのポッド内で、コンテナを同じ場所で同時にグループ化して管理する場合は、比較的に難しい使い方になりますので注意してください。このパターンが使えるのは、コンテナ間が密結合である特別なパターンの時のみでしょう。たとえば、共有ボリュームにあるファイルをウェブ・サーバとして処理するコンテナを持っているとします。また、別の "sidecar"（サイドカー）コンテナが、次の図のように、リモート上のソースからファイルを更新するとします。

{{< figure src="/images/docs/pod.svg" title="ポッド図" width="50%" >}}

<!--
Pods provide two kinds of shared resources for their constituent containers: *networking* and *storage*.
-->
ポッドが提供するのはコンテナの構成要素となる2種類の共有リソース *ネットワーキング* と *ストレージ* です。

<!--
#### Networking
-->
#### ネットワーキング {#networking}

<!--
Each Pod is assigned a unique IP address. Every container in a Pod shares the network namespace, including the IP address and network ports. Containers *inside a Pod* can communicate with one another using `localhost`. When containers in a Pod communicate with entities *outside the Pod*, they must coordinate how they use the shared network resources (such as ports).
-->
各ポッドにはユニークな IP アドレスが割り当てられます。ポッド内の各コンテナはネットワーク名前空間を共有します。これには IP アドレスとネットワーク・ポートも含みます。 *ポッド内部の* コンテナは、 `localhost` を使って他と通信できます。ポッド内のコンテナが *ポッド外部の* 存在と通信する場合は、ネットワーク・リソース（ポートなど）を共有するために使う手法を調整する必要があります。

<!--
#### Storage
-->
#### ストレージ {#storage}

<!--
A Pod can specify a set of shared storage *volumes*. All containers in the Pod can access the shared volumes, allowing those containers to share data. Volumes also allow persistent data in a Pod to survive in case one of the containers within needs to be restarted. See [Volumes](/docs/concepts/storage/volumes/) for more information on how Kubernetes implements shared storage in a Pod.
-->
ポッドは共有ストレージを *ボリューム（volume）* として指定できます。ポッド内の全てのコンテナが共有ボリュームに接続できます。これにより、各コンテナはデータを共有できます。また、再起動が必用なコンテナがあったとしても、ポッド内のボリュームに持続データを保管できます。ポッド内で Kubernetes がストレージをどのようにして共有するか、この詳しい情報は [ボリューム](/jp/docs/concepts/storage/volumes/) をご覧ください。

<!--
## Working with Pods
-->
## ポッドと連携するには {#working-with-pods}

<!--
You'll rarely create individual Pods directly in Kubernetes--even singleton Pods. This is because Pods are designed as relatively ephemeral, disposable entities. When a Pod gets created (directly by you, or indirectly by a Controller), it is scheduled to run on a Node in your cluster. The Pod remains on that Node until the process is terminated, the pod object is deleted, the pod is *evicted* for lack of resources, or the Node fails.
-->
Kubernetes では個々のポッドを直接作成するのは滅多にありません。たとえ１つのポッドだとしてもです。これはポッドが比較的に一過性（ephemeral）で使い捨て可能ものとして設計されているためです。ポッドが作成されると（直接、あるいはコントローラによって間接的に）、クラスタ上のノードでの実行がスケジュールされます。ポッドはノードでプロセスが停止（terminate）されるまで残り続けます。ポッド・オブジェクトが削除されると、ポッドはリソースが欠如により *撤去（evicted）* されます。そうでない場合は、ノードが処理の失敗（fail）となります。

{{< note >}}
<!--
**Note:** Restarting a container in a Pod should not be confused with restarting the Pod. The Pod itself does not run, but is an environment the containers run in and persists until it is deleted.
-->
**メモ：** ポッド内のコンテナ再起動は、ポッドの再起動と混同すべきではありません。ポッド自身は実行しませんが、コンテナを実行するための環境とその維持が、削除されるまで続きます。
{{< /note >}}

<!--
Pods do not, by themselves, self-heal. If a Pod is scheduled to a Node that fails, or if the scheduling operation itself fails, the Pod is deleted; likewise, a Pod won't survive an eviction due to a lack of resources or Node maintenance. Kubernetes uses a higher-level abstraction, called a *Controller*, that handles the work of managing the relatively disposable Pod instances. Thus, while it is possible to use Pod directly, it's far more common in Kubernetes to manage your pods using a Controller. See [Pods and Controllers](#pods-and-controllers) for more information on how Kubernetes uses Controllers to implement Pod scaling and healing.
-->
ポッドが行わないのは、ポッド自身の自己修復（self-heal）です。ノードに対するポッドのスケジュールが失敗するか、あるいは、スケジュール作業そのものが失敗すると、ポッドは削除されます。さらに、ポッドはリソース不足による退去される場合、ノードのメンテナンスの場合も、そのまま存続できません。Kubernetes は *コントローラ（Controller）* と呼ぶ、高いレベルでの抽象化を使います。コントローラは比較的使い捨てのポッド・インスタンス（そのもの／実体）を管理する作業を扱います。


<!--
### Pods and Controllers
-->
### ポッドとコントローラ {#pods-and-controllers}

<!--
A Controller can create and manage multiple Pods for you, handling replication and rollout and providing self-healing capabilities at cluster scope. For example, if a Node fails, the Controller might automatically replace the Pod by scheduling an identical replacement on a different Node.
-->
コントローラは複数のポッドの作成と管理ができます。コントローラはクラスタの範囲内において、レプリケーション（複製）とロールアウト（展開）の処理と、自己修復能力を提供します。たとえば、ノードで障害が起これば、コントローラは自動的にポッドを置き換えるため、直ちに別のノード上への配置転換をスケジューリング（計画）します。

<!--
Some examples of Controllers that contain one or more pods include:
-->
以下は１つまたは複数のポッドを含むコントローラの例です：

<!--
* [Deployment](/docs/concepts/workloads/controllers/deployment/)
* [StatefulSet](/docs/concepts/workloads/controllers/statefulset/)
* [DaemonSet](/docs/concepts/workloads/controllers/daemonset/)
-->
* [Deployment（デプロイメント）](/jp/docs/concepts/workloads/controllers/deployment/)
* [StatefulSet（ステートフルセット）](/jp/docs/concepts/workloads/controllers/statefulset/)
* [DaemonSet（デーモンセット）](/jp/docs/concepts/workloads/controllers/daemonset/)

<!--
In general, Controllers use a Pod Template that you provide to create the Pods for which it is responsible.
-->
通常、コントローラはポッド・テンプレートを使い、コントローラが責任を負うポッドを作成します。
<!--
## Pod Templates
-->
## ポッド・テンプレート（Pod Templates） {#pod-templates}

<!--
Pod templates are pod specifications which are included in other objects, such as
[Replication Controllers](/docs/concepts/workloads/controllers/replicationcontroller/), [Jobs](/docs/concepts/jobs/run-to-completion-finite-workloads/), and
[DaemonSets](/docs/concepts/workloads/controllers/daemonset/).  Controllers use Pod Templates to make actual pods.
The sample below is a simple manifest for a Pod which contains a container that prints
a message.
-->
ポッド・テンプレートとはポッドの仕様であり、
[Replication Controllers（レプリケーション・コントローラ）](/docs/concepts/workloads/controllers/replicationcontroller/)、 [ジョブ](/docs/concepts/jobs/run-to-completion-finite-workloads/)、 [DaemonSets（デーモンセット）](/docs/concepts/workloads/controllers/daemonset/) のような他のオブジェクトを含みます。コントローラは実際のポッドを作成するために、ポッド・テンプレートを使います。以下の例はポッド向けの簡単なマニフェストです。ポッドに含まれるコンテナはメッセージを表示します。


```yaml
apiVersion: v1
kind: Pod
metadata:
  name: myapp-pod
  labels:
    app: myapp
spec:
  containers:
  - name: myapp-container
    image: busybox
    command: ['sh', '-c', 'echo Hello Kubernetes! && sleep 3600']
```

<!--
Rather than specifying the current desired state of all replicas, pod templates are like cookie cutters. Once a cookie has been cut, the cookie has no relationship to the cutter. There is no "quantum entanglement". Subsequent changes to the template or even switching to a new template has no direct effect on the pods already created. Similarly, pods created by a replication controller may subsequently be updated directly. This is in deliberate contrast to pods, which do specify the current desired state of all containers belonging to the pod. This approach radically simplifies system semantics and increases the flexibility of the primitive.
-->
全ての複製（レプリカ）に対して期待状態を指定するのに比べれば、ポッド・テンプレートはクッキーの抜き型（cookie cutter）のようなものです。クッキーをカッターを使って切り離したあとは、クッキーとカッターは何の関係も持ちません。ここには「量子もつれ（quantum entanglement）」はありません。テンプレートの変更後や、新しいテンプレートに切り替えた後も、既に作成したポッドに対しては何ら影響を与えません。一方で、レプリケーション・コントローラによって作成されたポッドであれば、その後も直接更新できます。これがポッドと比べて考慮が必要な点です。ポッドでであれば現時点における全コンテナの期待状態を指定するからです。この手法はシステムの形式化（セマンティクス）を極めて簡単にし、初期状態（プリミティブ）の柔軟性も高めます。

{{% /capture %}}

{{% capture whatsnext %}}
<!--
* Learn more about Pod behavior:
  * [Pod Termination](/docs/concepts/workloads/pods/pod/#termination-of-pods)
  * Other Pod Topics
-->
* ポッドの挙動について学ぶ：
  * [ポッドの停止](/jp/docs/concepts/workloads/pods/pod/#termination-of-pods)
  * 他のポッドの話題

{{% /capture %}}


