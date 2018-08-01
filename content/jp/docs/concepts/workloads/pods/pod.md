---
reviewers:
title: ポッド
content_template: templates/concept
weight: 20
---

{{% capture overview %}}

<!--
_Pods_ are the smallest deployable units of computing that can be created and
managed in Kubernetes.
-->

_ポッド（Pod）_ とは、Kubernetes が作成・管理できる計算用の最小デプロイ単位です。

{{% /capture %}}

{{< toc >}}

{{% capture body %}}

<!--
## What is a Pod?
-->
## ポットとは何でしょうか？ {#what-is-a-pod}

<!--
A _pod_ (as in a pod of whales or pea pod) is a group of one or more containers
(such as Docker containers), with shared storage/network, and a specification
for how to run the containers.  A pod's contents are always co-located and
co-scheduled, and run in a shared context.  A pod models an
application-specific "logical host" - it contains one or more application
containers which are relatively tightly coupled &mdash; in a pre-container
world, they would have executed on the same physical or virtual machine.
-->
_ポッド（Pod）_ （意味は鯨の群れ、あるいは、豆の鞘）とは、１つまたは複数のコンテナ（Docker コンテナなど）のグループであり、ストレージとネットワークを共有し、コンテナをどのように実行するかの仕様を持ちます。ポッドに含まれるものは、共有のコンテクスト（訳者注：プロセス実行に必要な最小のデータ・セット）において、常に一緒の場所で、同時にスケジュール（実行を予定）し、実行します。ポッドのモデル（雛形）となるのは、アプリケーション固有の「論理的なホスト」であり、ここには１つまたは比較的に関連性の強いそれ以上のアプリケーション・コンテナが入っています。このアプリケーションは、コンテナが登場する前の世界では、同じ物理もしくは仮想マシン上で実行されるべきものでした。

<!--
While Kubernetes supports more container runtimes than just Docker, Docker is
the most commonly known runtime, and it helps to describe pods in Docker terms.
-->
kubernetes は Docker のような複数のコンテナ・ランタイムをサポートします。Docker は最も一般的に知られているランタイムであり、Docker 用語でポッドを説明するのに役立ちます。

<!--
The shared context of a pod is a set of Linux namespaces, cgroups, and
potentially other facets of isolation - the same things that isolate a Docker
container.  Within a pod's context, the individual applications may have
further sub-isolations applied.
-->
ポッドの共有コンテクストとは、Linux 名前区間（namespace）、コントロール・グループ（cgroup）、その他の独立（isolation）に関する面（facet：ファセット）の集まりであり、これは Docker コンテナにおける独立（isolate）と同じものです。ポッドのコンテクストでは、個々のアプリケーションが更に下位の独立（sub-isolations）をしている場合があります。

<!--
Containers within a pod share an IP address and port space, and
can find each other via `localhost`. They can also communicate with each
other using standard inter-process communications like SystemV semaphores or
POSIX shared memory.  Containers in different pods have distinct IP addresses
and can not communicate by IPC without
[special configuration](/docs/concepts/policy/pod-security-policy/).
These containers usually communicate with each other via Pod IP addresses.
-->
ポッド内のコンテナは IP アドレスをとポート範囲を共有し、お互いを `localhost` を経由して発見できます。また、SystemV セマフォや POSIX 共有メモリのように、標準プロセス間通信（IPC）を使っても相互に通信できます。違うポッドにあるコンテナは相異なる IP ドレスを持つため、 IPC では [特別な設定](/docs/concepts/policy/pod-security-policy/) がなければ通信できません。コンテナの通信は、常にポッドの IP アドレスを経由して行います。

<!--
Applications within a pod also have access to shared volumes, which are defined
as part of a pod and are made available to be mounted into each application's
filesystem.
-->
また、ポッド内のアプリケーションは共有ボリュームに接続できます。共有ボリュームはポッドの一部として定義されるもので、各アプリケーションのファイルシステムにマウントして利用できます。

