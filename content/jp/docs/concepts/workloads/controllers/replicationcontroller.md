---
reviewers:
- bprashanth
- janetkuo
title: ReplicationController（レプリケーション・コントローラ）
content_template: templates/concept
weight: 20
---

{{% capture overview %}}

{{< note >}}
<!--
**NOTE:** A [`Deployment`](/docs/concepts/workloads/controllers/deployment/) that configures a [`ReplicaSet`](/docs/concepts/workloads/controllers/replicaset/) is now the recommended way to set up replication.
-->
**メモ：**  複製（レプリケーション）をセットアップするには、[`Deployment`（デプロイメント）](/jp/docs/concepts/workloads/controllers/deployment/) の使用が [`ReplicaSet`（レプリカ・セット）](/jp/docs/concepts/workloads/controllers/replicaset/) よりも推奨される手法です。
{{< /note >}}

<!--
A _ReplicationController_ ensures that a specified number of pod replicas are running at any one
time. In other words, a ReplicationController makes sure that a pod or a homogeneous set of pods is
always up and available.
-->
_ReplicationController_ （レプリケーション・コントローラ）は指定した数のポッド複製（レプリカ）の常時実行を確実に行います。
言い換えると、ReplicationController はポッドまたはポッドと同質のものを常に起動かつ利用可能にします。

{{% /capture %}}


{{% capture body %}}

<!--
## How a ReplicationController Works
-->
### ReplicationController の挙動 {#how-a-replicationcontroller-works}

<!--
If there are too many pods, the ReplicationController terminates the extra pods. If there are too few, the
ReplicationController starts more pods. Unlike manually created pods, the pods maintained by a
ReplicationController are automatically replaced if they fail, are deleted, or are terminated.
For example, your pods are re-created on a node after disruptive maintenance such as a kernel upgrade.
For this reason, you should use a ReplicationController even if your application requires
only a single pod. A ReplicationController is similar to a process supervisor,
but instead of supervising individual processes on a single node, the ReplicationController supervises multiple pods
across multiple nodes.
-->
大量のポッドが存在すると、ReplicationController は余分なポッドを終了します。
あまりにも少なければ、ReplicationController は追加ポッドを起動します。
手動でポッドを作成するのとは異なり、ポッドの維持は ReplicationController によって行われ、もしも障害があれば自動的に置き換えますし、自動的に削除や終了を処理します。
たとえば、ノードがカーネルのアップグレードのような破壊的なメンテナスをした後に、ポッドを自動的に再作成します。
そのため、アプリケーションが必要なのはポッド１つだとしても、ReplicationController を使うべきでしょう。
ReplicationController はプロセス・スーパーバイザと似ていますが、１つのノード上の個々のプロセスを監視するスーパーバイザとは異なり、ReplicationController は複数のノードを横断する複数のポッドを監視（スーパーバイズ）します。

<!--
ReplicationController is often abbreviated to "rc" or "rcs" in discussion, and as a shortcut in
kubectl commands.
-->
ReplicationController は議論において頻繁に「rc」や「rcs」と省略されます。また、同様に kubectl コマンドでもショートカットとして使えます。

<!--
A simple case is to create one ReplicationController object to reliably run one instance of
a Pod indefinitely.  A more complex use case is to run several identical replicas of a replicated
service, such as web servers.
-->
簡単な利用例は、１つの ReplicationController オブジェクトを作成し、直ちにポッドの中で１つのインスタンスを確実に実行することです。
より複雑な利用例は、ウェブサーバのようなサービスを複製するために、複数の全く同じ複製（レプリカ）を実行します。

<!--
## Running an example ReplicationController
-->
## ReplicationController 実行例 {#running-an-example-replicationcontroller}

<!--
This example ReplicationController config runs three copies of the nginx web server.
-->
こちらはnginx ウェブサーバの３つのコピーを ReplicationController で設定する例です。

{{< codenew file="controllers/replication.yaml" >}}

<!--
Run the example job by downloading the example file and then running this command:
-->
サンプルのジョブを実行するには、こちらのコマンドを実行し、サンプルファイルをダウンロードして実行します：

```shell
$ kubectl create -f https://k8s.io/examples/controllers/replication.yaml
replicationcontroller "nginx" created
```

<!--
Check on the status of the ReplicationController using this command:
-->
ReplicationController の状態を確認するにはこちらのコマンドを使います：

