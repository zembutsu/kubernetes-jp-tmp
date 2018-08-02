---
reviewers:
- erictune
title: 初期化コンテナ
content_template: templates/concept
weight: 40
---

{{% capture overview %}}
<!--
This page provides an overview of Init Containers, which are specialized
Containers that run before app Containers and can contain utilities or setup
scripts not present in an app image.
-->
このページは初期化コンテナ（Init コンテナ）の概要を紹介します。初期化コンテナは特殊なコンテナであり、アプリケーション・コンテナを実行する前に実行し、アプリケーション・イメージに存在しないユーティリティやセットアップ・スクリプトも含められます。
{{% /capture %}}

{{< toc >}}

<!--
This feature has exited beta in 1.6. Init Containers can be specified in the PodSpec
alongside the app `containers` array. The beta annotation value will still be respected
and overrides the PodSpec field value, however, they are deprecated in 1.6 and 1.7.
In 1.8, the annotations are no longer supported and must be converted to the PodSpec field.
-->
この機能は 1.6 でベータが外れました。初期化コンテナ（Init Container）は PodSpec にある app `containers` アレイと並んで指定できます。ベータの注釈値（beta annotation value）が尊重されるため、 PodSpec フィールドの値を上書きします。しかしながら、1.6 と 1.7 で廃止されました。1.8 では注釈でサポートされないため、PodSpec フィールドでカバーする必要があります。

{{% capture body %}}
<!--
## Understanding Init Containers
-->
## 初期化コンテナの理解 {#understanding-init-containers}

<!--
A [Pod](/docs/concepts/workloads/pods/pod-overview/) can have multiple Containers running
apps within it, but it can also have one or more Init Containers, which are run
before the app Containers are started.
-->

[ポッド](/jp/docs/concepts/workloads/pods/pod-overview/) はポッドの中でアプリケーションを実行する複数のコンテナを持てます。また、これに加えて１つまたは複数の初期化コンテナ（Init Container）も持てます。これは、アプリケーション・コンテナを起動する前に実行するコンテナです。

<!--
Init Containers are exactly like regular Containers, except:
-->
初期化コンテナは、厳密には通常のコンテナと同じですが、以下の手が異なります：

<!--
* They always run to completion.
* Each one must complete successfully before the next one is started.
-->
* コンテナの実行は常に完了する。
* 次のコンテナを開始する前に、正常に完了している必要がある。

<!--
If an Init Container fails for a Pod, Kubernetes restarts the Pod repeatedly until the Init
Container succeeds. However, if the Pod has a `restartPolicy` of Never, it is not restarted.
-->
ポッド用の初期化コントローラで障害が起きれば、Kubernetes は Init コンテナが成功するまでポッドの再起動を繰り返します。しかしながら、ポッドの `restartPolicy` （再起動方針）が Never（再起動しない）であれば、再起動を行いミズ園。

