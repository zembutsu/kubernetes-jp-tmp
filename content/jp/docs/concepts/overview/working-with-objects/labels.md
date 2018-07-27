---
reviewers:
- mikedanese
title: ラベルとセレクタ
content_template: templates/concept
weight: 40
---

{{% capture overview %}}

<!--
_Labels_ are key/value pairs that are attached to objects, such as pods.
Labels are intended to be used to specify identifying attributes of objects that are meaningful and relevant to users, but do not directly imply semantics to the core system.
Labels can be used to organize and to select subsets of objects.  Labels can be attached to objects at creation time and subsequently added and modified at any time.
Each object can have a set of key/value labels defined.  Each Key must be unique for a given object.
-->
_ラベル（Labels）_ とはキー・バリューの組み合わせであり、ポッドのようなオブジェクトに割り当てます。ラベルを使う目的は、オブジェクトの属性を明確に識別するためです。つまりユーザにとっては意味や関連性がありますが、コア・システムにとっては直接的な意味はありません。オブジェクトの集まりを整理・選ぶためにラベルを使えます。オブジェクトの作成時にラベルを付けられますし、後から何時でも追加や変更できます。各オブジェクトはキー・バリューのセットでラベルを定義します。各キーはオブジェクトごとにユニークな必要があります。

```json
"metadata": {
  "labels": {
    "key1" : "value1",
    "key2" : "value2"
  }
}
```

<!--
We'll eventually index and reverse-index labels for efficient queries and watches, use them to sort and group in UIs and CLIs, etc. We don't want to pollute labels with non-identifying, especially large and/or structured, data. Non-identifying information should be recorded using [annotations](/docs/concepts/overview/working-with-objects/annotations/).
-->
効果的なクエリ（問い合わせ）や監視で使うため、最終的にはラベルの一覧と逆引き一覧を UI や CLI 等で並べ替えたりグループ化したりに使えます。識別する以外の用途でラベルを汚染するつもりはありません。特に大規模なデータ構造においては尚更です。識別以外の情報は [アノテーション（annotation）](/jp/docs/concepts/overview/working-with-objects/annotations/) を使って記録すべきです。

{{% /capture %}}

{{< toc >}}

{{% capture body %}}

<!--
## Motivation
-->
## 動機 {#motivation}

<!--
Labels enable users to map their own organizational structures onto system objects in a loosely coupled fashion, without requiring clients to store these mappings.
-->
ラベルの利用により、ユーザは自分の組織構造システム・オブジェクトと緩やかに連動して関連付けられます。これには、クライアント側では割り当てに関する情報を持つ必要はありません。

<!--
Service deployments and batch processing pipelines are often multi-dimensional entities (e.g., multiple partitions or deployments, multiple release tracks, multiple tiers, multiple micro-services per tier). Management often requires cross-cutting operations, which breaks encapsulation of strictly hierarchical representations, especially rigid hierarchies determined by the infrastructure rather than by users.
-->
サービス展開（デプロイメント）とバッチ処理パイプラインでは、頻繁に多次元要素を扱います（例：複数に分割した展開（デプロイメント）、複数のリリース・トラック（軌道）、ティアごとの複数のマイクロサービス）。管理では分野横断的な運用が頻繁に求められます。運用では、厳密な階層構造をカプセル化して表現するのではありません。むしろ、ユーザではなく基盤（インフラストラクチャ）に基づき、特別に固定した階層構造を用います。
<!--
Example labels:
-->
ラベルの例：

   * `"release" : "stable"`, `"release" : "canary"`
   * `"environment" : "dev"`, `"environment" : "qa"`, `"environment" : "production"`
   * `"tier" : "frontend"`, `"tier" : "backend"`, `"tier" : "cache"`
   * `"partition" : "customerA"`, `"partition" : "customerB"`
   * `"track" : "daily"`, `"track" : "weekly"`

<!--
These are just examples of commonly used labels; you are free to develop your own conventions. Keep in mind that label Key must be unique for a given object.
-->
これらはラベルとして一般的に使われる例です。そのため、自分が使いやすい様に自由に作成できます。ラベルのキーは対象となるオブジェクト内ではユニークな必要がありますので、この点はご注意ください。