```shell
$ kubectl describe replicationcontrollers/nginx
Name:        nginx
Namespace:   default
Selector:    app=nginx
Labels:      app=nginx
Annotations:    <none>
Replicas:    3 current / 3 desired
Pods Status: 0 Running / 3 Waiting / 0 Succeeded / 0 Failed
Pod Template:
  Labels:       app=nginx
  Containers:
   nginx:
    Image:              nginx
    Port:               80/TCP
    Environment:        <none>
    Mounts:             <none>
  Volumes:              <none>
Events:
  FirstSeen       LastSeen     Count    From                        SubobjectPath    Type      Reason              Message
  ---------       --------     -----    ----                        -------------    ----      ------              -------
  20s             20s          1        {replication-controller }                    Normal    SuccessfulCreate    Created pod: nginx-qrm3m
  20s             20s          1        {replication-controller }                    Normal    SuccessfulCreate    Created pod: nginx-3ntk0
  20s             20s          1        {replication-controller }                    Normal    SuccessfulCreate    Created pod: nginx-4ok8v
```

<!--
Here, three pods are created, but none is running yet, perhaps because the image is being pulled.
A little later, the same command may show:
-->
こちらでは、３つのポッドが作成されましたが、まだ起動中ではありません。おそらく、イメージを取得しているからでしょう。もうしばらく待ってから同じコマンドを実行すると、次のように表示されるでしょう：

```shell
Pods Status:    3 Running / 0 Waiting / 0 Succeeded / 0 Failed
```

<!--
To list all the pods that belong to the ReplicationController in a machine readable form, you can use a command like this:
-->
マシン上で ReplicationController に所属しちえる全ポッドの一覧を表示するには、次のようなコマンドを使えます：

```shell
$ pods=$(kubectl get pods --selector=app=nginx --output=jsonpath={.items..metadata.name})
echo $pods
nginx-3ntk0 nginx-4ok8v nginx-qrm3m
```