<!--
To specify a Container as an Init Container, add the `initContainers` field on the PodSpec as
a JSON array of objects of type
[Container](/docs/reference/generated/kubernetes-api/{{< param "version" >}}/#container-v1-core)
alongside the app `containers` array.
The status of the init containers is returned in `.status.initContainerStatuses`
field as an array of the container statuses (similar to the `.status.containerStatuses`
field).
-->
初期化コンテナとしてコンテナを指定するには、PodSpec に app `containers` アレイと並んで `initContainers` フィールドを追加し、  JSON アレイのオブジェクト・タイプとして記述します。初期化コンテナが返すステータスは、コンテナ・ステータスの配列として `.status.initContainerStatus` にあります（ `.status.containerStatuses` フィールドと似ています ）。

<!--
### Differences from regular Containers
-->
### 通常のコンテナとの違い {#differences-from-regular-containers}

<!--
Init Containers support all the fields and features of app Containers,
including resource limits, volumes, and security settings. However, the
resource requests and limits for an Init Container are handled slightly
differently, which are documented in [Resources](#resources) below.  Also, Init Containers do not
support readiness probes because they must run to completion before the Pod can
be ready.
-->
初期化コンテナはアプリケーション・コンテナの全てのフィールドと機能をサポートします。ここにはリソース上限、ボリューム、セキュリティ設定も含まれます。しかしながら、初期化コンテナに対するリソース要求と上限の取り扱いは極めて異なります。違いは以下の [リソース](#resources) にドキュメント化しています。また、初期化コンテナは読込性診断をサポートしません。なぜなら、ポッドが利用可能になるのに先立って、初期化コンテナの実行を完了している必要があるからです。

<!--
If multiple Init Containers are specified for a Pod, those Containers are run
one at a time in sequential order. Each must succeed before the next can run.
When all of the Init Containers have run to completion, Kubernetes initializes
the Pod and runs the application Containers as usual.
-->
ポッドに対して複数の初期化コンテナを指定すると、各コンテナは連続した順番で一度だけ実行されます。次のコンテナを実行する前に、各コンテナは処理が完了している必要があります。全ての初期化コンテナの実行が完了すると、kubernetes はポッドを初期化し、津城通にアプリケーション・コンテナを実行します。


<!--
## what can init containers be used for?
-->
## 初期化コンテナは何のために使えますか？ {#what-can-init-containers-be-used-for}

<!--
Because Init Containers have separate images from app Containers, they
have some advantages for start-up related code:
-->
初期化コンテナはアプリケーション・コンテナとは分離できるため、起動処理に関連するコードのために役立ちます：

<!--
* They can contain and run utilities that are not desirable to include in the
  app Container image for security reasons.
* They can contain utilities or custom code for setup that is not present in an app
  image. For example, there is no need to make an image `FROM` another image just to use a tool like
  `sed`, `awk`, `python`, or `dig` during setup.
* The application image builder and deployer roles can work independently without
  the need to jointly build a single app image.
* They use Linux namespaces so that they have different filesystem views from app Containers.
  Consequently, they can be given access to Secrets that app Containers are not able to
  access.
* They run to completion before any app Containers start, whereas app
  Containers run in parallel, so Init Containers provide an easy way to block or
  delay the startup of app Containers until some set of preconditions are met.
-->
* セキュリティ上の理由により、アプリケーション・コンテナ・イメージ含められないユーティリティを、初期化コンテナに含めて実行できます。
* セットアップのためにユーティリティやカスタム・コードを含められます。例えば、イメージに `FROM` で作成するにあたり、他のイメージに含む必要の無いツール、例えば `sed`、 `awk`、 `python`、 `dig` をセットアップ中に扱えます
* アプリケーション・イメージ構築担当者や開発担当者が、１つのアプリケーション・イメージを一緒に構築する必要が無く、独立して作業できます。
* Linux 名前空間を使うため、アプリケーション・コンテナごとに異なるファイルシステムを持てます。そのため、アプリケーション・コンテナがアクセスできないシークレットにアクセス可能にもできます。
* あらゆるアプリケーション・コンテナを実行する前に実行を完了する必要があるのに対して、アプリケーション・コンテナは並列に実行できます。そのため、アプリケーション・コンテナが起動するまで、何らかの状態に一致するまでブロック（待機）したり遅延するのを簡単にするために、初期化コンテナを使えます。

<!--
### Examples
-->
### 例 {#examples}

<!--
Here are some ideas for how to use Init Containers:
-->
ここにあるのは初期化コンテナをどう使うかのアイディアです：

<!--
* Wait for a service to be created with a shell command like:

      for i in {1..100}; do sleep 1; if dig myservice; then exit 0; fi; done; exit 1

* Register this Pod with a remote server from the downward API with a command like:

      curl -X POST http://$MANAGEMENT_SERVICE_HOST:$MANAGEMENT_SERVICE_PORT/register -d 'instance=$(<POD_NAME>)&ip=$(<POD_IP>)'

* Wait for some time before starting the app Container with a command like `sleep 60`.
* Clone a git repository into a volume.
* Place values into a configuration file and run a template tool to dynamically
  generate a configuration file for the main app Container. For example,
  place the POD_IP value in a configuration and generate the main app
  configuration file using Jinja.
-->
* 次のようなシェル・コマンドを使い、サービスが作成されるのを待ちます：

      for i in {1..100}; do sleep 1; if dig myservice; then exit 0; fi; done; exit 1

* 以下にある API のコマンドを使い、対象のポッドをリモート・サーバ上に登録します：

      curl -X POST http://$MANAGEMENT_SERVICE_HOST:$MANAGEMENT_SERVICE_PORT/register -d 'instance=$(<POD_NAME>)&ip=$(<POD_IP>)'

* アプリケーション・コンテナが起動する前に、 `sleep 60` のようなコマンドを使って待機します。
* ボリューム内に git リポジトリをクローンします。
* メインのアプリケーション・コンテナ用の設定ファイルを動的に生成するような、テンプレート・ツールの設定ファイルに値を入れ、実行します。たとえば、設定ファイルに POD_IP の値を入れた設定ファイルを準備しておき、メインのアプリケーション用が使う Jinja 用のファイルを生成します。

<!--
More detailed usage examples can be found in the [StatefulSets documentation](/docs/concepts/workloads/controllers/statefulset/)
and the [Production Pods guide](/docs/tasks/configure-pod-container/configure-pod-initialization/).
-->
より詳しいサンプルの使い方は、[StatefulSets ドキュメント](/jp/docs/concepts/workloads/controllers/statefulset/)
と [プロダクション向けのポッド・ガイド](/jp/docs/tasks/configure-pod-container/configure-pod-initialization/) にあります。

<!--
### Init Containers in use
-->
### 初期化コンテナ使用例 {init-containers-in-use}

<!--
the following yaml file for kubernetes 1.5 outlines a simple Pod which has two Init Containers.
The first waits for `myservice` and the second waits for `mydb`. Once both
containers complete, the Pod will begin.
-->
以下の YAML ファイルは Kubernetes 1.5 用にまとめたシンプルなポッドであり、２つの初期化コンテナがあります。まずは `myservice` を待機し、次に `mydb` の処理を待ちます。どちらのコンテナも完了したら、ポッドを介します。

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: myapp-pod
  labels:
    app: myapp
  annotations:
    pod.beta.kubernetes.io/init-containers: '[
        {
            "name": "init-myservice",
            "image": "busybox",
            "command": ["sh", "-c", "until nslookup myservice; do echo waiting for myservice; sleep 2; done;"]
        },
        {
            "name": "init-mydb",
            "image": "busybox",
            "command": ["sh", "-c", "until nslookup mydb; do echo waiting for mydb; sleep 2; done;"]
        }
    ]'
spec:
  containers:
  - name: myapp-container
    image: busybox
    command: ['sh', '-c', 'echo The app is running! && sleep 3600']
```

<!--
There is a new syntax in Kubernetes 1.6, although the old annotation syntax still works for 1.6 and 1.7.  The new syntax must be used for 1.8 or greater. We have moved the declaration of Init Containers to `spec`:
-->
こちらは Kubernetes 1.6 に対応した新しい構文です。以前の古い構文も 1.6 と 1.7 で利用できます。新しい構文を利用できるのは 1.8 以上です。初期化コンテナの宣言は `spec` に移動しました：

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
    command: ['sh', '-c', 'echo The app is running! && sleep 3600']
  initContainers:
  - name: init-myservice
    image: busybox
    command: ['sh', '-c', 'until nslookup myservice; do echo waiting for myservice; sleep 2; done;']
  - name: init-mydb
    image: busybox
    command: ['sh', '-c', 'until nslookup mydb; do echo waiting for mydb; sleep 2; done;']
```

<!--
1.5 syntax still works on 1.6, but we recommend using 1.6 syntax. In Kubernetes 1.6, Init Containers were made a field in the API. The beta annotation is still respected in 1.6 and 1.7, but is not supported in 1.8 or greater.
-->
1.5 構文は 1.6 でも動作しますが、1.6 構文の仕様を推奨します。また、Kubernetes 1.6 では、初期化コンテナは API のフィールドでも作成できます。ベータ・アノテーションは 1.6 と 1.7 までは尊重されていますが、1.8 以上ではサポートされていません。

<!--
Yaml file below outlines the `mydb` and `myservice` services:
-->
以下の YAML ファイルは、 `mydb` と `myservice` サービスのアウトラインです：

```yaml
kind: Service
apiVersion: v1
metadata:
  name: myservice
spec:
  ports:
  - protocol: TCP
    port: 80
    targetPort: 9376
---
kind: Service
apiVersion: v1
metadata:
  name: mydb
spec:
  ports:
  - protocol: TCP
    port: 80
    targetPort: 9377
```

<!--
This Pod can be started and debugged with the following commands:
-->
このポッドは以下のコマンドで開始およびデバッグできます：

```shell
$ kubectl create -f myapp.yaml
pod "myapp-pod" created
$ kubectl get -f myapp.yaml
NAME        READY     STATUS     RESTARTS   AGE
myapp-pod   0/1       Init:0/2   0          6m
$ kubectl describe -f myapp.yaml
Name:          myapp-pod
Namespace:     default
[...]
Labels:        app=myapp
Status:        Pending
[...]
Init Containers:
  init-myservice:
[...]
    State:         Running
[...]
  init-mydb:
[...]
    State:         Waiting
      Reason:      PodInitializing
    Ready:         False
[...]
Containers:
  myapp-container:
[...]
    State:         Waiting
      Reason:      PodInitializing
    Ready:         False
[...]
Events:
  FirstSeen    LastSeen    Count    From                      SubObjectPath                           Type          Reason        Message
  ---------    --------    -----    ----                      -------------                           --------      ------        -------
  16s          16s         1        {default-scheduler }                                              Normal        Scheduled     Successfully assigned myapp-pod to 172.17.4.201
  16s          16s         1        {kubelet 172.17.4.201}    spec.initContainers{init-myservice}     Normal        Pulling       pulling image "busybox"
  13s          13s         1        {kubelet 172.17.4.201}    spec.initContainers{init-myservice}     Normal        Pulled        Successfully pulled image "busybox"
  13s          13s         1        {kubelet 172.17.4.201}    spec.initContainers{init-myservice}     Normal        Created       Created container with docker id 5ced34a04634; Security:[seccomp=unconfined]
  13s          13s         1        {kubelet 172.17.4.201}    spec.initContainers{init-myservice}     Normal        Started       Started container with docker id 5ced34a04634
$ kubectl logs myapp-pod -c init-myservice # Inspect the first init container
$ kubectl logs myapp-pod -c init-mydb      # Inspect the second init container
```

<!--
Once we start the `mydb` and `myservice` services, we can see the Init Containers
complete and the `myapp-pod` is created:
-->
`mydb` と `myservice` サービスを開始すると、初期化コンテナの完了後に `myapp-pod` が作成されます：

```shell
$ kubectl create -f services.yaml
service "myservice" created
service "mydb" created
$ kubectl get -f myapp.yaml
NAME        READY     STATUS    RESTARTS   AGE
myapp-pod   1/1       Running   0          9m
```

<!--
This example is very simple but should provide some inspiration for you to
create your own Init Containers.
-->
この例は非常にシンプルですが、皆さん自身が初期化コンテナを作成する時に参考となるでしょう。

<!--
## Detailed behavior
-->
## 詳細な挙動 {#detailed-behaivor}

<!--
During the startup of a Pod, the Init Containers are started in order, after the
network and volumes are initialized. Each Container must exit successfully before
the next is started. If a Container fails to start due to the runtime or
exits with failure, it is retried according to the Pod `restartPolicy`. However,
if the Pod `restartPolicy` is set to Always, the Init Containers use
`RestartPolicy` OnFailure.
-->
ポッドの起動中、初期化コンテナは順番通りに起動した後、ネットワークとボリュームが初期化されます。各コンテナを開始するには、前のコンテナは実行を完了している必要があります。コンテナのランタイムが原因か障害による終了で起動できあんければ、ポッドの `restartPolicy` （再起動方針）に従ってポッドの再起動を試みます。しかしながら、ポッドの `restartPolicy` が `Always`（常時）に設定されていると、初期化コンテナは `RestartPolicy` （再起動方針）を OnFailure（障害時のみ）として扱います。

<!--
A Pod cannot be `Ready` until all Init Containers have succeeded. The ports on an
Init Container are not aggregated under a service. A Pod that is initializing
is in the `Pending` state but should have a condition `Initializing` set to true.
-->
全ての初期化コンテナが成功するまで、ポッドは `Ready` （待機）になりません。初期化コンテナのポートはサービスと関連付けられていません。ポッドの初期化中は `Pending`（待機中）の状態ですが、 `Initilizing` （初期化）の状態も true となるべきでしょう。

<!--
If the Pod is [restarted](#pod-restart-reasons), all Init Containers must
execute again.
-->
ポッドが [再起動したら](#pod-restart-reasons)、全ての初期化コンテナを再実行する必要があります。

<!--
Changes to the Init Container spec are limited to the container image field.
Altering an Init Container image field is equivalent to restarting the Pod.
-->
初期化コンテナの spec 変更は、コンテナ・イメージ・フィールドのみに制限されています。初期化コンテナのイメージ・フィールドを反映するには、ポッドの再起動をしますのでご注意ください。

<!--
Because Init Containers can be restarted, retried, or re-executed, Init Container
code should be idempotent. In particular, code that writes to files on `EmptyDirs`
should be prepared for the possibility that an output file already exists.
-->
全ての初期化コンテナは再起動され、リトライされ、再実行されうるので、初期化コンテナのコードは冪等（idempotent）であるべきです。特に、 `EnptyDirs` 上にフィアルを書き出す場合は、出力するファイルが既に存在しうるものとして準備すべきでしょう。

<!--
Init Containers have all of the fields of an app Container. However, Kubernetes
prohibits `readinessProbe` from being used because Init Containers cannot
define readiness distinct from completion. This is enforced during validation.
-->
初期化コンテナが持つ全てのフィールドは、アプリケーション・コンテナ上にあります。しかしながら、Kubernetes は `readinessProbe` が使われるのを禁止します。これは初期化コンテナが読込性を定義しても完了するかどうか確認できないからです。この整合性（validation）の確認が必ず行われます。

<!--
Use `activeDeadlineSeconds` on the Pod and `livenessProbe` on the Container to
prevent Init Containers from failing forever. The active deadline includes Init
Containers.
-->
ポッド上の `activeDeadlineSeconds`（アクティブ期限秒） とコンテナ上の `livenessProve` （生存性診断）によって、初期化コンテナの障害継続を防止します。アクティブ期限（デッドライン）には初期化コンテナも含みます。

<!--
The name of each app and Init Container in a Pod must be unique; a
validation error is thrown for any Container sharing a name with another.
-->
ポッド内の各アプリケーション・コンテナおよび初期化コンテナの名前は、ユニークである必要があります。つまり、あらゆるコンテナが他と名前の重複があると、整合性のエラーとなります。

<!--
### Resources
-->
### リソース {#resources}

<!--
Given the ordering and execution for Init Containers, the following rules
for resource usage apply:
-->
初期化コンテナの並べ替えの指定と実行には、以下のルールに従ってリソース使用が適用されます：

<!--
* The highest of any particular resource request or limit defined on all Init
  Containers is the *effective init request/limit*
* The Pod's *effective request/limit* for a resource is the higher of:
  * the sum of all app Containers request/limit for a resource
  * the effective init request/limit for a resource
* Scheduling is done based on effective requests/limits, which means
  Init Containers can reserve resources for initialization that are not used
  during the life of the Pod.
* QoS tier of the Pod's *effective QoS tier* is the QoS tier for Init Containers
  and app containers alike.
-->
* 何らかの最も高いリソース要求か、全ての初期化コンテナで定義されている上限が、 *効率的な初期化要求/上限（effective init request/limit）* です
* ポッドのリソースに対する効率的な初期化要求/上限は、以下どちらかの高い方です：
  * 全てのアプリケーション・コンテナのリソース要求/上限の合計
  * リソースに対する効率的な初期化要求/上限
* 効率的な要求/上限に基づきスケジューリングが実施されます。つまり、初期化コンテナは初期化のためにリソースを予約できます。これはポッドの稼働中には使われないものも含みます。
* ポッドの効率的な QoS ティア（tier）は、初期化コンテナとアプリケーション・コンテナに対する QoS ティアと同じです。

<!--
Quota and limits are applied based on the effective Pod request and
limit.
-->
クォータ（容量制限）と上限は、効率的なポッド要求と上限に基づいて適用されます。

<!--
Pod level cgroups are based on the effective Pod request and limit, the
same as the scheduler.
-->
ポッド・レベルの cgroup はスケジューラと同様に、効率的なポッド要求と上限に基づいて適用されます。

<!--
### Pod restart reasons
-->
### ポッド再起動条件 {#pod-restart-reasons}

<!--
A Pod can restart, causing re-execution of Init Containers, for the following
reasons:
-->
以下の条件に従って、初期化コンテナの再実行によるポッドは再起動（restart）が発生します；

<!--
* A user updates the PodSpec causing the Init Container image to change.
  App Container image changes only restart the app Container.
* The Pod infrastructure container is restarted. This is uncommon and would
  have to be done by someone with root access to nodes.
* All containers in a Pod are terminated while `restartPolicy` is set to Always,
  forcing a restart, and the Init Container completion record has been lost due
  to garbage collection.
-->
* ユーザの PodSpec 更新は、初期化コンテナ・イメージの変更を引き起こします。アプリケーション・コンテナ・イメージの変更は、アプリケーション・コンテナのみ再起動します。
* ポッドの基盤（infrastructure）コンテナの再起動です。これは一般的では無く、ノードに対する何らかの root アクセスによる処理が行われた場合です。
* ポッド内の全てのコンテナが `restartPolicy` （再起動方針）が Always （常に）に設定された場合、再起動は強制され、ガベージ・コレクション（不要データの削除処理）によって初期化コンテナ管理の記録は破棄されます。

<!--
## Support and compatibility
-->
## サポートと互換性 {#support-and-compatibility}

<!--
A cluster with Apiserver version 1.6.0 or greater supports Init Containers
using the `.spec.initContainers` field. Previous versions support Init Containers
using the alpha or beta annotations. The `.spec.initContainers` field is also mirrored
into alpha and beta annotations so that Kubelets version 1.3.0 or greater can execute
Init Containers, and so that a version 1.6 apiserver can safely be rolled back to version
1.5.x without losing Init Container functionality for existing created pods.
-->
Apiserver バージョン 1.6.0 以上のクラスタは `.spec.initContainers` フィールドで初期化コンテナの使用をサポートします。それよりも以前のバージョンでは、初期化コンテナの使用にはアルファまたはベータの注記がありました。また、 `.spec.initContainers` フィールドはアルファおよびベータ注記を反映していますので、Kubernetes 1.3.0 以上では初期化コンテナを実行できます。また、バージョン 1.6 以上の apiserver は安全にバージョン 1.5.x にロールバックでき、そのときに作成済みのポッドに対する初期化コンテナの機能性は失われません。

<!--
In Apiserver and Kubelet versions 1.8.0 or greater, support for the alpha and beta annotations
is removed, requiring a conversion from the deprecated annotations to the
`.spec.initContainers` field.
-->
Kubernetes バージョン 1.8.0 以上の Apiserver では、アルファおよびベータ注記のサポートが削除されました。 `.spec.initContainers` フィールドにおける重複注記も削除されています。

{{% /capture %}}


{{% capture whatsnext %}}
<!--
* [Creating a Pod that has an Init Container](/docs/tasks/configure-pod-container/configure-pod-initialization/#creating-a-pod-that-has-an-init-container)
-->
* [初期化コンテナがあるポッドを作成](/jp/docs/tasks/configure-pod-container/configure-pod-initialization/#creating-a-pod-that-has-an-init-container)

{{% /capture %}}



