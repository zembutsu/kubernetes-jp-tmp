---
reviewers:
- mikedanese
title: 設定情報のベスト・プラクティス
content_template: templates/concept
weight: 10
---

{{% capture overview %}}
<!--
This document highlights and consolidates configuration best practices that are introduced throughout the user guide, Getting Started documentation, and examples.
-->
このドキュメントはユーザガイド、導入ドキュメント、サンプルを通して、
主要部分（ハイライト）かつ整理された設定情報のベスト・プラクティス（成功事例）です。

<!--
This is a living document. If you think of something that is not on this list but might be useful to others, please don't hesitate to file an issue or submit a PR.
-->
これは生きている（ライブ）ドキュメントです。
皆さんの考えていることが一覧になくても、他人に役立つ情報であれば、ファイルを issue や PR に遠慮なく送信ください。


{{% /capture %}}

{{% capture body %}}
<!--
## General Configuration Tips
-->
## 一般的な設定情報のヒント {#general-configuration-tips}

<!--
- When defining configurations, specify the latest stable API version.

- Configuration files should be stored in version control before being pushed to the cluster. This allows you to quickly roll back a configuration change if necessary. It also aids cluster re-creation and restoration.

- Write your configuration files using YAML rather than JSON. Though these formats can be used interchangeably in almost all scenarios, YAML tends to be more user-friendly.