<!--
Here, the selector is the same as the selector for the ReplicationController (seen in the
`kubectl describe` output, and in a different form in `replication.yaml`.  The `--output=jsonpath` option
specifies an expression that just gets the name from each pod in the returned list.
-->
ここでは、セレクタは ReplicationController と同じセレクタにしています。 `kubectl describe` の出力で見られますが `repliation.yaml` とは違う形式になっています。 `--output=jsonpath` オプションを指定して、各ポッドごとの名前をリストにして表示できます。

<!--
## Writing a ReplicationController Spec
-->
## ReplicationController Spec を書くには {#writing-a-replicationcontroller-spec}

<!--
As with all other Kubernetes config, a ReplicationController needs `apiVersion`, `kind`, and `metadata` fields.
For general information about working with config files, see [object management ](/docs/concepts/overview/object-management-kubectl/overview/).
-->
他すべての Kubernetes 設定と同様に、 ReplicationController には `apiVersion` 、 `kind` 、 `metadata` フィールドが必要です。
設定情報ファイルの役割に関する一般的な情報は、[オブジェクト管理](/jp/docs/concepts/overview/object-management-kubectl/overview/) をご覧ください。

<!--
A ReplicationController also needs a [`.spec` section](https://git.k8s.io/community/contributors/devel/api-conventions.md#spec-and-status).
-->
また、ReplicationController には  [`.spec` セレクション](https://git.k8s.io/community/contributors/devel/api-conventions.md#spec-and-status) が必要です。

<!--
### Pod Template
-->
### ポッド・テンプレート {#pod-template}

<!--
The `.spec.template` is the only required field of the `.spec`.
-->
`.spec.template` に最低限必要なフィールドは `.spec` です。

<!--
The `.spec.template` is a [pod template](/docs/concepts/workloads/pods/pod-overview/#pod-templates). It has exactly the same schema as a [pod](/docs/concepts/workloads/pods/pod/), except it is nested and does not have an `apiVersion` or `kind`.
-->
`.spec.template` は [ポッド・テンプレート](/jp/docs/concepts/workloads/pods/pod-overview/#pod-templates) です。
これは [ポッド](/jp/docs/concepts/workloads/pods/pod/) と完全に同じスキーマですが、階層下されておらず、 `apiVersion` や `kind` を持ちません。

<!--
In addition to required fields for a Pod, a pod template in a ReplicationController must specify appropriate
labels and an appropriate restart policy. For labels, make sure not to overlap with other controllers. See [pod selector](#pod-selector).
-->
ポッドに対して必要なフィールドとしては、ReplicationController のポッドテンプレートでは、適切なラベルと適切な再起動方針（restart policy）の指定が必須です。
ラベルの場合、他のコントローラと重複しないようにする必要があります。
[ポッド・セレクタ](#pod-selector) をご覧ください。

<!--
Only a [`.spec.template.spec.restartPolicy`](/docs/concepts/workloads/pods/pod-lifecycle/#restart-policy) equal to `Always` is allowed, which is the default if not specified.
-->
[`.spec.template.spec.restartPolicy`](/jp/docs/concepts/workloads/pods/pod-lifecycle/#restart-policy) のデフォルトが指定されていなければ、常に `Always` が許可されたものとみなします。

<!--
For local container restarts, ReplicationControllers delegate to an agent on the node,
for example the [Kubelet](/docs/admin/kubelet/) or Docker.
-->
ローカル・コンテナの再起動については、ReplicationControllersあノード上のエージェントに権限を委譲しています。例えば [Kubelet](/jp/doc/admin/kubelet/) や Docker です。

<!--
### Labels on the ReplicationController
-->
### ReplicationController 上のラベル {#labels-on-the-replicationcontroller}

<!--
The ReplicationController can itself have labels (`.metadata.labels`).  Typically, you
would set these the same as the `.spec.template.metadata.labels`; if `.metadata.labels` is not specified
then it defaults to  `.spec.template.metadata.labels`.  However, they are allowed to be
different, and the `.metadata.labels` do not affect the behavior of the ReplicationController.
-->
ReplicationController は自身のラベル（ `.metadata.labels`）を持てます。
たいていは、`.spec.template.metadata.labels` と同じものを設定します。
つまり `.metadata.labels` を指定しなければ、デフォルトで `.spec.template.metadata.labels` となります。
しかしながら、異なったものを指定できますし、 `.metadata.labels` は ReplicationController の挙動に対して影響を与えません。

<!--
### Pod Selector
-->
### ポッド・セレクタ {#pod-selector}

<!--
The `.spec.selector` field is a [label selector](/docs/concepts/overview/working-with-objects/labels/#label-selectors). A ReplicationController
manages all the pods with labels that match the selector. It does not distinguish
between pods that it created or deleted and pods that another person or process created or
deleted. This allows the ReplicationController to be replaced without affecting the running pods.
-->
`.spec.selector` フィールドは  [ラベル・セレクタ](/jp/docs/concepts/overview/working-with-objects/labels/#label-selectors) です。
ReplicationController はセレクタが一致するラベルを持つポッドすべてを管理します。
これはポッド間の違いについては認識しません。つまり、ポッドが他人またはプロセスによって作成されたか削除されたかは識別しません。
これにより、ReplicationController は実行中のポッドに影響を与えることなく置き換えが可能です。

<!--
If specified, the `.spec.template.metadata.labels` must be equal to the `.spec.selector`, or it will
be rejected by the API.  If `.spec.selector` is unspecified, it will be defaulted to
`.spec.template.metadata.labels`.
-->
`.spec.template.metadata.labels` を指定する場合は、 `.spec.selector` と一緒にする必要があります。そうしないと API から拒否されます。
もしも `.spec.selector` を指定しなければ、`.spec.template.metadata.labels` がデフォルトとなります。

<!--
Also you should not normally create any pods whose labels match this selector, either directly, with 
another ReplicationController, or with another controller such as Job. If you do so, the
ReplicationController thinks that it created the other pods.  Kubernetes does not stop you
from doing this.
-->
また、通常はラベルがセレクタと一致するポッドを作成すべきではありません。
どうしても必要があれば、他の ReplicationController を直接使うか、ジョブのような他のコントローラを使います。
そうしておけば、ReplicationController は他のポッドが作成されたと考えます。
こうすることで、Kubernetes は作成したものを停止しません。

<!--
If you do end up with multiple controllers that have overlapping selectors, you
will have to manage the deletion yourself (see [below](#working-with-replicationcontrollers)).
-->
もしも、最終的に複数のコントローラを使うことになれば、セレクタが重複するでしょう。そのような場合は、自分自身で削除を管理する必要があります（詳細は [以下](#working-with-replicationcontrollers)）。

<!--
### Multiple Replicas
-->
### 複数の複製{#multiple-replicas}

<!--
You can specify how many pods should run concurrently by setting `.spec.replicas` to the number
of pods you would like to have running concurrently.  The number running at any time may be higher
or lower, such as if the replicas were just increased or decreased, or if a pod is gracefully
shutdown, and a replacement starts early.
-->
`.spec.replicas` の指定によって、並列に実行すべきポッドの数を指定できます。
ポッド数の設定はいつでも上下できますので、複製（レプリカ）を常に増加または減少できます。
あるいは、ポッドに対する丁寧なシャットダウンや、早期の複製の作成が可能です。

<!--
If you do not specify `.spec.replicas`, then it defaults to 1.
-->
`.spec.replicas` を指定しなければ、デフォルトは 1 になります。

<!--
## Working with ReplicationControllers
-->
## ReplicationControllers と連携 {#working-with-replicationcontrollers}

<!--
### Deleting a ReplicationController and its Pods
-->
### ReplicationControllers と中にあるポッドを削除 {#deleting-a-replicationcontroller-and-its-pods}

<!--
To delete a ReplicationController and all its pods, use [`kubectl
delete`](/docs/reference/generated/kubectl/kubectl-commands#delete).  Kubectl will scale the ReplicationController to zero and wait
for it to delete each pod before deleting the ReplicationController itself.  If this kubectl
command is interrupted, it can be restarted.
-->
ReplicationController と中にあるポッドを削除するには、 [`kubectl delete`](/jp/docs/reference/generated/kubectl/kubectl-commands#delete) を使います。
Kubectl は ReplicationController で規模を０に変更できます。そして、ReplicationController 自身を削除する前に、各ポッドの削除のために待機します。
もしも kubectl コマンドが中断されれば、再起動できます。

<!--
When using the REST API or go client library, you need to do the steps explicitly (scale replicas to
0, wait for pod deletions, then delete the ReplicationController).
-->
REST API や go クライアント・ライブラリを使う場合は、このステップを明示的に行う必要があります（複製数を０にし、ポッドの削除まで待機し、それから ReplicationController を削除）。


<!--
### Deleting just a ReplicationController
-->
### ReplicationController のみ削除 {#deleting-just-a-replicationcontroller}

<!--
You can delete a ReplicationController without affecting any of its pods.
-->
あらゆるポッドに影響を与えず ReplicationController だけを削除出来ます。

<!--
Using kubectl, specify the `--cascade=false` option to [`kubectl delete`](/docs/reference/generated/kubectl/kubectl-commands#delete).
-->
kubectl を使い、[`kubectl delete`](/jp/docs/reference/generated/kubectl/kubectl-commands#delete) のオプションに `--cascade=false` を指定します。

<!--
When using the REST API or go client library, simply delete the ReplicationController object.
-->
REST API や go クライアント・ライブラリを使う場合は、単純に ReplicationController オブジェクトを削除するだけです。

<!--
Once the original is deleted, you can create a new ReplicationController to replace it.  As long
as the old and new `.spec.selector` are the same, then the new one will adopt the old pods.
However, it will not make any effort to make existing pods match a new, different pod template.
To update pods to a new spec in a controlled way, use a [rolling update](#rolling-updates).
-->
一度元からある（オリジナルの） ReplicationController  を削除すると、新しいものを作成して置き換え可能です。
新旧の `.spec.selector` が同じで有る限り、新しいポッドは古いポッドのものを受け入れます（採用します）。
しかしながら、異なったポッド・テンプレートを使わない限り、既存のポッドを新しいものと一致させるには大変ではありません。
ポッドを新しい spec に更新するには管理された手法 [ローリング・アップデート](#rolling-update) を使います。

<!--
### Isolating pods from a ReplicationController
-->
### ポッドを ReplicationController から独立（分離） {#isolating-pods-from-a-replicationcontroller}

<!--
Pods may be removed from a ReplicationController's target set by changing their labels. This technique may be used to remove pods from service for debugging, data recovery, etc. Pods that are removed in this way will be replaced automatically (assuming that the number of replicas is not also changed).
-->
ポッドを ReplicationController の対象から外したい時は、ポッドに割り当てられているラベルを変更します。
この手法はサービスのデバッグやデータ修復等のためにポッドを削除するために役立つでしょう。
ポッドをこの手法で削除すると、自動的に置き換えられます（前提として複製数も変更していない場合）。

<!--
## Common usage patterns
-->
## 共通の利用パターン {#common-usage-patterns}

<!--
### Rescheduling
-->
### 再スケジュール {#rescheduling}

<!--
As mentioned above, whether you have 1 pod you want to keep running, or 1000, a ReplicationController will ensure that the specified number of pods exists, even in the event of node failure or pod termination (for example, due to an action by another control agent).
-->
先ほど言及したように、１つのポッドを実行し続けたい場合、あるいは 1000 に増やしたい場合でも、ReplicationController は指定したポッド数が確実に存在するようにします。たとえ、ノード障害やポッドの終了が発生してもです（例えば、他のコントローラ・エージェントによる影響を受けてもです）。

<!--
### Scaling
-->
### 規模変更（スケーリング） {#scaling}

<!--
The ReplicationController makes it easy to scale the number of replicas up or down, either manually or by an auto-scaling control agent, by simply updating the `replicas` field.
-->
ReplicationController は複製数を増減して規模変更（スケール）をしやすいようにします。複製数の変更は手動だけでなく、オートスケーリング（自動規模調整）制御エージェントによっても行えます。その場合は単純に `replicas` （複製）フィールドを更新するだけです。

<!--
### Rolling updates
-->
### ローリング・アップデート（逐次更新） {#rolling-update}

<!--
The ReplicationController is designed to facilitate rolling updates to a service by replacing pods one-by-one.
-->
ReplicationController はサービスに対して１つ１つのポッドを置き換えるローリング・アップデート（逐次更新）を調整するために設計されています。

<!--
As explained in [#1353](http://issue.k8s.io/1353), the recommended approach is to create a new ReplicationController with 1 replica, scale the new (+1) and old (-1) controllers one by one, and then delete the old controller after it reaches 0 replicas. This predictably updates the set of pods regardless of unexpected failures.
-->
 [#1353](http://issue.k8s.io/1353) で説明があるように、推奨する手法は、新しい ReplicationController が１つの複製があるなら、新しいものをスケールし(+1)、それから古いコントローラの削除(-1)するのを１つ１つ行います。古いコントローラを削除後、複製数は０になります。
 この予測可能な更新は、ポッドの集まりは、予期せぬ障害が起こったとしても行えます。

<!--
Ideally, the rolling update controller would take application readiness into account, and would ensure that a sufficient number of pods were productively serving at any given time.
-->
理想的には、ローリング・アップデート（逐次更新）コントローラは、アプリケーションの読込性を報告しながら、常に十分な数のポッドを確実に維持します。

<!--
The two ReplicationControllers would need to create pods with at least one differentiating label, such as the image tag of the primary container of the pod, since it is typically image updates that motivate rolling updates.
-->
２つの ReplicationController でポッドを作成するには、少なくとも１つはラベルを変える必要があります。ラベルではポッドで使われる主なコンテナのイメージ・タグなどです。これは、イメージのローリング・アップデート（逐次更新）のために使うのが典型的です。

<!--
Rolling update is implemented in the client tool
[`kubectl rolling-update`](/docs/reference/generated/kubectl/kubectl-commands#rolling-update). Visit [`kubectl rolling-update` task](/docs/tasks/run-application/rolling-update-replication-controller/) for more concrete examples.
-->
ローリング・アップデート（逐次更新）はクライアント・ツール [`kubectl rolling-update`](/jp/docs/reference/generated/kubectl/kubectl-commands#rolling-update) で実装されています。
より明確な例は  [`kubectl rolling-update` タスク](/jp/docs/tasks/run-application/rolling-update-replication-controller/) をご覧ください。

<!--
### Multiple release tracks
-->
### 複数のリリース・トラック軌道（Multiple release tracks） {#multiple-release-tracks}

<!--
In addition to running multiple releases of an application while a rolling update is in progress, it's common to run multiple releases for an extended period of time, or even continuously, using multiple release tracks. The tracks would be differentiated by labels.
-->
アプリケーションの複数のリリースを提供するにあたって、ローリング・アップデート（逐次更新）を実行できるのに加えて、長期間にわたって複数のリリースを継続するのは一般的です。あるいは、継続的に複数のリリース・トラック（軌道）を使います。トラック（軌道）はラベルとは異なります。

<!--
For instance, a service might target all pods with `tier in (frontend), environment in (prod)`.  Now say you have 10 replicated pods that make up this tier.  But you want to be able to 'canary' a new version of this component.  You could set up a ReplicationController with `replicas` set to 9 for the bulk of the replicas, with labels `tier=frontend, environment=prod, track=stable`, and another ReplicationController with `replicas` set to 1 for the canary, with labels `tier=frontend, environment=prod, track=canary`.  Now the service is covering both the canary and non-canary pods.  But you can mess with the ReplicationControllers separately to test things out, monitor the results, etc.
-->
例として、サービスが全てのポッドが  `tier in (frontend), environment in (prod)` （「フロントエンド」層の「プロダクション」環境）と仮定します。
この層は10の複製されたポッドによって構成されています。
ここで、構成要素（コンポーネント）の新しいバージョン「canary」（カナリア）を有効にしたいとします。
ReplicationController を使って `replicas` （複製）の集まりから、 9 つあるラベルを `tier=frontend, environment=prod, track=stable` とします（層＝フロントエンド、環境＝プロダクション、トラック＝stable）に設定します。
そして、ReplicationController では `replicas` をカナリア用に 1 を設定し、ラベルは  `tier=frontend, environment=prod, track=canary` とします（層＝フロントエンド、環境＝プロダクション、トラック＝canary）。
これでサービスを canary と canary ではないポッドを扱います。
しかし ReplicationControllers を分けているため、混乱を引き起こす可能性があるため、テストを行い、結果の監視などが必要になります。

<!--
### Using ReplicationControllers with Services
-->
### ReplicationController をサービスに使う {#using-replicationcontrollers-with-services}

<!-
Multiple ReplicationControllers can sit behind a single service, so that, for example, some traffic
goes to the old version, and some goes to the new version.
-->
複数の ReplicationController を１つのサービスの後ろに添えられます。たとえば、トラフィックを古いバージョンに流しますが、いくつかを新しいバージョンに流すことが可能です。

<!--
A ReplicationController will never terminate on its own, but it isn't expected to be as long-lived as services. Services may be composed of pods controlled by multiple ReplicationControllers, and it is expected that many ReplicationControllers may be created and destroyed over the lifetime of a service (for instance, to perform an update of pods that run the service). Both services themselves and their clients should remain oblivious to the ReplicationControllers that maintain the pods of the services.
-->
ReplicationController は自分自身を決して停止させません。
しかし、長期間にわたってサービスとしての稼働は予想されていません。
サービスはポッドの集合であり、複数の ReplicationController によって管理されるものです。
そのため、多くの ReplicationController が作成され、サービスのライフタイムを越えると破棄される可能性があります（例えば、サービスを動かしながら、ポッドの性能をアップデートする場合）。
サービス自身とクライアントの両方が ReplicationController の存在を記憶しません。これはサービスのポッドを運用（維持：maintain）するためです。

<!--
## Writing programs for Replication
-->
## 複製（レプリケーション）用のプログラムを書く {#writing-programs-for-replication}

<!--
Pods created by a ReplicationController are intended to be fungible and semantically identical, though their configurations may become heterogeneous over time. This is an obvious fit for replicated stateless servers, but ReplicationControllers can also be used to maintain availability of master-elected, sharded, and worker-pool applications. Such applications should use dynamic work assignment mechanisms, such as the [RabbitMQ work queues](https://www.rabbitmq.com/tutorials/tutorial-two-python.html), as opposed to static/one-time customization of the configuration of each pod, which is considered an anti-pattern. Any pod customization performed, such as vertical auto-sizing of resources (for example, cpu or memory), should be performed by another online controller process, not unlike the ReplicationController itself.
-->
ReplicationController によって作成されるポッドとは、代替可能かつ意味的にはどちらも同一なものを意図していますが、時間が経てば経つほど、設定ファイルは異質なものになる場合があるでしょう。
これは複製されたステートレスなサーバの用途と明確に一致します。しかしまた、ReplicationController はマスタ選出（master-elected）、sharded（断片化）、ワーカーをプールするアプリケーションを運用するためにも利用できます。
このようなアプリケーションは動的な処理の割り当て機構を使うべきです。例えば [RabbitMQ ワーク・キュー（work queues）(https://www.rabbitmq.com/tutorials/tutorial-two-python.html) などです。
従って、ポッドに対して静的な／一度きりの設定のカスタマイズは行うべきではありません。これは、アンチ・パターンと考えられます。
あらゆるポッドに対するカスタマイズとは、リソースの水平オートスケーリング（例：CPU やメモリ）のような場合は、他のオンライン制御プロセスによって行われるべきであり、ReplicationController 自身では行われるべきではありません。

<!--
## Responsibilities of the ReplicationController
-->
## ReplicationController の責務 {#responsibilities-of-the-replicationcontroller}

<!--
The ReplicationController simply ensures that the desired number of pods matches its label selector and are operational. Currently, only terminated pods are excluded from its count. In the future, [readiness](http://issue.k8s.io/620) and other information available from the system may be taken into account, we may add more controls over the replacement policy, and we plan to emit events that could be used by external clients to implement arbitrarily sophisticated replacement and/or scale-down policies.
-->
ReplicationController がシンプルに保証するのは、ラベル・セレクタに一致する必要な数のポッドを準備し、それらが操作可能なことです。
現時点では、停止したポッドのみをカウント対象から除外できます。
将来的には  [readiness（準備）](http://issue.k8s.io/620)や他のシステムで利用可能な情報でカウントできる可能性があります。
そのために私たちは複製ポリシーを越えた制御を行うかもしれません。また私たちは外部のクライアントが使うためのイベント発行、これは独自に洗練されたものに置き換える、あるいは、ポリシーをスケールダウンできるようにするつもりです。

<!--
The ReplicationController is forever constrained to this narrow responsibility. It itself will not perform readiness nor liveness probes. Rather than performing auto-scaling, it is intended to be controlled by an external auto-scaler (as discussed in [#492](http://issue.k8s.io/492)), which would change its `replicas` field. We will not add scheduling policies (for example, [spreading](http://issue.k8s.io/367#issuecomment-48428019)) to the ReplicationController. Nor should it verify that the pods controlled match the currently specified template, as that would obstruct auto-sizing and other automated processes. Similarly, completion deadlines, ordering dependencies, configuration expansion, and other features belong elsewhere. We even plan to factor out the mechanism for bulk pod creation ([#170](http://issue.k8s.io/170)).
-->
ReplicationController が持つと考えられる責任範囲とは、永久に狭いままです。
自身では読込性診断や生存性診断を処理しません。
それよりもむしろ、オートスケーリングを処理するため、外部のオートスケーラによって `replicas` フィールドが変更可能な管理ができるように意図しています（[#492](http://issue.k8s.io/492)で議論中）。
私たちはスケジューリング・ポリシーを追加しないつもりです（たとえば、ReplicationController に対する [spreading（スプレッディング）](http://issue.k8s.io/367#issuecomment-48428019) ）。
ポッドであれば現在指定したテンプレートに一致するものしか制御しないどころか、オートサイジングや他の自動化プロセスに対する抽象化はありません。
同様に、コレクションのデッドライン、順番の依存性、設定情報の拡張、その他の機能が上げられます。
まっさらなポッドを作成時の要素や機構については、計画しています（([#170](http://issue.k8s.io/170)）。

<!--
The ReplicationController is intended to be a composable building-block primitive. We expect higher-level APIs and/or tools to be built on top of it and other complementary primitives for user convenience in the future. The "macro" operations currently supported by kubectl (run, scale, rolling-update) are proof-of-concept examples of this. For instance, we could imagine something like [Asgard](http://techblog.netflix.com/2012/06/asgard-web-based-cloud-management-and.html) managing ReplicationControllers, auto-scalers, services, scheduling policies, canaries, etc.
-->
ReplicationController によって構築ブロック・プリミティブを構成可能にするつもりです。
私たちは高レベルの API やツールがそれらの上で、あるいはユーザが役立つ他の補完的なプリミティブを将来的に導入する可能性があります
「マクロ」操作が現在サポートされているのは kubectl （run、scale、rolling-update）であり、これらの例は実証実験中（proof-of-concept）です。
たとえば、私たちは [Asgard](http://techblog.netflix.com/2012/06/asgard-web-based-cloud-management-and.html) のようなものが ReplicationControllers 、オートスケーラ、サービス、スケジューリング方針（ポリシー）、カナリア、等を管理するのを想像しています。

<!--
## API Object
-->
## API オブジェクト {#api-object}

<!--
Replication controller is a top-level resource in the Kubernetes REST API. More details about the
API object can be found at:
[ReplicationController API object](/docs/reference/generated/kubernetes-api/{{< param "version" >}}/#replicationcontroller-v1-core).
-->
レプリケーション・コントローラは Kubernetes REST API におけるトップ・レベルのリソースです。
API オブジェクトの詳細に関しては [ReplicationController API object](/jp/docs/reference/generated/kubernetes-api/{{< param "version" >}}/#replicationcontroller-v1-core) をご覧ください。

<!--
## Alternatives to ReplicationController
-->
## ReplicationController の代替 {#alternatives-to-replicationcontroller}

<!--
### ReplicaSet
-->
### ReplicaSet（レプリカセット）

<!--
[`ReplicaSet`](/docs/concepts/workloads/controllers/replicaset/) is the next-generation ReplicationController that supports the new [set-based label selector](/docs/concepts/overview/working-with-objects/labels/#set-based-requirement).
It’s mainly used by [`Deployment`](/docs/concepts/workloads/controllers/deployment/) as a mechanism to orchestrate pod creation, deletion and updates.
Note that we recommend using Deployments instead of directly using Replica Sets, unless you require custom update orchestration or don’t require updates at all.
-->
[`ReplicaSet`](/jp/docs/concepts/workloads/controllers/replicaset/) は次世代の ReplicationController であり、新しい[set-based label selector（セットに基づくラベル・セレクタ）](/jp/docs/concepts/overview/working-with-objects/labels/#set-based-requirement)をサポートします。
ポッドの作成、削除、更新を調整（オーケストレート）する仕組みとして主に使われるのは [`Deployment`](/jp/docs/concepts/workloads/controllers/deployment/) です。

<!--
### Deployment (Recommended)
-->
### Deployment（デプロイメント）（推奨） {#deployment-recommended}

<!--
[`Deployment`](/docs/concepts/workloads/controllers/deployment/) is a higher-level API object that updates its underlying Replica Sets and their Pods
in a similar fashion as `kubectl rolling-update`. Deployments are recommended if you want this rolling update functionality,
because unlike `kubectl rolling-update`, they are declarative, server-side, and have additional features.
-->
[`Deployment`](/jp/docs/concepts/workloads/controllers/deployment/) は高レベルの API オブジェクトであり、根底にあるのはレプリカ・セットとそれらのポッドであり、これらが `kubectl rolling-update` と似たような手法を提供します。
ローリング・アップデート（逐次更新）機能を必要とする場合、Deployment が推奨されています。
これは `kubectl rolling-update` と同様に、これらは宣言型であり、サーバ・サイドであり、追加機能を備えるからです。

<!--
### Bare Pods
-->
### Bare Pods（ベア・ポッド） {#barepods}

<!--
Unlike in the case where a user directly created pods, a ReplicationController replaces pods that are deleted or terminated for any reason, such as in the case of node failure or disruptive node maintenance, such as a kernel upgrade. For this reason, we recommend that you use a ReplicationController even if your application requires only a single pod. Think of it similarly to a process supervisor, only it supervises multiple pods across multiple nodes instead of individual processes on a single node.  A ReplicationController delegates local container restarts to some agent on the node (for example, Kubelet or Docker).
-->


<!--
### Job
-->
### Job（ジョブ） {#job}

<!--
Use a [`Job`](/docs/concepts/jobs/run-to-completion-finite-workloads/) instead of a ReplicationController for pods that are expected to terminate on their own
(that is, batch jobs).
-->
ポッドに対して ReplicationController の代わりに  [`Job`](/docs/concepts/jobs/run-to-completion-finite-workloads/)  を使う方法は、自身で終了する挙動が期待されます（つまり、バッチ・ジョブです）。

<!--
### DaemonSet
-->
### DamonSet（デーモンセット） {#daemonset}

<!--
Use a [`DaemonSet`](/docs/concepts/workloads/controllers/daemonset/) instead of a ReplicationController for pods that provide a
machine-level function, such as machine monitoring or machine logging.  These pods have a lifetime that is tied
to a machine lifetime: the pod needs to be running on the machine before other pods start, and are
safe to terminate when the machine is otherwise ready to be rebooted/shutdown.
-->
ポッドに対して ReplicationController の代わりに [`DaemonSet`](/docs/concepts/workloads/controllers/daemonset/) を使う方法は、マシン・レベルの機能、たとえばマシン監視やマシンのログ記録を提供します。
各ポッドはマシンのライフタイムに強く結びつくライフタイム（生存期間）を持ちます。つまり、ポッドに必要なのはマシン上で他のポッドの起動前に実行することであり、マシンが再起動またはシャットダウンなど利用できなくなる前に、安全に停止できるようにします。

<!--
## For more information
-->
## 詳しい情報 {#for-more-information}

<!--
Read [Run Stateless AP Replication Controller](/docs/tutorials/stateless-application/run-stateless-ap-replication-controller/).
-->
[ステートレス API レプリケーション・コントローラの実行](/jp/docs/tutorials/stateless-application/run-stateless-ap-replication-controller/) をご覧ください。


{{% /capture %}}


