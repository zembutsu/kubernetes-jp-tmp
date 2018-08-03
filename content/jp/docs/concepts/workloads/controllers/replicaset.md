---
reviewers:
- Kashomon
- bprashanth
- madhusudancs
title: ReplicaSet（レプリカセット）
content_template: templates/concept
weight: 10
---

{{% capture overview %}}
<!--
ReplicaSet is the next-generation Replication Controller. The only difference
between a _ReplicaSet_ and a
[_Replication Controller_](/docs/concepts/workloads/controllers/replicationcontroller/) right now is
the selector support. ReplicaSet supports the new set-based selector requirements
as described in the [labels user guide](/docs/concepts/overview/working-with-objects/labels/#label-selectors)
whereas a Replication Controller only supports equality-based selector requirements.
-->
ReplicaSet（レプリカセット）は次世代のレプリケーション・コントローラです。 _ReplicaSet_ と [_Replication Controller（レプリケーション・コントローラ）_](/jp/docs/concepts/workloads/controllers/replicationcontroller/) との相違点は、セレクタのサポートのみです。Replication Controller がサポートするのは品質ベースのセレクタ要件であるのに対して、新しい設定ベース（set-based）セレクタの必要条件は [ラベル・ユーザ・ガイド](/jp/docs/concepts/overview/working-with-objects/labels/#label-selectors) に記述があります。

{{% /capture %}}

{{% capture body %}}
<!--
## How to use a ReplicaSet
-->
## ReplicaSet の使い方 {#how-to-use-a-replicaset}

<!--
Most [`kubectl`](/docs/user-guide/kubectl/) commands that support
Replication Controllers also support ReplicaSets. One exception is the
[`rolling-update`](/docs/reference/generated/kubectl/kubectl-commands#rolling-update) command. If
you want the rolling update functionality please consider using Deployments
instead. Also, the
[`rolling-update`](/docs/reference/generated/kubectl/kubectl-commands#rolling-update) command is
imperative whereas Deployments are declarative, so we recommend using Deployments
through the [`rollout`](/docs/reference/generated/kubectl/kubectl-commands#rollout) command.
-->
ほとんどの  [`kubectl`](/jp/docs/user-guide/kubectl/) コマンドは Replication Controller と ReplicaSet もサポートします。例外の１つは [`rolling-update`](/jp/docs/reference/generated/kubectl/kubectl-commands#rolling-update) （ローリング・アップデート）コマンドです。ローリング・アップデート機能を使いたい場合は、かわりに Deployment（デプロイメント）の使用を検討してください。また、[`rolling-update`](/jp/docs/reference/generated/kubectl/kubectl-commands#rolling-update) （ローリング・アップデート）コマンドコマンドは命令型として扱われるのに対し、Deployment（デプロイメント）では宣言型です。そのため、私たちが推奨するのは Deployment（デプロイメント）を通して [`rollout`](/docs/reference/generated/kubectl/kubectl-commands#rollout) （ロールアウト）コマンドをご利用ください。

<!--
While ReplicaSets can be used independently, today it's mainly used by
[Deployments](/docs/concepts/workloads/controllers/deployment/) as a mechanism to orchestrate pod
creation, deletion and updates. When you use Deployments you don't have to worry
about managing the ReplicaSets that they create. Deployments own and manage
their ReplicaSets.
-->
ReplicaSet の利用は独立的であり、ポッドの作成、削除、更新を調整（オーケストレート）するための仕組みとしては [Deployments](/jp/docs/concepts/workloads/controllers/deployment/) （デプロイメント）が主に使われます。Deployment （デプロイメント）の利用にあたっては、既に作成した ReplicaSet に対する心配は不要です。Deployment は自身と ReplicaSet の両方を管理します。

<!--
## When to use a ReplicaSet
-->
## ReplicaSet の利用時 {#when-to-use-a-replicaset}

<!--
A ReplicaSet ensures that a specified number of pod replicas are running at any given
time. However, a Deployment is a higher-level concept that manages ReplicaSets and
provides declarative updates to pods along with a lot of other useful features.
Therefore, we recommend using Deployments instead of directly using ReplicaSets, unless
you require custom update orchestration or don't require updates at all.
-->
ReplicaSet は、指定したポッドの複製数が常に実行中にするのを保証します。しかしながら、Deployment は ReplicaSet を管理する高レベルの概念であり、ポッドに対する宣言型の更新は、他の多くの便利な機能をもたらします。そのため、私たちは ReplicaSet を直接使うのではなく Deployment の使用を推奨します。ただし、カスタム・アップデート・オーケストレーションが必要ではない場合と、全てを更新する必要がない場合を除きます。

<!--
This actually means that you may never need to manipulate ReplicaSet objects:
use a Deployment instead, and define your application in the spec section.
-->
つまり、ReplicaSet オブジェクトを操作する必要は一切ありません。そのかわり、Deployment を使って、アプリケーションの spec セクションで定義します。

<!--
## Example
-->
## 例 {#example}

{{< codenew file="controllers/frontend.yaml" >}}

<!--
Saving this manifest into `frontend.yaml` and submitting it to a Kubernetes cluster should
create the defined ReplicaSet and the pods that it manages.
-->
このマニフェストを `frontend.yaml` に保存し、Kubernetes クラスタに送信すると、定義した ReplicaSet をポッドに作成し、管理できるようになります。

```shell
$ kubectl create -f http://k8s.io/examples/controllers/frontend.yaml
replicaset "frontend" created
$ kubectl describe rs/frontend
Name:		frontend
Namespace:	default
Selector:	tier=frontend,tier in (frontend)
Labels:		app=guestbook
		tier=frontend
Annotations:	<none>
Replicas:	3 current / 3 desired
Pods Status:	3 Running / 0 Waiting / 0 Succeeded / 0 Failed
Pod Template:
  Labels:       app=guestbook
                tier=frontend
  Containers:
   php-redis:
    Image:      gcr.io/google_samples/gb-frontend:v3
    Port:       80/TCP
    Requests:
      cpu:      100m
      memory:   100Mi
    Environment:
      GET_HOSTS_FROM:   dns
    Mounts:             <none>
  Volumes:              <none>
Events:
  FirstSeen    LastSeen    Count    From                SubobjectPath    Type        Reason            Message
  ---------    --------    -----    ----                -------------    --------    ------            -------
  1m           1m          1        {replicaset-controller }             Normal      SuccessfulCreate  Created pod: frontend-qhloh
  1m           1m          1        {replicaset-controller }             Normal      SuccessfulCreate  Created pod: frontend-dnjpy
  1m           1m          1        {replicaset-controller }             Normal      SuccessfulCreate  Created pod: frontend-9si5l
$ kubectl get pods
NAME             READY     STATUS    RESTARTS   AGE
frontend-9si5l   1/1       Running   0          1m
frontend-dnjpy   1/1       Running   0          1m
frontend-qhloh   1/1       Running   0          1m
```

<!--
## Writing a ReplicaSet Spec
-->
## ReplicaSet Spec を書く {#writing-a-replicaset-spec}

<!--
As with all other Kubernetes API objects, a ReplicaSet needs the `apiVersion`, `kind`, and `metadata` fields.  For
general information about working with manifests, see [object management using kubectl](/docs/concepts/overview/object-management-kubectl/overview/).
-->
他のすべての Kubernetes API オブジェクトと同様に、ReplicaSet には `apiVersion` 、`kind`、 `metadata`  フィールドが必要です。
マニフェストの扱い方に関する一般的な情報は [object management using kubectl](/docs/concepts/overview/object-management-kubectl/overview/) をご覧ください。

<!--
A ReplicaSet also needs a [`.spec` section](https://git.k8s.io/community/contributors/devel/api-conventions.md#spec-and-status).
-->
また、ReplicaSet には [`.spec` セレクション](https://git.k8s.io/community/contributors/devel/api-conventions.md#spec-and-status)が必要です。

<!--
### Pod Template
-->
### Pod テンプレート（Pod Template） {#pod-template}

<!--
The `.spec.template` is the only required field of the `.spec`. The `.spec.template` is a 
[pod template](/docs/concepts/workloads/pods/pod-overview/#pod-templates). It has exactly the same schema as a 
[pod](/docs/concepts/workloads/pods/pod/), except that it is nested and does not have an `apiVersion` or `kind`.
-->
`.spec.template` は `.spec` で唯一必須のフィールドです。
`.spec.template` は [ポッド template](/jp/docs/concepts/workloads/pods/pod-overview/#pod-templates) です。
これは厳密には [ポッド](/jp/docs/concepts/workloads/pods/pod/) と同じスキーマですが、ネスト化（階層化）しているのと、 `apiVersion` や `kind` を持たないのが違います。

<!--
In addition to required fields of a pod, a pod template in a ReplicaSet must specify appropriate
labels and an appropriate restart policy.
-->
ポッドの必須フィールドに付け加えると、ReplicaSet 内のポッド・テンプレートには適切なラベルと適切な再起動方針（restart policy）の指定が必須です。

<!--
For labels, make sure to not overlap with other controllers. For more information, see [pod selector](#pod-selector).
-->
ラベルに関しては、他のコントローラと重複しないようにする必要があります。
詳しい情報は[ポッド・セレクタ（pod selector）](#pod-selector)をご覧ください。

<!--
For [restart policy](/docs/concepts/workloads/pods/pod-lifecycle/#restart-policy), the only allowed value for `.spec.template.spec.restartPolicy` is `Always`, which is the default.
-->
[再起動方針（restart policy）](/jp/docs/concepts/workloads/pods/pod-lifecycle/#restart-policy) については、許可されている値は `.spec.template.spec.restartPolicy`  に対してのみであり、デフォルトは `Always` （常時）です。

<!--
For local container restarts, ReplicaSet delegates to an agent on the node,
for example the [Kubelet](/docs/admin/kubelet/) or Docker.
-->
ローカル・コンテナの再起動については、ReplicaSet はノード上のエージェントに権限を委譲しています。例えば [Kubelet](/jp/docs/admin/kubelet/) や Docker です。

<!--
### Pod Selector
-->
### ポッド・セレクタ（Pod Selector）{#pod-selector}

<!--
The `.spec.selector` field is a [label selector](/docs/concepts/overview/working-with-objects/labels/). A ReplicaSet
manages all the pods with labels that match the selector. It does not distinguish
between pods that it created or deleted and pods that another person or process created or
deleted. This allows the ReplicaSet to be replaced without affecting the running pods.
-->
`.spec.selector` フィールドは[ラベル・セレクタ](/jp/docs/concepts/overview/working-with-objects/labels/)です。
ReplicaSet は全てのポッドの管理を、ラベルとラベルに一致するセレクタで行います。
セレクタはポッドの作成または削除時に、ポッドが人もしくはプロセスが作成または削除するかどうかを区別しません。
これにより、ReplicaSet は実行しているポッドの影響を受けることなく、置き換え（replace）可能です。

<!--
The `.spec.template.metadata.labels` must match the `.spec.selector`, or it will
be rejected by the API.
-->
`.spec.template.metadata.labels` は `.spec.selector` と一致する必要があります。一致しない場合は API によって拒否されます。

<!--
In Kubernetes 1.9 the API version `apps/v1` on the ReplicaSet kind is the current version and is enabled by default. The API version `apps/v1beta2` is deprecated.
-->
Kubernetes 1.9 の API バージョン `apps/v1` では、ReplicaSet kind が現在のバージョンではデフォルトで有効化されています。
API バージョン `apps/v1beta2` は廃止済みです。

<!--
Also you should not normally create any pods whose labels match this selector, either directly, with 
another ReplicaSet, or with another controller such as a Deployment. If you do so, the ReplicaSet thinks that it 
created the other pods. Kubernetes does not stop you from doing this.
-->
また、ポッドの作成にあたって、このセレクタに一致するラベルのポッドを重複して指定できません。
さらに、他の ReplicaSet や、他の Deployment などのコントローラ等とも重複できません。
もしも重複すると、ReplicaSet は他のポッドを作成したと認識します。
Kubernetes はこのような作業を止められません。

<!--
If you do end up with multiple controllers that have overlapping selectors, you
will have to manage the deletion yourself.
-->
もしも複数のコントローラが存在してしまうと、セレクタが重複してしまい、削除を自分自身で行う必要が出てきます。

<!--
### Labels on a ReplicaSet
-->
### ReplicaSet 上のラベル {#labels-on-a-replicaset}

<!--
The ReplicaSet can itself have labels (`.metadata.labels`).  Typically, you
would set these the same as the `.spec.template.metadata.labels`.  However, they are allowed to be
different, and the `.metadata.labels` do not affect the behavior of the ReplicaSet.
-->
ReplicaSet は自分自身のラベルを持てます（`.metadata.labels`）。
典型的なのは `.spec.template.metadata.labels` と同じものをセットする方法です。
しかしながら、これらは別々のものとして認識されるため、ReplicaSet の挙動に影響を与えないためには `.metadata.labels` を使います。

<!--
### Replicas
-->
### 複製（レプリカ：Replicas） {#replicas}

<!--
You can specify how many pods should run concurrently by setting `.spec.replicas`. The number running at any time may be higher
or lower, such as if the replicas were just increased or decreased, or if a pod is gracefully
shut down, and a replacement starts early.
-->
いくつのポッドを同時に実行すべきかを `.spec.replicas` で指定できます。
実行中の数は常に増減する可能性がありますが、もしも複製が増えたり減ったりしたら、ポッドは丁寧にシャットダウンするか、早々に複製を起動します。

<!--
If you do not specify `.spec.replicas`, then it defaults to 1.
-->
`.spec.replicas` を指定しなければ、デフォルトの 1 として扱われます。

<!--
## Working with ReplicaSets
-->
## ReplicaSets を使う {#working-with-replicasets}

<!--
### Deleting a ReplicaSet and its Pods
-->
### ReplicaSet と Pod の削除 {#deleting-a-replicaset-and-its-pods}

<!--
To delete a ReplicaSet and all its pods, use [`kubectl
delete`](/docs/reference/generated/kubectl/kubectl-commands#delete). Kubectl will scale the ReplicaSet to zero and wait
for it to delete each pod before deleting the ReplicaSet itself. If this kubectl command is interrupted, it can
be restarted.
-->
ReplicaSet とポッドの全てを削除するには、[`kubectl delete`](/jp/docs/reference/generated/kubectl/kubectl-commands#delete) を使います。
Kubectl は ReplicaSet をゼロにスケール（規模変更）し、各ポッドが削除されるまで待機したら、ReplicaSet 自身を削除します。
kubectl コマンドが中断されると、再起動されます。

<!--
When using the REST API or go client library, you need to do the steps explicitly (scale replicas to
0, wait for pod deletions, then delete the ReplicaSet).
-->
REST API や go クライアント・ライブラリを使用する場合は、各ステップを明示的に行う必要があります（複製を 0 にスケールし、ポッドの削除まで待機し、その後 ReplicaSet を削除）。


<!--
### Deleting just a ReplicaSet
-->
### ReplicaSet で削除 {deleting-just-a-replicaset}

<!--
You can delete a ReplicaSet without affecting any of its pods, using [`kubectl delete`](/docs/reference/generated/kubectl/kubectl-commands#delete) with the `--cascade=false` option.
-->
[`kubectl delete`](/jp/docs/reference/generated/kubectl/kubectl-commands#delete) に `--cascade=false` オプションを指定すると、あらゆるポッドに影響を与えず ReplicaSet を削除できます。

<!--
When using the REST API or go client library, simply delete the ReplicaSet object.
-->
REST API や go クライアント・ライブラリを使う場合は、単純に ReplicaSet オブジェクトを削除します。

<!--
Once the original is deleted, you can create a new ReplicaSet to replace it.  As long
as the old and new `.spec.selector` are the same, then the new one will adopt the old pods.
However, it will not make any effort to make existing pods match a new, different pod template.
To update pods to a new spec in a controlled way, use a [rolling update](#rolling-updates).
-->
オリジナルの ReplicaSetを削除したら、新しい ReplicaSet を作成し、置き換えできます。
新旧どちらの `.spec.selector` が同じであれば、新しい ReplicaSet が古いほうのポッドを扱えます。
しかしながら、既存のポットと新しいポッドが異なるポッド・テンプレートの場合、何ら影響を及ぼせません。
ポッドを新しい spec で管理できるように更新するには、 [ローリング・アップデート](#rolling-update) を使います。

<!--
### Isolating pods from a ReplicaSet
-->
### ポッドを他の ReplicaSet から独立（Isolating）する {#isolating-pods-from-a-replicaset}

<!--
Pods may be removed from a ReplicaSet's target set by changing their labels. This technique may be used to remove pods 
from service for debugging, data recovery, etc. Pods that are removed in this way will be replaced automatically (
  assuming that the number of replicas is not also changed).
-->
ポッドを ReplicaSet の対象から削除するには、ラベルの変更で行える場合があります。
このテクニックはサービスのデバッグ用やデータ修復用など、ポッドを削除するために使えるでしょう。
この方法によって削除すると、自動的に新しいものが置き換えられます（複製の数は変更していないとみなした場合）。

<!--
### Scaling a ReplicaSet
-->
### ReplicaSet のスケーリング {#scalling-replicaset}

<!--
A ReplicaSet can be easily scaled up or down by simply updating the `.spec.replicas` field. The ReplicaSet controller
ensures that a desired number of pods with a matching label selector are available and operational.
-->
ReplicaSet は `.spec.replicas` フィールドをシンプルに変更するだけで、簡単にスケールアップまたはダウンができます。
ReplicaSet コントローラはポッドを希望する数の維持を確実に行い、一致するラベル・セレクタを利用可能かつ操作可能にします。

<!--
### ReplicaSet as an Horizontal Pod Autoscaler Target
-->
### ReplicaSet を水平ポッド・オートスケーラのターゲットとする {replicaset-as-an-horizontal-pod-autoscaler-target}

<!--
A ReplicaSet can also be a target for
[Horizontal Pod Autoscalers (HPA)](/docs/tasks/run-application/horizontal-pod-autoscale/). That is,
a ReplicaSet can be auto-scaled by an HPA. Here is an example HPA targeting
the ReplicaSet we created in the previous example.
-->
また、ReplicaSet は [水平ポッド・オートスケーラ(HPA)](/jp/docs/tasks/run-application/horizontal-pod-autoscale/) の対象（ターゲット）にもなれます。
これにより、ReplicaSet は HPA によってオートスケールされるようになります。
こちらにあるのは以前の例で作成した ReplicaSet を HPA のターゲットとなるようにしたサンプルです。

{{< codenew file="controllers/hpa-rs.yaml" >}}

<!--
Saving this manifest into `hpa-rs.yaml` and submitting it to a Kubernetes cluster should
create the defined HPA that autoscales the target ReplicaSet depending on the CPU usage
of the replicated pods.
-->
このマニフェストを `hpa-rs.yaml` として保存し、 Kubernetes クラスタに対して送信すると、定義した HPA が作成され、ターゲットとなる ReplicaSet は複製されたポッドの CPU 使用率に応じてオートスケールします。

```shell
kubectl create -f https://k8s.io/examples/controllers/hpa-rs.yaml
```

<!--
Alternatively, you can use the `kubectl autoscale` command to accomplish the same
(and it's easier!)
-->
あるいは、 `kubectl autocale` コマンドを使って同じ処理をします（そして、こちらが簡単です！）。

```shell
kubectl autoscale rs frontend
```

<!--
## Alternatives to ReplicaSet
-->
## ReplicaSet の代替 {#alternatives-to-replicaset}

<!--
### Deployment (Recommended)
-->
### Deployment（デプロイメント）（推奨） {#deployment-recommended}

<!--
[`Deployment`](/docs/concepts/workloads/controllers/deployment/) is a higher-level API object that updates its underlying ReplicaSets and their Pods
in a similar fashion as `kubectl rolling-update`. Deployments are recommended if you want this rolling update functionality,
because unlike `kubectl rolling-update`, they are declarative, server-side, and have additional features. For more information on running a stateless
application using a Deployment, please read [Run a Stateless Application Using a Deployment](/docs/tasks/run-application/run-stateless-application-deployment/).
-->
[`Deployment（デプロイメント）`](/jp/docs/concepts/workloads/controllers/deployment/) は高レベルの API オブジェクトであり、これを使って基礎となる ReplicaSet とそれらのポッドを `kubectl rolling-update` のように簡単に更新します。
Deployment はローリング・アップデートを機能的に使いたい時に推奨されます。
なぜなら、 `kubectl rolling-update` とは異なり、こちらは宣言型であり、サーバ側での処理（サーバサイド）で、かつ、追加機能があります。
ステートレス名アプリケーションを実行するために Deployment を使う場合は [Deployment でステートレスなアプリケーションを実行する](/jp/docs/tasks/run-application/run-stateless-application-deployment/) をご覧ください。

<!--
### Bare Pods
-->
### ベア・ポッド （Bare Pod） {#bare-pods}

<!--
Unlike the case where a user directly created pods, a ReplicaSet replaces pods that are deleted or terminated for any reason, such as in the case of node failure or disruptive node maintenance, such as a kernel upgrade. For this reason, we recommend that you use a ReplicaSet even if your application requires only a single pod. Think of it similarly to a process supervisor, only it supervises multiple pods across multiple nodes instead of individual processes on a single node. A ReplicaSet delegates local container restarts to some agent on the node (for example, Kubelet or Docker).
-->
ユーザが直接ポッドを作成するのとは異なり、ReplicaSet がポッドを置き換えます。これはノードの障害や、カーネルのアップグレードのようなノードの分断的なメンテナンスなどの状況下など、あらゆる理由によってポッドの削除または停止を ReplicaSet が行います。
そのため、アプリケーションが必要となるのが１つのポッドだとしても、私たちは ReplicaSet の使用を推奨します。
プロセスのスーパーバイザと同じようなものと考えると、1つのノード上で個々のプロセスを管理するのではなく、複数のノードに横断する複数のポッドをスーパーバイズ（監督）するようなものです。
ReplicaSet はローカルにあるコンテナの再起動ですら、ノード上の何らかのコントローラに委任します（例：Kubelet や Docker）。

<!--
### Job
-->
### ジョブ（Job） {#job}

<!--
Use a [`Job`](/docs/concepts/jobs/run-to-completion-finite-workloads/) instead of a ReplicaSet for pods that are expected to terminate on their own
(that is, batch jobs).
-->
ポッドに ReplicaSet の代わりに [`Job（ジョブ）`](/jp/docs/concepts/jobs/run-to-completion-finite-workloads/) を使う方法です。これは各々が自身を終了（terminate）するのが期待されます（つまり、バッチ・ジョブです）。

<!--
### DaemonSet
-->
### DaemonSet（デーモンセット） {#daemonset}

<!--
Use a [`DaemonSet`](/docs/concepts/workloads/controllers/daemonset/) instead of a ReplicaSet for pods that provide a
machine-level function, such as machine monitoring or machine logging.  These pods have a lifetime that is tied
to a machine lifetime: the pod needs to be running on the machine before other pods start, and are
safe to terminate when the machine is otherwise ready to be rebooted/shutdown.
-->
ポッドに対して ReplicaSet の代わりに [`DaemonSet`（デーモンセット）](/jp/docs/concepts/workloads/controllers/daemonset/) を使う方法です。
こちらはマシンの監視やマシンのログ記録など、マシン・レベルの機能を提供します。
各ポッドはマシンのライフタイムに強く結びつくライフタイム（生存期間）を持ちます。つまり、ポッドに必要なのはマシン上で他のポッドの起動前に実行することであり、マシンが再起動またはシャットダウンなど利用できなくなる前に、安全に停止できるようにします。

{{% /capture %}}