<!--
## Syntax and character set
-->
## 構文と文字セット {#syntax-and-character-set}

<!--
_Labels_ are key/value pairs. Valid label keys have two segments: an optional prefix and name, separated by a slash (`/`).  The name segment is required and must be 63 characters or less, beginning and ending with an alphanumeric character (`[a-z0-9A-Z]`) with dashes (`-`), underscores (`_`), dots (`.`), and alphanumerics between.  The prefix is optional.  If specified, the prefix must be a DNS subdomain: a series of DNS labels separated by dots (`.`), not longer than 253 characters in total, followed by a slash (`/`).
If the prefix is omitted, the label Key is presumed to be private to the user. Automated system components (e.g. `kube-scheduler`, `kube-controller-manager`, `kube-apiserver`, `kubectl`, or other third-party automation) which add labels to end-user objects must specify a prefix.  The `kubernetes.io/` prefix is reserved for Kubernetes core components.
-->
_ラベル_ はキー・バリューの組み合わせです。有効なラベル・キーには２つの部分あります。オプションのプレフィックスと名前を分けるのはスラッシュ（`/`）です。名前の部分に必要なのは、63文字以内の文字列であり、冒頭・末尾が英数字（`[a-z0-9A-Z]`）です。途中には、ダッシュ（`-`）、アンダースコア（`_`）、ドット（`.`）と英数字が使えます。プレフィックスはオプションです。もし指定した場合は、プレフィックスは DNS サブドメインの必要があります。つまり、DNS が続くラベルはドット（`.`）で区切るひつようがあり、合計で 253文字を越えられず、後にはスラッシュ（`/`）が続きます。もしもプリフィックスがなければ、ラベル・キーはユーザにとってプライベートなものとみなされます。システム構成要素は自動化されています（例：`kube-scheduler`、 `kube-controller-manager`、 `kube-apiserver`、 `kubectl`、あるいは他のサードパーティによる自動設定） 。エンドユーザがオブジェクトにラベルを追加するには、プレフィックスの指定が必須です。 `kubernetes.io/` プレフィックスは Kubernetes の中心となる構成要素のために予約済みです。

<!--
Valid label values must be 63 characters or less and must be empty or begin and end with an alphanumeric character (`[a-z0-9A-Z]`) with dashes (`-`), underscores (`_`), dots (`.`), and alphanumerics between.
-->
有効なラベルの値は63文字以下の文字か、内容がからであるか、冒頭・末尾が英数字（`[a-z0-9A-Z]`）です。途中には、ダッシュ（`-`）、アンダースコア（`_`）、ドット（`.`）と英数字が使えます。

<!--
## Label selectors
-->
## ラベル・セレクタ {#label-selectors}

<!--
Unlike [names and UIDs](/docs/user-guide/identifiers), labels do not provide uniqueness. In general, we expect many objects to carry the same label(s).
-->
[名前や UID](/jp/docs/user-guide/identifiers) とは違い、ラベルは一意性を提供しません。通常、多くのオブジェクトが同じラベルを持つのが予想されます。

<!--
Via a _label selector_, the client/user can identify a set of objects. The label selector is the core grouping primitive in Kubernetes.
-->
 _ラベル・セレクタ（label selector）_ を経由すると、クライアントやユーザはオブジェクトの集まりを識別できます。ラベル・セレクタは Kubernetes におけるコアなグループ化単位（プリミティブ）です。

<!--
The API currently supports two types of selectors: _equality-based_ and _set-based_.
A label selector can be made of multiple _requirements_ which are comma-separated. In the case of multiple requirements, all must be satisfied so the comma separator acts as a logical _AND_ (`&&`) operator.
-->
API は現時点で２種類のセレクタをサポートしています。等号を使うもの（ _quality-based_）と、組み合わせを使うもの（ _set-based_ ）です。ラベル・セレクタはコンマ記号で分割する複数の _必要条件_ で構成されます。複数の必要条件のすべてをコンマ記号で記述しますが、これは論理演算子 _AND_ （`&&`）として扱われます。