- Group related objects into a single file whenever it makes sense. One file is often easier to manage than several. See the [guestbook-all-in-one.yaml](https://github.com/kubernetes/examples/tree/{{< param "githubbranch" >}}/guestbook/all-in-one/guestbook-all-in-one.yaml) file as an example of this syntax.

- Note also that many `kubectl` commands can be called on a directory. For example, you can call `kubectl create` on a directory of config files.

- Don't specify default values unnecessarily: simple, minimal configuration will make errors less likely.

- Put object descriptions in annotations, to allow better introspection.
-->
- 設定情報（configuration）を定義する時は、最新の安定 API バージョンを指定します。

- 設定情報ファイルは、クラスタに送信する前にバージョン管理に保管すべきでしょう。そうしておけば、設定情報の変更が必要になっても、素早く設定の巻き戻し（ロールバック）が可能になります。また、クラスタの再作成や修復の補助にもなります。

- 設定情報ファイルの記述は JSON ではなく YAML を使います。これらのフォーマット（書式）には大部分で互換性がありますが、YAML のほうがより利用者に親切（ユーザ・フレンドリー）です。

- グループに関連するオブジェクト１つのファイルに入れておくのが、常に道理にかないます。複数のファイルを管理するよりも１つのほうが常に簡単です。[guestbook-all-in-one.yaml](https://github.com/kubernetes/examples/tree/{{< param "githubbranch" >}}/guestbook/all-in-one/guestbook-all-in-one.yaml) 

- 多くの `kubectl` コマンドをディレクトリ上で呼び出せます。例えば、設定情報ファイルのあるディレクトリで `kubectl create` を呼び出せます。

- 必要がなければデフォルトの値を指定しないでください。つまり、シンプルで最小な設定情報によって、エラーを起こしづらくします。

- 見直しをしやすくするために（introspection）、アノテーションにオブジェクトの説明を入れます。

<!--
## "Naked" Pods vs ReplicaSets, Deployments, and Jobs
-->
## 「無防備」なポッド vs ReplicaSet、Deployment、Job {#naked-pods-vs-replicasets-deloyments-and-jobs}

<!--
- Don't use naked Pods (that is, Pods not bound to a [ReplicaSet](/docs/concepts/workloads/controllers/replicaset/) or [Deployment](/docs/concepts/workloads/controllers/deployment/)) if you can avoid it. Naked Pods will not be rescheduled in the event of a node failure.
-->
- 回避可能であれば、無防備なポッド（naked Pod）を使わないでください（つまり、ポッドは  [ReplicaSet](/jp/docs/concepts/workloads/controllers/replicaset/) や [Deployment](/jp/docs/concepts/workloads/controllers/deployment/) に拘束されていません）。無防備なポッドはノード障害のイベントがあっても再スケジュールされません。

<!--
  A Deployment, which both creates a ReplicaSet to ensure that the desired number of Pods is always available, and specifies a strategy to replace Pods (such as [RollingUpdate](/docs/concepts/workloads/controllers/deployment/#rolling-update-deployment)), is almost always preferable to creating Pods directly, except for some explicit [`restartPolicy: Never`](/docs/concepts/workloads/pods/pod-lifecycle/#restart-policy) scenarios. A [Job](/docs/concepts/workloads/controllers/jobs-run-to-completion/) may also be appropriate.
-->
Deployment では、ReplicaSet を作成して常に希望するポッド数を使えるようにするのと、ポッドの置き換え方針（ストラテジ）指定（[逐次更新（ローリング・アップデート）](/docs/concepts/workloads/controllers/deployment/#rolling-update-deployment) など）の両方において、[`restartPolicy: Never`](/docs/concepts/workloads/pods/pod-lifecycle/#restart-policy) を明示するとき以外は、ほとんど常に望ましいのがポッドの直接作成です。また、[ジョブ](/jp/docs/concepts/workloads/controllers/jobs-run-to-completion/) も適切な場合があるでしょう。

<!--
## Services
-->
## サービス  {#services}

<!--
- Create a [Service](/docs/concepts/services-networking/service/) before its corresponding backend workloads (Deployments or ReplicaSets), and before any workloads that need to access it. When Kubernetes starts a container, it provides environment variables pointing to all the Services which were running when the container was started. For example, if a Service named `foo` exists, all containers will get the following variables in their initial environment:
-->
- [サービス](/jp/docs/concepts/services-networking/service/) の作成前に、これに対応するバックエンド・ワークロード（Deployment や ReplicaSets）と、あらゆる事前のワークロードを接続する必要があります。Kubernetes がコンテナを起動すると、環境変数はコンテナが開始されて実行中の全サービスを示すようになります。たとえば、サービス名 `foo` が存在すると、全てのコンテナが各々の初期環境で以下の変数を得ます：

<!--
  ```shell
  FOO_SERVICE_HOST=<the host the Service is running on>
  FOO_SERVICE_PORT=<the port the Service is running on>
  ```
-->
  ```shell
  FOO_SERVICE_HOST=<サービスを実行中のホスト>
  FOO_SERVICE_PORT=<サービスを実行中のポート>
  ```
<!--
  If you are writing code that talks to a Service, don't use these environment variables; use the [DNS name of the Service](/docs/concepts/services-networking/dns-pod-service/) instead. Service environment variables are provided only for older software which can't be modified to use DNS lookups, and are a much less flexible way of accessing Services.
-->
もしもコードを書くのであれば、サービスとの通信には、これら環境変数を使わないでください。そのかわりに [サービスの DNS 名](/jp/docs/concepts/services-networking/dns-pod-service/) を使います。サービス環境変数が提供されるのは古いソフトウェアのみです。古いソフトウェアは DNS を調べて（lookup）も変更できませんし、サービスにアクセスする手法と比べても柔軟性がありません。

<!--
- Don't specify a `hostPort` for a Pod unless it is absolutely necessary. When you bind a Pod to a `hostPort`, it limits the number of places the Pod can be scheduled, because each <`hostIP`, `hostPort`, `protocol`> combination must be unique. If you don't specify the `hostIP` and `protocol` explicitly, Kubernetes will use `0.0.0.0` as the default `hostIP` and `TCP` as the default `protocol`.
-->
-  絶対的な必要性がなければ `hostPort` をポッドに対して指定しないでください。ポッドに対して `hostPort` を割り当てると、ポッドのスケジュールされる数が制限されます。理由は、それぞれの <`ホストIP`, `ホストのポート番号`, `プロトコル`> の組み合わせがユニークである必要があるためです。 `ホストIP` と `プロトコル` を明示して指定しなければ、Kubernetes は `0.0.0.0` をデフォルトの `ホストIP` とし、 `TCP` をデフォルトの `プロトコル` にします。

<!--
  If you only need access to the port for debugging purposes, you can use the [apiserver proxy](/docs/tasks/access-application-cluster/access-cluster/#manually-constructing-apiserver-proxy-urls) or [`kubectl port-forward`](/docs/tasks/access-application-cluster/port-forward-access-application-cluster/).
-->
もしもデバッグ用途で特定のポートに対する接続が必要な場合は、[apiserver proxy](/jp/docs/tasks/access-application-cluster/access-cluster/#manually-constructing-apiserver-proxy-urls) や [`kubectl port-forward`](/jp/docs/tasks/access-application-cluster/port-forward-access-application-cluster/) が使えます。

<!--
  If you explicitly need to expose a Pod's port on the node, consider using a [NodePort](/docs/concepts/services-networking/service/#type-nodeport) Service before resorting to `hostPort`.
-->
もしもノード上のポッドが外に向かって公開するポートが必要であれば、 `hostPort` の手段を選ぶ前に、 [NodePort](/jp/docs/concepts/services-networking/service/#type-nodeport) の利用を検討してください。

<!--
- Avoid using `hostNetwork`, for the same reasons as `hostPort`.

- Use [headless Services](/docs/concepts/services-networking/service/#headless-
services) (which have a `ClusterIP` of `None`) for easy service discovery when you don't need `kube-proxy` load balancing.
-->
- `hostPort` と同様の理由のため、 `hostNetwork`  の利用を避けます。

- `kube-proxy` 負荷分散を使わない場合は 、[ヘッドレス・サービス（headless Services）](/jp/docs/concepts/services-networking/service/#headless-services) を使うとサービス発見（ディスカバリ）が簡単です。

<!--
## Using Labels
-->
## ラベルを使う {#using-labels}

<!--
- Define and use [labels](/docs/concepts/overview/working-with-objects/labels/) that identify __semantic attributes__ of your application or Deployment, such as `{ app: myapp, tier: frontend, phase: test, deployment: v3 }`. You can use these labels to select the appropriate Pods for other resources; for example, a Service that selects all `tier: frontend` Pods, or all `phase: test` components of `app: myapp`. See the [guestbook](https://github.com/kubernetes/examples/tree/{{< param "githubbranch" >}}/guestbook/) app for examples of this approach.
-->
- [ラベル](/jp/docs/concepts/overview/working-with-objects/labels/) の定義と使用には、アプリケーションやデプロイメントに対する個々の __semantic attributes（意味属性）__  であり、 `{ app: myapp, tier: frontend, phase: test, deployment: v3 }` のようなものです。これらのラベルを使い、他のリソースから適切なポッドを選択できるようにします。たとえば、サービスから `tier: frontend` のポッドを全ての選択や、 `app: myapp` の構成要素から `phase: test`  を全て選択するなどです。この手法を使う例は [guestbook](https://github.com/kubernetes/examples/tree/{{< param "githubbranch" >}}/guestbook/) をご覧ください。

<!--
A Service can be made to span multiple Deployments by omitting release-specific labels from its selector. [Deployments](/docs/concepts/workloads/controllers/deployment/) make it easy to update a running service without downtime.
-->
サービスは、セレクタからリリースを指定するラベルを省略しても、複数のデプロイメントの期間を経て作られます。[Deployment](/jp/docs/concepts/workloads/controllers/deployment/) によって、実行中のサービスに停止時間の無い更新がスムーズになります。

<!--
A desired state of an object is described by a Deployment, and if changes to that spec are _applied_, the deployment controller changes the actual state to the desired state at a controlled rate.
-->
オブジェクトの期待状態（desired state）はデプロイメントで記述します。
そして、もしも spec の変更が _適用されると_ 、デプロイメント・コントローラは現在の状態を期待状態に、制御した比率（レート）に従って変更します。

<!--
- You can manipulate labels for debugging. Because Kubernetes controllers (such as ReplicaSet) and Services match to Pods using selector labels, removing the relevant labels from a Pod will stop it from being considered by a controller or from being served traffic by a Service. If you remove the labels of an existing Pod, its controller will create a new Pod to take its place. This is a useful way to debug a previously "live" Pod in a "quarantine" environment. To interactively remove or add labels, use [`kubectl label`](/docs/reference/generated/kubectl/kubectl-commands#label).
-->
- デバッグ用途でラベルを操作できます。Kubernetes コントローラ（ReplicaSet など）とサービスは、ラベルに一致するポッドの発見や、ラベルに関連するポッド削除するためにセレクタ・ラベルを使います。
ポッドの削除によって、コントローラやサービスによって提供されるトラフィックが停止すると見なされます。
また、既に存在するポッドから適切なラベルを削除すると、ポッドを管理しているコントローラが対象のポッドを置き換えるため、新しいポッドを作成します。
これは直前まで「生きていた」ポッドを「隔離」環境でデバッグするのに役立ちます。
ラベルを対話的に追加・削除するには  [`kubectl label`](/docs/reference/generated/kubectl/kubectl-commands#label) を使います。

<!--
## Container Images
-->
## コンテナ・イメージ {#container-images}

<!--
- The default [imagePullPolicy](/docs/concepts/containers/images/#updating-images) for a container is `IfNotPresent`, which causes the [kubelet](/docs/admin/kubelet/) to pull an image only if it does not already exist locally. If you want the image to be pulled every time Kubernetes starts the container, specify `imagePullPolicy: Always`.
-->
- コンテナに対するデフォルトの [イメージ取得方針（imagePullPolicy）](/jp/docs/concepts/containers/images/#updating-images) は、 `IfNotPresent` （もしも存在しなければ）です。これはローカル環境上にイメージが存在しない時に限り、 [kubelet](/jp/docs/admin/kubelet/) がイメージを取得（pull）します。
もしも Kubernetes がコンテナの開始時に毎回イメージを取得したい場合は、 `imagePullPolicy: Always` を指定します。

<!--
  An alternative, but deprecated way to have Kubernetes always pull the image is to use the `:latest` tag, which will implicitly set the `imagePullPolicy` to `Always`.
-->
代替案として、非推奨な手法としては、Kubernetes でイメージの取得時常に `:latest` （最新）タグの指定です。これは `imagePullPolicy` を `Always` （常に）と暗黙的に指定します。

{{< note >}}
<!--
  **Note:** You should avoid using the `:latest` tag when deploying containers in production, because this makes it hard to track which version of the image is running and hard to roll back.
-->
  **メモ：** 本番敢行でのコンテナの配置（デプロイ）では、 `:latest` タグの使用を避けるべきでしょう。そうしなければ、実行しているイメージのバージョン追跡が困難ですし、巻き戻し（ロールバック）も困難です。
{{< /note >}}

<!--
- To make sure the container always uses the same version of the image, you can specify its [digest](https://docs.docker.com/engine/reference/commandline/pull/#pull-an-image-by-digest-immutable-identifier) (for example `sha256:45b23dee08af5e43a7fea6c4cf9c25ccf269ee113168c19722f87876677c5cb2`). This uniquely identifies a specific version of the image, so it will never be updated by Kubernetes unless you change the digest value.
-->
- コンテナで常に同じバージョンのイメージを確実に使うためには、[ダイジェスト（digest）](https://docs.docker.com/engine/reference/commandline/pull/#pull-an-image-by-digest-immutable-identifier) を使えます（例  `sha256:45b23dee08af5e43a7fea6c4cf9c25ccf269ee113168c19722f87876677c5cb2` ）。
これはイメージに対して固有の識別子でバージョンを指定しますので、ダイジェスト値を変更しない限り、Kubernetes が勝手にイメージを更新することは決してありません。 


<!--
## Using kubectl
-->
## kubectl を使う {#using-kubectl}

<!--
- Use `kubectl apply -f <directory>` or `kubectl create -f <directory>`. This looks for Kubernetes configuration in all `.yaml`, `.yml`, and `.json` files in `<directory>` and passes it to `apply` or `create`.
-->
- `kubectl apply -f <ディレクトリ>` や `kubectl create -f <ディレクトリ>` を使います。この形式は `<ディレクトリ>` 内にある全ての `.yaml` 、 `.yml` 、 `.json` ファイルにある構成情報をさがし、 `apply` や `create` に渡します。

<!--
- Use label selectors for `get` and `delete` operations instead of specific object names. See the sections on [label selectors](/docs/concepts/overview/working-with-objects/labels/#label-selectors) and [using labels effectively](/docs/concepts/cluster-administration/manage-deployment/#using-labels-effectively).
-->
- オブジェクト名を指定するかわりに、 `get` と `delete` ではラベル・セレクタを使います。
[ラベル・セレクタ](/jp/docs/concepts/overview/working-with-objects/labels/#label-selectors) と [ラベルを効率的に使う](/jp/docs/concepts/cluster-administration/manage-deployment/#using-labels-effectively) をご覧ください。

<!--
- Use `kubectl run` and `kubectl expose` to quickly create single-container Deployments and Services. See [Use a Service to Access an Application in a Cluster](/docs/tasks/access-application-cluster/service-access-application-cluster/) for an example.
-->
- `kubectl run` と `kubectl expose`  で、素早く１つのコンテナデプロイメントとサービスを作成できます。例は [サービスをクラスタ内のアプリケーションへのアクセスのために使う](/jp/docs/tasks/access-application-cluster/service-access-application-cluster/) をご覧ください。


{{% /capture %}}