<!--
In terms of [Docker](https://www.docker.com/) constructs, a pod is modelled as
a group of Docker containers with shared namespaces and shared
[volumes](/docs/concepts/storage/volumes/).
-->
[Docker](https://www.docker.com/) の概念における言葉を使うと、名前空間を共有する Docker コンテナと共有 [ボリューム](/jp/docs/concepts/storage/volumes/) のグループとしてモデル化したもの（雛形にしたもの）がポッドです。

<!--
Like individual application containers, pods are considered to be relatively
ephemeral (rather than durable) entities. As discussed in [life of a
pod](/docs/concepts/workloads/pods/pod-lifecycle/), pods are created, assigned a unique ID (UID), and
scheduled to nodes where they remain until termination (according to restart
policy) or deletion. If a node dies, the pods scheduled to that node are
scheduled for deletion, after a timeout period. A given pod (as defined by a UID) is not
"rescheduled" to a new node; instead, it can be replaced by an identical pod,
with even the same name if desired, but with a new UID (see [replication
controller](/docs/concepts/workloads/controllers/replicationcontroller/) for more details).
-->
個々のアプリケーション・コンテナのように、ポッドとは（常に使うものと比べて）比較的に短期間の存在（ephemeral entity）と考えられます。[ポッドの一生（life of a pod）](/jp/docs/concepts/workloads/pods/pod-lifecycle/) で論じているように、ポッドを作成すると、ユニークな ID （UID）が割り当てられ、終了もしくは削除されるまで、どこかのノード上で実行しつづけるようにスケジュール（計画）されます。ノードが停止すると、タイムアウト時間（一定の停止時間）を経過後、そのノードに対するスケジュールは削除されます。対象のポッド（UID によって定義されます）が新しいノードに「再スケジュール（rescheduled）」されるのではありません。そのかわりに、全く同じポッドに置き換えられます。必要があれば同じ名前にできますが、新しい UID を割り当てられます（詳細は [replication controller（レプリケーション・コントローラ）](/jpdocs/concepts/workloads/controllers/replicationcontroller/) をご覧ください。）。

<!--
When something is said to have the same lifetime as a pod, such as a volume,
that means that it exists as long as that pod (with that UID) exists. If that
pod is deleted for any reason, even if an identical replacement is created, the
related thing (e.g. volume) is also destroyed and created anew.
-->
もし誰かがボリュームなどがポッドと同じライフタイムを持つと言及する場合は、ポッドが（その UID で）存在する限り（ボリュームなどが）存在するという意味です。ポッドが何らかの理由によって削除された場合、たとえ全く同じものに置き換えるための再作成だとしても、関連したもの（例：ボリューム）も同時に削除され、新しいものが再び作られます。

{{< figure src="/images/docs/pod.svg" title="pod diagram" width="50%" >}}

<!--
*A multi-container pod that contains a file puller and a
web server that uses a persistent volume for shared storage between the containers.*
-->
*file puller（ファイル取得）とウェブサーバと、この複数のコンテナがあるポッドは、コンテナ間で共有するストレージとして、持続ボリューム（persistent volume）を使います。*

<!--
## Motivation for pods
-->
## ポッドの目的（モチベーション） {#motivation-for-pods}

<!--
### Management
-->
### 管理 {#management}

<!--
Pods are a model of the pattern of multiple cooperating processes which form a
cohesive unit of service.  They simplify application deployment and management
by providing a higher-level abstraction than the set of their constituent
applications. Pods serve as unit of deployment, horizontal scaling, and
replication. Colocation (co-scheduling), shared fate (e.g. termination),
coordinated replication, resource sharing, and dependency management are
handled automatically for containers in a pod.
-->
ポッドとは、密接なサービス単位が提供する、複数の協働（協調しながら動作する）プロセスのパターンをモデル化したものです。アプリケーションの構成要素の集まりと比較し、高いレベルの抽象化によってアプリケーションの展開と管理を簡単にします。ポッドが提供するのは展開（デプロイメント）、水平スケール、複製（レプリケーション）の単位（ユニット）です。同じ場所にあり（同時にスケジュールし）、命運を共にし（つまり、削除です）、複製を調整し、リソースを共有し、ポッド内におけるコンテナの依存性を自動的に管理します。

<!--
### Resource sharing and communication
-->
### リソース共有と通信 {#resource-sharing-and-communication}

<!--
Pods enable data sharing and communication among their constituents.
-->
ポッドは構成要素間でのデータの共有と通信を可能にします。

<!--
The applications in a pod all use the same network namespace (same IP and port
space), and can thus "find" each other and communicate using `localhost`.
Because of this, applications in a pod must coordinate their usage of ports.
Each pod has an IP address in a flat shared networking space that has full
communication with other physical computers and pods across the network.
-->
ポッド内のアプリケーションは、共通のネットワーク名前空間（同じ IP アドレスとポート範囲）を使います。そして、さらに通信するためには `localhost` を使って相互に「発見」できます。このため、ポッド内のアプリケーションは使用するポートの調整が必須です。各ポッドは IP アドレスを平らに共有するネットワーク範囲を持っているため、ポッドは他の物理コンピュータやネットワークを横断するポッドと完全に通信できます。

<!--
The hostname is set to the pod's Name for the application containers within the
pod. [More details on networking](/docs/concepts/cluster-administration/networking/).
-->
ポッド内のアプリケーション・コンテナのホスト名には、ポッドの名前が設定されます。詳細は [ネットワーク機能](/jp/docs/concepts/cluster-administration/networking/) をご覧ください。

<!--
In addition to defining the application containers that run in the pod, the pod
specifies a set of shared storage volumes. Volumes enable data to survive
container restarts and to be shared among the applications within the pod.
-->
アプリケーション・コンテナが定義するのは、ポッド内で何を実行するかに加え、ポッドの共有ストレージ・ボリュームのセットも定義します。ボリュームはコンテナを再起動してもデータを残し続けるのが可能であり、ポッド内のアプリケーション間でデータを共有できます。

<!--
## Uses of pods
-->
## ポッドの使い方 {#uses-of-pods}

<!--
Pods can be used to host vertically integrated application stacks (e.g. LAMP),
but their primary motivation is to support co-located, co-managed helper
programs, such as:
-->
（LAMP のような）アプリケーション・スタック（積み重ね）としてホストを水平統合するためにポッドを使えます。しかし、ポッドの主な目的は、次のような、同じ場所にあり、同時に管理するのに役立つプログラムをサポートするためです：

<!--
* content management systems, file and data loaders, local cache managers, etc.
* log and checkpoint backup, compression, rotation, snapshotting, etc.
* data change watchers, log tailers, logging and monitoring adapters, event publishers, etc.
* proxies, bridges, and adapters
* controllers, managers, configurators, and updaters
-->
* コンテンツの管理システム、ファイル、データ読み込み、ローカルキャッシュ管理など。
* ログとチェックポイントのバックアップ、圧縮、ローテーション、スナップショットなど。
* データ変更監視、ログの追跡、ログ機能と監視のためのアダプタ、イベント送信（パブリッシャー）など
* プロキシ、ブリッジ、アダプタ
* コントローラ、マネージャ、設定変更（コンフィギュレータ）、アップデータ

<!--
Individual pods are not intended to run multiple instances of the same
application, in general.
-->
一般的に、個々のポッドでは同じアプリケーションを複数実行するのは意図されていません。

<!--
For a longer explanation, see [The Distributed System ToolKit: Patterns for
Composite
Containers](https://kubernetes.io/blog/2015/06/the-distributed-system-toolkit-patterns).
--->
より長い説明は、 [分散システムツールキット：コンテナが混在するためのパターン(https://kubernetes.io/blog/2015/06/the-distributed-system-toolkit-patterns) をご覧ください。

<!--
## Alternatives considered
-->
## 代替案の考慮 {#alternatives-considered}

<!--
_Why not just run multiple programs in a single (Docker) container?_
-->
_なぜ複数のプログラムを１つの（Docker）コンテナで実行しないのでしょうか？_

<!--
1. Transparency. Making the containers within the pod visible to the
   infrastructure enables the infrastructure to provide services to those
   containers, such as process management and resource monitoring. This
   facilitates a number of conveniences for users.
1. Decoupling software dependencies. The individual containers may be
   versioned, rebuilt and redeployed independently. Kubernetes may even support
   live updates of individual containers someday.
1. Ease of use. Users don't need to run their own process managers, worry about
   signal and exit-code propagation, etc.
1. Efficiency. Because the infrastructure takes on more responsibility,
   containers can be lighter weight.
-->
1. 透明性です。ポッド内のコンテナがインフラ（基盤）から見えるようにするため、プロセス管理とリソース監視のように、インフラがコンテナに対するサービスを提供可能になります。
1. ソフトウェア依存性を切り離します。個々のコンテナは、別々にバージョン管理、再構築（リビルド）、再配置（リデプロイ）されます。Kubernetes は各コンテナの個別なライブ・アップデートに対応しています。
1. 扱うのが簡単です。ユーザは自分でプロセス・マネージャを実行する必要は無く、シグナルや終了コードの伝播（プロパゲーション）などの心配が不要です。
1. 効率性です。基盤（インフラ）側が責任を取るため、コンテナは軽量なままです。

<!--
_Why not support affinity-based co-scheduling of containers?_
-->
_なぜアフィニティ（親和性）をベースとしたコンテナの協働スケジューリングをサポートしないのですか？_

<!--
That approach would provide co-location, but would not provide most of the
benefits of pods, such as resource sharing, IPC, guaranteed fate sharing, and
simplified management.
-->
この手法は同じ場所での動作（co-location）を提供しますが、リソース共有、IPC、同時終了の保証（guaranteed fate sharing）、シンプルな監視など、ポッドの大部分の利点を提供しません。

<!--
## Durability of pods (or lack thereof)
-->
## ポッドの耐久性（あるいは耐久性の欠如） {#durability-of-pods-or-lack-thereof}

<!--
Pods aren't intended to be treated as durable entities. They won't survive scheduling failures, node failures, or other evictions, such as due to lack of resources, or in the case of node maintenance.
-->
ポッドを耐久性があるものとして扱うのは意図していません。スケジューリングの失敗、ノードの障害、リソースの欠乏やノードのメンテナンスなど、その他の退去（eviction）があれば存続しません。

<!--
In general, users shouldn't need to create pods directly. They should almost
always use controllers even for singletons, for example,
[Deployments](/docs/concepts/workloads/controllers/deployment/)).
Controllers provide self-healing with a cluster scope, as well as replication
and rollout management.
Controllers like [StatefulSet](/docs/concepts/workloads/controllers/statefulset.md)
can also provide support to stateful pods.
-->
通常、ユーザはポッドを直接作成する必要はありません。ほとんどの場合、常に個々のコントローラ（例：([Deployments（デプロイメント）](/jp/docs/concepts/workloads/controllers/deployment/)）を使うべきでしょう。コントローラはクラスタ領域内で自己修復（セルフ・ヒーリング）を提供するだけでなく、複製（レプリケーション）と展開（ロールアウト）も管理します。また、[StatefulSet（ステートフルセット）](/docs/concepts/workloads/controllers/statefulset/) のようなコントローラは、ステートフルな（状態を保存する）ポッドのサポートを提供できます。

<!--
The use of collective APIs as the primary user-facing primitive is relatively common among cluster scheduling systems, including [Borg](https://research.google.com/pubs/pub43438.html), [Marathon](https://mesosphere.github.io/marathon/docs/rest-api.html), [Aurora](http://aurora.apache.org/documentation/latest/reference/configuration/#job-schema), and [Tupperware](http://www.slideshare.net/Docker/aravindnarayanan-facebook140613153626phpapp02-37588997).
-->
[Borg](https://research.google.com/pubs/pub43438.html)、 [Marathon](https://mesosphere.github.io/marathon/docs/rest-api.html)、 [Aurora](http://aurora.apache.org/documentation/latest/reference/configuration/#job-schema)、 [Tupperware](http://www.slideshare.net/Docker/aravindnarayanan-facebook140613153626phpapp02-37588997) といった
クラスタ・スケジューリング・リステムに比べると、ユーザが直面するのは API の集合を使った操作です。

<!--
Pod is exposed as a primitive in order to facilitate:
-->
ポッドはプリミティブを容易にする（ファシリテートする）手段として、外に向かって公開します：
ポッドを使いやすくするために、プリミティブを外に向かって公開（expose）します：

<!--
* scheduler and controller pluggability
* support for pod-level operations without the need to "proxy" them via controller APIs
* decoupling of pod lifetime from controller lifetime, such as for bootstrapping
* decoupling of controllers and services &mdash; the endpoint controller just watches pods
* clean composition of Kubelet-level functionality with cluster-level functionality &mdash; Kubelet is effectively the "pod controller"
* high-availability applications, which will expect pods to be replaced in advance of their termination and certainly in advance of deletion, such as in the case of planned evictions or image prefetching.
-->
* スケジューラとコントローラを取り付け・取り外しやすいようにする（pluggability）
* コントローラ API を「代理」（proxy）することなく、ポッド単位での操作を可能とする
* 起動時の処理（ブートストラッピング）のようなコンテナのライフタイムと、ポッドのライフタイムを切り離す
* コントローラとサービスを切り離す。つまり、エンドポイント・コントローラはポッドしか監視しない
* kubelet レベルの機能性をクラスタ・レベルの機能性と共にクリーンにするため、kubelet は効率的な「ポッド・コントローラ」である
* アプリケーションの高可用性のためには、退去計画（planned eviction）やイメージの先行取得（image prefetching）のように、ポッドを事前に置き換え（リプレース）たり、ポッドの確実な事前削除を行ったりする可能性があります。

<!--
## Termination of Pods
-->
## ポッドの終了（Termination） {#termination-of-pods}

<!--
Because pods represent running processes on nodes in the cluster, it is important to allow those processes to gracefully terminate when they are no longer needed (vs being violently killed with a KILL signal and having no chance to clean up). Users should be able to request deletion and know when processes terminate, but also be able to ensure that deletes eventually complete. When a user requests deletion of a pod the system records the intended grace period before the pod is allowed to be forcefully killed, and a TERM signal is sent to the main process in each container. Once the grace period has expired the KILL signal is sent to those processes and the pod is then deleted from the API server. If the Kubelet or the container manager is restarted while waiting for processes to terminate, the termination will be retried with the full grace period.
-->
ポッドとは、クラスタ内のノード上で実行するプロセスを表しているため、各プロセスの必要性がなくなれば、プロセスを丁寧に終了（gracefully terminate）するのは重要です（対して、KILL シグナルで手荒に停止すると、クリーンアップするための機会も与えません）。ユーザは削除の要求を送れるようにすべきであり、いつプロセスが終了（terminate）するかを知れるようにする必要があります。しかしまた、最終的に確実に削除完了とする必要もあります。ユーザがポッドの削除をシステム・レコードに対して要求する時には、ポッドを強制的に停止可能にするのではなく、猶予期間（grace period）に到達するまで停止できるよう、各コンテナ内のメインプロセスに対して TERM シグナルを送信します。猶予期間が時間切れとなれば、KILL シグナルを各プロセスに対して送信し、API サーバからポッドが削除されます。もしもプロセス終了の待機中に kubelet やコンテナ・マネージャが再起動した場合は、猶予期間に到達していなければ停止を再試行します。

<!--
An example flow:
-->
例となるフロー：

<!--
1. User sends command to delete Pod, with default grace period (30s)
1. The Pod in the API server is updated with the time beyond which the Pod is considered "dead" along with the grace period.
1. Pod shows up as "Terminating" when listed in client commands
1. (simultaneous with 3) When the Kubelet sees that a Pod has been marked as terminating because the time in 2 has been set, it begins the pod shutdown process.
    1. If the pod has defined a [preStop hook](/docs/concepts/containers/container-lifecycle-hooks/#hook-details), it is invoked inside of the pod. If the `preStop` hook is still running after the grace period expires, step 2 is then invoked with a small (2 second) extended grace period.
    1. The processes in the Pod are sent the TERM signal.
1. (simultaneous with 3) Pod is removed from endpoints list for service, and are no longer considered part of the set of running pods for replication controllers. Pods that shutdown slowly cannot continue to serve traffic as load balancers (like the service proxy) remove them from their rotations.
1. When the grace period expires, any processes still running in the Pod are killed with SIGKILL.
1. The Kubelet will finish deleting the Pod on the API server by setting grace period 0 (immediate deletion). The Pod disappears from the API and is no longer visible from the client.
-->
1. ユーザはポッドを削除するためにコマンドを送信します。デフォルトの猶予期間は 30 秒です。
1. API サーバ内では、猶予期間内にポッドが「dead」（停止）に更新されるとみなされます。
1. クライアントで一覧コマンドを実行すると、ポッドは「Terminating」（停止中）として表示されます。
1. （3 と似ていますが）Kubelete を見ると、ポッドは停止中のマークが付けられて見えます。これは 2 の手順が開始され、ポッドの停止手続きが始まったからです。
    1. もしもポッドに [preStop hook（停止前フック）](/docs/concepts/containers/container-lifecycle-hooks/#hook-details) の指定があれば、ポッドの中で呼び出されます。もしも `preStop` フックが猶予期間が切れても実行中であれば、手順 2 は短い間（2秒）猶予期間が延長されます。
    1. ポットの中にプロセスに対して `TERM` シグナルが送信されます。
1. （3 と似ていますが）サービスの一覧からポッドのエンドポイントが削除され、レプリケーション・コントローラからは実行中のポッドを構成する一部とは認識されません。ポッドの停止（シャットダウン）が緩やかですが、（サービス・プロキシのように）負荷分散のトラフィック提供は継続できないため、ローテーションから削除されます。
1. 猶予期間が切れると、ポッド内のあらゆるプロセスは SIGKILL で停止させられます。
1. 最終的に Kubelet は API サーバの猶予期間を０秒とし（直ちに削除）、ポッドの削除を完了します。API からポッドは消滅し、クライアントからは二度と見えなくなります。

<!--
By default, all deletes are graceful within 30 seconds. The `kubectl delete` command supports the `--grace-period=<seconds>` option which allows a user to override the default and specify their own value. The value `0` [force deletes](/docs/concepts/workloads/pods/pod/#force-deletion-of-pods) the pod. In kubectl version >= 1.5, you must specify an additional flag `--force` along with `--grace-period=0` in order to perform force deletions.
-->

<!--
### Force deletion of pods
-->
### ポッドの強制削除（force deletion） {#force-deletion}

<!--
Force deletion of a pod is defined as deletion of a pod from the cluster state and etcd immediately. When a force deletion is performed, the apiserver does not wait for confirmation from the kubelet that the pod has been terminated on the node it was running on. It removes the pod in the API immediately so a new pod can be created with the same name. On the node, pods that are set to terminate immediately will still be given a small grace period before being force killed.
-->
ポッドの強制削除とは、クラスタの状態と etcd からポッドを直ちに削除するものと定義されています。強制削除が行われると、apiserver は kuberlet からの確認を待ちません。つまり、ノード上のポッドが稼働中であっても直ちに停止させられます。API からポッドは直ちに停止させられるため、同じ名前のポッドが作成可能になります。ノード上では、ポッドはただちに停止状態にセットされるため、短時間の猶予なく強制的に削除されます。

<!--
Force deletions can be potentially dangerous for some pods and should be performed with caution. In case of StatefulSet pods, please refer to the task documentation for [deleting Pods from a StatefulSet](/docs/tasks/run-application/force-delete-stateful-set-pod/).
-->
ポッドによっては強制削除は危険であり、実施には注意を払うべきです。StatefulSet ポッドの場合、 [StatefulSet からポッドを削除](/docs/tasks/run-application/force-delete-stateful-set-pod/) にあるタスク・ドキュメントをご覧ください。

<!--
## Privileged mode for pod containers
-->
## ポッド・コンテナ用の特権モード {#-rivileged-mode-for-pod-containers}

<!--
From Kubernetes v1.1, any container in a pod can enable privileged mode, using the `privileged` flag on the `SecurityContext` of the container spec. This is useful for containers that want to use linux capabilities like manipulating the network stack and accessing devices. Processes within the container get almost the same privileges that are available to processes outside a container. With privileged mode, it should be easier to write network and volume plugins as separate pods that don't need to be compiled into the kubelet.
-->
Kubernetes v1.11 から、ポッド内のあらゆるコンテナが特権モード（privileged mode）を有効化できるようになりました。そのためには、コンテナ spec の `SecurityContext` で `privileged` フラグを使います。これはネットワーク・スタックとデバイスに接続して操作するような、 Linux ケーパビリティ（capability）を使いたいコンテナ用に役立ちます。

<!--
If the master is running Kubernetes v1.1 or higher, and the nodes are running a version lower than v1.1, then new privileged pods will be accepted by api-server, but will not be launched. They will be pending state.
If user calls `kubectl describe pod FooPodName`, user can see the reason why the pod is in pending state. The events table in the describe command output will say:
`Error validating pod "FooPodName"."FooPodNamespace" from api, ignoring: spec.containers[0].securityContext.privileged: forbidden '<*>(0xc2089d3248)true'`
-->
もしもマスタで実行している Kubernetes が v1.1 以上で、ノードで実行しているバージョンが v1.1 よりも低ければ、新しい特権ポッドは api-server では受け入れられますが、起動できません。保留中（pending）の状態になります。ユーザが `kubectl describe pod FooPodName` を呼び出すと、なぜポッドが保留状態になっているかの理由が表示されます。コマンドの出力でイベント・テーブルにある説明は、次のようなものです：
`Error validating pod "FooPodName"."FooPodNamespace" from api, ignoring: spec.containers[0].securityContext.privileged: forbidden '<*>(0xc2089d3248)true'`

<!--
If the master is running a version lower than v1.1, then privileged pods cannot be created. If user attempts to create a pod, that has a privileged container, the user will get the following error:
`The Pod "FooPodName" is invalid.
spec.containers[0].securityContext.privileged: forbidden '<*>(0xc20b222db0)true'`
-->
もしもマスタで実行しているバージョンが v1.1 よりも低ければ、特権ポッドは作成できません。ユーザが特権コンテナを持つポッドの作成を試みても、次のようなエラーを受け取るだけです：
`The Pod "FooPodName" is invalid.
spec.containers[0].securityContext.privileged: forbidden '<*>(0xc20b222db0)true'`


<!--
## API Object
-->
## API オブジェクト {#api-object}

<!--
Pod is a top-level resource in the Kubernetes REST API. More details about the
API object can be found at:
[Pod API object](/docs/reference/generated/kubernetes-api/{{< param "version" >}}/#pod-v1-core).
-->
ポッドは Kubernets REST API においてはトップ・レベルのリソースです。API オブジェクトのしょうさいについては [ポッド API オブジェクト](/docs/reference/generated/kubernetes-api/{{< param "version" >}}/#pod-v1-core) をご覧ください。


{{% /capture %}}