<!--
An empty label selector (that is, one with zero requirements) selects every object in the collection.
-->
空のラベル・セレクタ（使うには少なくとも0が必要）は、コレクション内にある全てのオブジェクトから選びます。

<!--
A null label selector (which is only possible for optional selector fields) selects no objects.
-->
ヌル（null）ラベル・セレクタ（オプションのセレクタ・フィールドのみの可能性）はオブジェクトがないものを選びます。


{{< note >}}
<!--
**Note**: the label selectors of two controllers must not overlap within a namespace, otherwise they will fight with each other.
-->
**メモ**：２つのコレクタを持つラベル・セレクタは、名前空間内で重複できないだけでなく、お互いの競合もできません。
{{< /note >}}

<!--
### _Equality-based_ requirement
-->
### 等号を使う（ _Equality-based_ ）要件

<!--
_Equality-_ or _inequality-based_ requirements allow filtering by label keys and values. Matching objects must satisfy all of the specified label constraints, though they may have additional labels as well.
Three kinds of operators are admitted `=`,`==`,`!=`. The first two represent _equality_ (and are simply synonyms), while the latter represents _inequality_. For example:
-->
ラベルのキーとバリューのフィルタに等号や不等号を使えます。条件が一致するオブジェクトは、指定したラベル条件をすべて満たす必要があります。追加のラベルがあっても同様です。３つの演算子  `=` 、 `==` 、 `!=` が許可されています。はじめの２つは等しい意味を表し（そして簡単にした表記です）、最後のは等しくない意味を表します。例：

```
environment = production
tier != frontend
```

<!--
The former selects all resources with key equal to `environment` and value equal to `production`.
The latter selects all resources with key equal to `tier` and value distinct from `frontend`, and all resources with no labels with the `tier` key.
One could filter for resources in `production` excluding `frontend` using the comma operator: `environment=production,tier!=frontend`
-->
前者で選択するのは、すべてのリソース中からキーが `environment` であり、値が `production` と等しいものです。後者で選択するのは、キーが `tier` に一致しますが、値が `frontend`ではないものかつ、すべてのリソースから `tier` キーがないものです。`production` にあるリソースから `frontend` を除くフィルタを使うには、カンマ区切りを使います：`environment=production,tier!=frontend` 。

<!--
One usage scenario for equality-based label requirement is for Pods to specify
node selection criteria. For example, the sample Pod below selects nodes with
the label "`accelerator=nvidia-tesla-p100`".
-->
使用例としては、ポッドを稼働するノードの条件を指定するためです。たとえば、以下はサンプル・ポッドが動作するノードは、ラベル  "`accelerator=nvidia-tesla-p100`" を選びます。

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: cuda-test
spec:
  containers:
    - name: cuda-test
      image: "k8s.gcr.io/cuda-vector-add:v0.1"
      resources:
        limits:
          nvidia.com/gpu: 1
  nodeSelector:
    accelerator: nvidia-tesla-p100
```

<!--
### _Set-based_ requirement
-->
### セットを使う（ _Set-based_ ）要件

<!--
_Set-based_ label requirements allow filtering keys according to a set of values. Three kinds of operators are supported: `in`,`notin` and `exists` (only the key identifier). For example:
-->
セットを使う（ _Set-based_）要件により、セットした値に一致するキーでフィルタできます。３種類の演算子 `in` 、 `notin` 、 `exist`（キー識別子のみ）をサポートします。例：

```
environment in (production, qa)
tier notin (frontend, backend)
partition
!partition
```

<!--
The first example selects all resources with key equal to `environment` and value equal to `production` or `qa`.
The second example selects all resources with key equal to `tier` and values other than `frontend` and `backend`, and all resources with no labels with the `tier` key.
The third example selects all resources including a label with key `partition`; no values are checked.
The fourth example selects all resources without a label with key `partition`; no values are checked.
Similarly the comma separator acts as an _AND_ operator. So filtering resources with a `partition` key (no matter the value) and with `environment` different than  `qa` can be achieved using `partition,environment notin (qa)`.
The _set-based_ label selector is a general form of equality since `environment=production` is equivalent to `environment in (production)`; similarly for `!=` and `notin`.
-->
１つめの例では、すべてのリソースからキーが `environment` に一致し、かつ、値が `production` か `qa` のものを選びます。２つめの例はすべてのリソースからキーが `tier` に一致し、値が `frontend` または `backend` ではないものを選ぶのと、すべてのリソースのうち `tier` キーのないラベルを持つものです。３つめの例はすべてのリソースのうち、ラベルのキーに `partition` を含むものを選びますが、値があるかどうかはチェックしません。４つめの例はすべてのリソースからラベルのキーに `partition` がないものを選びます。擬似的にカンマ記号を _AND_ 演算子として使えます。そのため、 `patition` キー（値が何であれ）と、 `environment` が `qa` でない条件を達成するには `partition,environment notin (qa)` を使います。セットを使うラベル・セレクタ `environment=production` は、 `environment in (production)` と同じです。つまり `!=` と `notin` は同じです。

<!--
_Set-based_ requirements can be mixed with _equality-based_ requirements. For example: `partition in (customerA, customerB),environment!=qa`.
-->
セットを使う条件と等号を使うものは、組み合わせて利用できます。例：`partition in (customerA, customerB),environment!=qa` 。


## API

<!--
### LIST and WATCH filtering
-->
### LIST と WATCH フィルタリング

<!--
LIST and WATCH operations may specify label selectors to filter the sets of objects returned using a query parameter. Both requirements are permitted (presented here as they would appear in a URL query string):
-->
ラベル・セレクタで LIST （一覧表示）と WATCH （監視）の処理は、オブジェクト・セットの問い合わせパラメータを使ってフィルタできます。

<!--
  * _equality-based_ requirements: `?labelSelector=environment%3Dproduction,tier%3Dfrontend`
  * _set-based_ requirements: `?labelSelector=environment+in+%28production%2Cqa%29%2Ctier+in+%28frontend%29`
-->
  * _equality-based_ 必要条件: `?labelSelector=environment%3Dproduction,tier%3Dfrontend`
  * _set-based_ 必要条件: `?labelSelector=environment+in+%28production%2Cqa%29%2Ctier+in+%28frontend%29`

<!--
Both label selector styles can be used to list or watch resources via a REST client. For example, targeting `apiserver` with `kubectl` and using _equality-based_ one may write:
-->
ラベル・セレクタ・スタイルの両方を、 REST クライアントを通してリソースの一覧表示または監視のために使えます。例えば、 `kubectl` で `apiserver` を対象にして  _equality-based_ を利用する方法の１つは：

```shell
$ kubectl get pods -l environment=production,tier=frontend
```

<!--
or using _set-based_ requirements:
-->
あるいは _set-based_ 必要条件を使います:

```shell
$ kubectl get pods -l 'environment in (production),tier in (frontend)'
```

<!--
As already mentioned _set-based_ requirements are more expressive.  For instance, they can implement the _OR_ operator on values:
-->
既に言及している _set-based_ 必要条件については、記法が多彩です。たとえば、 _OR_ 演算子を値に埋め込むには：


```shell
$ kubectl get pods -l 'environment in (production, qa)'
```

<!--
or restricting negative matching via _exists_ operator:
-->
あるいは _exists_ 演算子で一致しないものに制限するには：

```shell
$ kubectl get pods -l 'environment,environment notin (frontend)'
```

<!--
### Set references in API objects
-->
## API オブジェクト設定リファレンス {#set-references-in-api-objects}

<!--
Some Kubernetes objects, such as [`services`](/docs/user-guide/services) and [`replicationcontrollers`](/docs/user-guide/replication-controller), also use label selectors to specify sets of other resources, such as [pods](/docs/user-guide/pods).
-->
[`services`（サービス）](/jp/docs/user-guide/services) と [`replicationcontrollers`（レプリケーション・コントローラ）](/jp/docs/user-guide/replication-controller) といった一部の Kubernetes オブジェクトは、ラベル・セレクタとして [ポッド](/jp/docs/user-guide/pods) のような他のリソース指定も可能です。

<!--
#### Service and ReplicationController
--->
#### サービスとレプリケーション・コントローラ {#service-and-replicationcontroller}

<!--
The set of pods that a `service` targets is defined with a label selector. Similarly, the population of pods that a `replicationcontroller` should manage is also defined with a label selector.
-->
ポッドの集まりとは、ラベル・セレクタで `service` と定義したものが対象です。同様に、ポッドの集まりとはラベル・セレクタで   `replicationcontroller`  として定義したものが管理対象とも言えるでしょう。

<!--
Labels selectors for both objects are defined in `json` or `yaml` files using maps, and only _equality-based_ requirement selectors are supported:
-->
ラベル・セレクタがサポートしているのは、 `json` や `yaml`  ファイルを使って割り当てる（mapする）オブジェクトか、_equality-based_ 必要条件のセレクタのみの両方です。

```json
"selector": {
    "component" : "redis",
}
``
<!--
or
-->
または

```yaml
selector:
    component: redis
```

<!--
this selector (respectively in `json` or `yaml` format) is equivalent to `component=redis` or `component in (redis)`.
-->
このセレクタ（個々の `json` か `yaml` 形式）は `component=redis` や  `component in (redis)` と同等です。

<!--
#### Resources that support set-based requirements
-->
### set-based 必要条件をサポートするリソース {#resources-that-support-set-based-requirements}

<!--
Newer resources, such as [`Job`](/docs/concepts/jobs/run-to-completion-finite-workloads/), [`Deployment`](/docs/concepts/workloads/controllers/deployment/), [`Replica Set`](/docs/concepts/workloads/controllers/replicaset/), and [`Daemon Set`](/docs/concepts/workloads/controllers/daemonset/), support _set-based_ requirements as well.
-->

[`Job`（ジョブ）](/jp/docs/concepts/jobs/run-to-completion-finite-workloads/)、 [`Deployment`（デプロイメント）](/jp/docs/concepts/workloads/controllers/deployment/)、 [`Replica Set`（レプリカ・セット）](/jp/docs/concepts/workloads/controllers/replicaset/)、 [`Daemon Set`（デーモン・セット）](/jp/docs/concepts/workloads/controllers/daemonset/) のような新しいリソースは、 _set-based_ 必要条件も同様にサポートします：


```yaml
selector:
  matchLabels:
    component: redis
  matchExpressions:
    - {key: tier, operator: In, values: [cache]}
    - {key: environment, operator: NotIn, values: [dev]}
```

<!--
`matchLabels` is a map of `{key,value}` pairs. A single `{key,value}` in the `matchLabels` map is equivalent to an element of `matchExpressions`, whose `key` field is "key", the `operator` is "In", and the `values` array contains only "value". `matchExpressions` is a list of pod selector requirements. Valid operators include In, NotIn, Exists, and DoesNotExist. The values set must be non-empty in the case of In and NotIn. All of the requirements, from both `matchLabels` and `matchExpressions` are ANDed together -- they must all be satisfied in order to match.
-->
`matchLables` は `{key.value}` の組み合わせを割り当て（マップ）します。 `matchLables` 中の `{key.value}` が割り当てる（マップする）のは、`matchExpressions` 要素の `key` フィールドを「key」として、 `operator` を「In」として、 `values` フィールドをのアレイを「Value」として扱うのと同等です。 `matchExpressions`  はポッド・セレクタが必要とするリストです。有効な演算子には Notin、おExist、DoesNotExist が含まれます。In と notIn の場合は value に空ではない値が必ず入っている必要があります。すべての必要条件に `matchLables` と `matchExpressions` が含まれており、 AND も一緒に使えます。並べた順番で条件を満たす必要があります。

<!--
#### Selecting sets of nodes
-->
#### ノードのセットを選択 {#selecting-sets-of-nodes}

<!--
One use case for selecting over labels is to constrain the set of nodes onto which a pod can schedule.
See the documentation on [node selection](/docs/concepts/configuration/assign-pod-node/) for more information.
-->
ラベルの使用例としては、どのポッドがスケジュール可能かどうか制限する方法があります。詳細は [ノード選択](/jp/docs/concepts/configuration/assign-pod-node/) をご覧ください。

{{% /capture %}}