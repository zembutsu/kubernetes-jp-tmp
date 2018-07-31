---
title: Kubernetes オブジェクト管理に宣言型の設定情報ファイルを使う
content_template: templates/concept
weight: 40
---

{{% capture overview %}}
<!--
Kubernetes objects can be created, updated, and deleted by storing multiple
object configuration files in a directory and using `kubectl apply` to
recursively create and update those objects as needed. This method
retains writes made to live objects without merging the changes
back into the object configuration files.
-->
ディレクトリ内の設定情報ファイルに複数のオブジェクトを書くことで、Kubernetes オブジェクトの作成・更新・削除ができます。また、 `kubectl apply` を使うと必要に応じてオブジェクトの作成と更新ができます。この手法では、変更がオブジェクト設定情報ファイルには反映されず、稼働中のオブジェクトが書き込まれたデータを保持します。
{{% /capture %}}

{{% capture body %}}

<!--
## Trade-offs
-->
## トレードオフ

<!--
The `kubectl` tool supports three kinds of object management:
-->
`kubectl` ツールは３つのオブジェクト管理をサポートします。

<!--
* Imperative commands
* Imperative object configuration
* Declarative object configuration
-->
* 命令型コマンド
* 命令型オブジェクト設定情報
* 宣言型オブジェクト設定情報

<!--
See [Kubernetes Object Management](/docs/concepts/overview/object-management-kubectl/overview/)
for a discussion of the advantages and disadvantage of each kind of object management.
-->
それぞれのオブジェクト管理手法の利点と欠点に関する議論は、[Kubernetes オブジェクト管理](/jp/docs/concepts/overview/object-management-kubectl/overview/) をご覧ください。

<!--
## Before you begin
-->
## 始める前に {#before-you-begin}

<!--
Declarative object configuration requires a firm understanding of
the Kubernetes object definitions and configuration. Read and complete
the following documents if you have not already:
-->
宣言型オブジェクト設定情報には、Kubernetes オブジェクト定義と設定情報に関するしっかりとした理解が必要です。準備ができていなければ、以下のドキュメントを読み終えてください。

<!--
- [Managing Kubernetes Objects Using Imperative Commands](/docs/concepts/overview/object-management-kubectl/imperative-command/)
- [Imperative Management of Kubernetes Objects Using Configuration Files](/docs/concepts/overview/object-management-kubectl/imperative-config/)
-->
- [Kubernetes オブジェクトを命令型コマンドで管理](/jp/docs/concepts/overview/object-management-kubectl/imperative-command/)
- [Kubernetes オブジェクトを設定情報ファイルを使って命令型で管理](/docs/concepts/overview/object-management-kubectl/imperative-config/)

<!--
Following are definitions for terms used in this document:
-->
この文章では、用語の定義を次のようにしています：

<!--
- *object configuration file / configuration file*: A file that defines the
  configuration for a Kubernetes object. This topic shows how to pass configuration
  files to `kubectl apply`. Configuration files are typically stored in source control, such as Git.
- *live object configuration / live configuration*: The live configuration
  values of an object, as observed by the Kubernetes cluster. These are kept in the Kubernetes
  cluster storage, typically etcd.
- *declarative configuration writer /  declarative writer*: A person or software component
  that makes updates to a live object. The live writers referred to in this topic make changes
  to object configuration files and run `kubectl apply` to write the changes.
-->

- *オブジェクト設定情報ファイル（object configuration file） / 設定情報ファイル（configuration file）* ：Kubernetes オブジェクトに関する設定情報を定義したファイルです。このトピックでは設定情報ファイルを `kubectl apply` に渡す方法法を紹介します。設定情報ファイルは、通常 Git のようなソース管理システム（source control）に保管します。
* *稼働オブジェクト設定情報（live object configuration） / 稼働設定情報（live configuration） * ：オブジェクトの稼働設定情報の値とは、 Kubernetes クラスタ側から見えるものです。これらは Kubernetes クラスタ・ストレージ（記憶装置）、通常は etcd に保管されます。
* *宣言型設定情報の作者（declarative configuration writer） / 宣言型の作者（declarative writer）* ：ソフトウェア構成要素（コンポーネント）の稼働オブジェクトを更新する人です。このトピックで扱うのは、この作者が参照するためであり、オブジェクト設定情報ファイルを変更して `kubectl apply` を実行して設定を反映します。

<!--
## How to create objects
-->
## オブジェクトの作成方法 {#how-to-create-objects}

<!--
Use `kubectl apply` to create all objects, except those that already exist,
defined by configuration files in a specified directory:
-->
`kubectl apply` を使い、指定したディレクトリ内にある設定情報ファイルで定義した全オブジェクトを作成します。ただし既に存在しているものは除外します。

<!--
```shell
kubectl apply -f <directory>/
```
-->
```shell
kubectl apply -f <ディレクトリ>/
```

<!--
This sets the `kubectl.kubernetes.io/last-applied-configuration: '{...}'`
annotation on each object. The annotation contains the contents of the object
configuration file that was used to create the object.
-->
各オブジェクトには `kubectl.kubernetes.io/last-applied-configuration: '{...}'` アノテーション（注釈）が設定されています。アノテーション（注釈）にはオブジェクト作成時に使われたオブジェクト設定情報ファイルの内容を含んでいます。

{{< note >}}
<!--
**Note:** Add the `-R` flag to recursively process directories.
-->
**メモ：** `-R` フラグはディレクトリを再帰的に処理します（訳者注：下層にあるディレクトリもまとめて処理します）。
{{< /note >}}

<!--
Here's an example of an object configuration file:
-->
こちらがオブジェクト構成情報ファイルの例です：

{{< code file="simple_deployment.yaml" >}}

<!--
Create the object using `kubectl apply`:
-->
`kubectl apply` を使ってオブジェクトを作成します：

```shell
kubectl apply -f https://k8s.io/docs/concepts/overview/object-management-kubectl/simple_deployment.yaml
```

<!--
Print the live configuration using `kubectl get`:
-->
`kubectl get` を使って設定情報を表示します：

```shell
kubectl get -f https://k8s.io/docs/concepts/overview/object-management-kubectl/simple_deployment.yaml -o yaml
```

<!--
The output shows that the `kubectl.kubernetes.io/last-applied-configuration` annotation
was written to the live configuration, and it matches the configuration file:
-->
出力が表示しているのは  `kubectl.kubernetes.io/last-applied-configuration` アノテーション（注釈）が稼働設定情報に書き込まれた時のものであり、設定情報ファイルと一致します：


```yaml
kind: Deployment
metadata:
  annotations:
    # ...
    # This is the json representation of simple_deployment.yaml
    # It was written by kubectl apply when the object was created
    kubectl.kubernetes.io/last-applied-configuration: |
      {"apiVersion":"apps/v1","kind":"Deployment",
      "metadata":{"annotations":{},"name":"nginx-deployment","namespace":"default"},
      "spec":{"minReadySeconds":5,"selector":{"matchLabels":{"app":nginx}},"template":{"metadata":{"labels":{"app":"nginx"}},
      "spec":{"containers":[{"image":"nginx:1.7.9","name":"nginx",
      "ports":[{"containerPort":80}]}]}}}}
  # ...
spec:
  # ...
  minReadySeconds: 5
  selector:
    matchLabels:
      # ...
      app: nginx
  template:
    metadata:
      # ...
      labels:
        app: nginx
    spec:
      containers:
      - image: nginx:1.7.9
        # ...
        name: nginx
        ports:
        - containerPort: 80
        # ...
      # ...
    # ...
  # ...
```

<!--
## How to update objects
-->
## オブジェクトの更新方法 {#how-to-update-objects}

<!--
You can also use `kubectl apply` to update all objects defined in a directory, even
if those objects already exist. This approach accomplishes the following:
-->
`kubectl apply` を使って、ディレクトリで定義したすべてのオブジェクトの更新もできます。更新対象には既存のオブジェクトも含みます。この方法は以下のように進めます：

<!--
1. Sets fields that appear in the configuration file in the live configuration.
2. Clears fields removed from the configuration file in the live configuration.
-->
1. 稼働設定情報に含まれる設定情報ファイルに記載されたフィールドをセットする。
1. 稼働設定情報に含まれる設定情報ファイルから、不要なフィールドを削除する。

<!--
```shell
kubectl apply -f <directory>/
```
-->
```shell
kubectl apply -f <ディレクトリ>/
```

{{< note >}}
<!--
**Note:** Add the `-R` flag to recursively process directories.
-->
**メモ：** `-R` フラグを追加するとディレクトリを再帰的に処理します。
{{< /note >}}

<!--
Here's an example configuration file:
-->
コチラは設定情報ファイルの例です：

{{< code file="simple_deployment.yaml" >}}

<!--
Create the object using `kubectl apply`:
-->
`kubectl apply` を使ってオブジェクトを作成します：

```shell
kubectl apply -f https://k8s.io/docs/concepts/overview/object-management-kubectl/simple_deployment.yaml
```

{{< note >}}
<!--
**Note:** For purposes of illustration, the preceding command refers to a single
configuration file instead of a directory.
-->
**メモ：** インストール用途のため、ディレクトリではなく１つの設定情報ファイルを参照するコマンドを事前に実行します。
{{< /note >}}

<!--
Print the live configuration using `kubectl get`:
-->
`kubectl get` を使って稼働設定情報を表示します。


```shell
kubectl get -f https://k8s.io/docs/concepts/overview/object-management-kubectl/simple_deployment.yaml -o yaml
```

<!--
The output shows that the `kubectl.kubernetes.io/last-applied-configuration` annotation
was written to the live configuration, and it matches the configuration file:
-->
出力が表示しているのは  `kubectl.kubernetes.io/last-applied-configuration` アノテーション（注釈）が稼働設定情報に書き込まれた時のものであり、設定情報ファイルと一致します：

```yaml
kind: Deployment
metadata:
  annotations:
    # ...
    # This is the json representation of simple_deployment.yaml
    # It was written by kubectl apply when the object was created
    kubectl.kubernetes.io/last-applied-configuration: |
      {"apiVersion":"apps/v1","kind":"Deployment",
      "metadata":{"annotations":{},"name":"nginx-deployment","namespace":"default"},
      "spec":{"minReadySeconds":5,"selector":{"matchLabels":{"app":nginx}},"template":{"metadata":{"labels":{"app":"nginx"}},
      "spec":{"containers":[{"image":"nginx:1.7.9","name":"nginx",
      "ports":[{"containerPort":80}]}]}}}}
  # ...
spec:
  # ...
  minReadySeconds: 5
  selector:
    matchLabels:
      # ...
      app: nginx
  template:
    metadata:
      # ...
      labels:
        app: nginx
    spec:
      containers:
      - image: nginx:1.7.9
        # ...
        name: nginx
        ports:
        - containerPort: 80
        # ...
      # ...
    # ...
  # ...
```

<!--
Directly update the `replicas` field in the live configuration by using `kubectl scale`.
This does not use `kubectl apply`:
-->
`kubectl scale` を使い、稼働設定情報の `replicas` フィールドを直接更新します。ここでは `kubectl apply` を使いません：

```shell
kubectl scale deployment/nginx-deployment --replicas=2
```

<!--
Print the live configuration using `kubectl get`:
-->
`kubectl get` を使い、稼働設定情報を表示します。

```shell
kubectl get -f https://k8s.io/docs/concepts/overview/object-management-kubectl/simple_deployment.yaml -o yaml
```

<!--
The output shows that the `replicas` field has been set to 2, and the `last-applied-configuration`
annotation does not contain a `replicas` field:
-->
出力が表示するのは、 `replicas` フィールドが 2 に設定されており、 `last-applied-configuration` （最後に適用された変更）アノテーション（注釈）には `replicas` を含みません：

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  annotations:
    # ...
    # note that the annotation does not contain replicas
    # because it was not updated through apply
    kubectl.kubernetes.io/last-applied-configuration: |
      {"apiVersion":"apps/v1","kind":"Deployment",
      "metadata":{"annotations":{},"name":"nginx-deployment","namespace":"default"},
      "spec":{"minReadySeconds":5,"selector":{"matchLabels":{"app":nginx}},"template":{"metadata":{"labels":{"app":"nginx"}},
      "spec":{"containers":[{"image":"nginx:1.7.9","name":"nginx",
      "ports":[{"containerPort":80}]}]}}}}
  # ...
spec:
  replicas: 2 # written by scale
  # ...
  minReadySeconds: 5
  selector:
    matchLabels:
      # ...
      app: nginx
  template:
    metadata:
      # ...
      labels:
        app: nginx
    spec:
      containers:
      - image: nginx:1.7.9
        # ...
        name: nginx
        ports:
        - containerPort: 80
      # ...
```

<!--
Update the `simple_deployment.yaml` configuration file to change the image from
`nginx:1.7.9` to `nginx:1.11.9`, and delete the `minReadySeconds` field:
-->
`simple_deployment.yaml` 設定情報ファイルを更新し、イメージを `nginx:1.7.9` から `nginx:1.11.9` に変更します。そして、 `minReadySeconds` フィールドを削除します。

{{< code file="update_deployment.yaml" >}}

<!--
Apply the changes made to the configuration file:
-->
設定情ファイルに対する変更を反映します。

```shell
kubectl apply -f https://k8s.io/docs/concepts/overview/object-management-kubectl/update_deployment.yaml
```

<!--
Print the live configuration using `kubectl get`:
-->
`kubectl get` を使って稼働設定情報を表示します：

```
kubectl get -f https://k8s.io/docs/concepts/overview/object-management-kubectl/simple_deployment.yaml -o yaml
```

<!--
The output shows the following changes to the live configuration:
-->
出力が表示するのは、以下の稼働設定情報に対する変更です：

<!--
- The `replicas` field retains the value of 2 set by `kubectl scale`.
  This is possible because it is omitted from the configuration file.
- The `image` field has been updated to `nginx:1.11.9` from `nginx:1.7.9`.
- The `last-applied-configuration` annotation has been updated with the new image.
- The `minReadySeconds` field has been cleared.
- The `last-applied-configuration` annotation no longer contains the `minReadySeconds` field.
-->
- `replicas` （複製数）フィールドは `kubectl scale` で設定された 2 の値を保持。設定情報ファイルでは省略されている可能性がある。
- `image` （イメージ）フィールドは `nginx:1.7.9` から `nginx:1.11.9` に変更。
- `last-applied-configuration` （最後に適用された変更）アノテーション（注釈）は、新しいイメージに更新されている。
- `minReadySeconds` （最小待機秒）フィールどがクリアされる。
- `last-applied-configuration` （最後に適用された変更）アノテーション（注釈）には `minReadySeconds` フィールドが無くなる。


```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  annotations:
    # ...
    # The annotation contains the updated image to nginx 1.11.9,
    # but does not contain the updated replicas to 2
    kubectl.kubernetes.io/last-applied-configuration: |
      {"apiVersion":"apps/v1","kind":"Deployment",
      "metadata":{"annotations":{},"name":"nginx-deployment","namespace":"default"},
      "spec":{"selector":{"matchLabels":{"app":nginx}},"template":{"metadata":{"labels":{"app":"nginx"}},
      "spec":{"containers":[{"image":"nginx:1.11.9","name":"nginx",
      "ports":[{"containerPort":80}]}]}}}}
    # ...
spec:
  replicas: 2 # Set by `kubectl scale`.  Ignored by `kubectl apply`.
  # minReadySeconds cleared by `kubectl apply`
  # ...
  selector:
    matchLabels:
      # ...
      app: nginx
  template:
    metadata:
      # ...
      labels:
        app: nginx
    spec:
      containers:
      - image: nginx:1.11.9 # Set by `kubectl apply`
        # ...
        name: nginx
        ports:
        - containerPort: 80
        # ...
      # ...
    # ...
  # ...
```

{{< warning >}}
<!--
**Warning:** Mixing `kubectl apply` with the imperative object configuration commands
`create` and `replace` is not supported. This is because `create`
and `replace` do not retain the `kubectl.kubernetes.io/last-applied-configuration`
that `kubectl apply` uses to compute updates.
-->
**警告：** ``kubectl apply` は命令型オブジェクト設定コマンドの `create` および `replace` との混在をサポートしていません。これは `create` と `replace` は  `kubectl.kubernetes.io/last-applied-configuration` を保持しないからです。 `kubectl apply` は数を更新するために使います。
{{< /warning >}}

<!--
## How to delete objects
-->
## オブジェクトの削除方法 {#how-to-delete-object}

<!--
There are two approaches to delete objects managed by `kubectl apply`.
-->
オブジェクトを削除するには `kubectl apply` で管理する２つの方法があります。

<!--
### Recommended: `kubectl delete -f <filename>`
-->
### 推奨： `kubectl delete -f <filename>` {#recommended}

<!--
Manually deleting objects using the imperative command is the recommended
approach, as it is more explicit about what is being deleted, and less likely
to result in the user deleting something unintentionally:
-->
オブジェクトの削除には、命令型コマンドを使って直接削除する手法が推奨されています。これは何を削除するかの対象が明確だからです。また、ユーザの意図しない削除をより減らすためです。

<!--
```shell
kubectl delete -f <filename>
```
-->
```shell
kubectl delete -f <ファイル名>
```

<!--
### Alternative: `kubectl apply -f <directory/> --prune -l your=label`
-->
### 別の方法：: `kubectl apply -f <ディレクトリ/> --prune -l your=label` {#alternative}

<!--
Only use this if you know what you are doing.
-->
何を行おうとしているか理解している場合のみ使ってください：

{{< warning >}}
<!--
**Warning:** `kubectl apply --prune` is in alpha, and backwards incompatible
changes might be introduced in subsequent releases.
-->
**警告：**  `kubectl apply --prone` はアルファです。今後のリリースでは後方互換が変更される場合があります。
{{< /warning >}}

{{< warning >}}
<!--
**Warning:** You must be careful when using this command, so that you
do not delete objects unintentionally.
-->
**警告：** このコマンドを使うにあたり、意図せずオブジェクトを削除しないよう、注意が必要です。
{{< /warning >}}

<!--
As an alternative to `kubectl delete`, you can use `kubectl apply` to identify objects to be deleted after their
configuration files have been removed from the directory. Apply with `--prune`
queries the API server for all objects matching a set of labels, and attempts
to match the returned live object configurations against the object
configuration files. If an object matches the query, and it does not have a
configuration file in the directory, and it has a `last-applied-configuration` annotation,
it is deleted.
-->
`kubectl delete` で削除するオブジェクトを指定する代わりに、ディレクトリから設定情報ファイルを削除してから  `kubectl apply` を使っての削除もできます。 `--prune` クエリを指定すると、API サーバはオブジェクト設定譲歩宇ファイルと現在のオブジェクト設定が一致するかどうかを比較し、すべてのオブジェクトから一致するラベルを返します。もしもオブジェクトとクエリ（問い合わせ）が一致し、ディレクトリ内の設定情報ファイルと一致しなければ、また `last-applied-configuration` アノテーション（注釈）を持っていれば削除されます。

{{< comment >}}
TODO(pwittrock): We need to change the behavior to prevent the user from running apply on subdirectories unintentionally.
{{< /comment >}}

<!--
```shell
kubectl apply -f <directory/> --prune -l <labels>
```
-->

```shell
kubectl apply -f <ディレクトリ/> --prune -l <ラベル>
```


{{< warning >}}
<!--
**Warning:** Apply with prune should only be run ag
ainst the root directory
containing the object configuration files. Running against sub-directories
can cause objects to be unintentionally deleted if they are returned
by the label selector query specified with `-l <labels>` and
do not appear in the subdirectory.
-->
**警告：**  prune の実行を指定すべきなのは、オブジェクト設定情報ファイルを含むルートディレクトリです。サブディレクトリに対して実行すると、 `-l <ラベル>` で指定したラベル選択クエリの結果が一致しないため、意図しないオブジェクトの削除を引き起こします。そのため、サブディレクトリでは実行しないでください。
{{< /warning >}}

<!--
## How to view an object
-->
## オブジェクトを表示するには {#how-to-viewan-object}

<!--
You can use `kubectl get` with `-o yaml` to view the configuration of a live object:
-->
`kubectl get` に `-o yaml` を指定して、稼働オブジェクトの設定情報を表示できます：

<!--
```shell
kubectl get -f <filename|url> -o yaml
```
-->
```shell
kubectl get -f <ファイル名|url> -o yaml
```

<!--
## How apply calculates differences and merges changes
-->
## 差分の確認と変更を反映するには {#how-apply-calculates-differences-and-merges-changes}

{{< caution >}}
<!--
**Caution:** A *patch* is an update operation that is scoped to specific
fields of an object instead of the entire object.
This enables updating only a specific set of fields on an object without
reading the object first.
-->
**注意：** *patch* （パッチ）とは更新作業であり、オブジェクト全体ではなく、オブジェクトの特定のフィールドを対象としています。これにより、オブジェクトをまず読み込むのではなく、オブジェクト上の特定フィールドにあるセットのみを更新できます。
{{< /caution >}}

<!--
When `kubectl apply` updates the live configuration for an object,
it does so by sending a patch request to the API server. The
patch defines updates scoped to specific fields of the live object
configuration. The `kubectl apply` command calculates this patch request
using the configuration file, the live configuration, and the
`last-applied-configuration` annotation stored in the live configuration.
-->
`kubectl apply` で

### Merge patch calculation

The `kubectl apply` command writes the contents of the configuration file to the
`kubectl.kubernetes.io/last-applied-configuration` annotation. This
is used to identify fields that have been removed from the configuration
file and need to be cleared from the live configuration. Here are the steps used
to calculate which fields should be deleted or set:

1. Calculate the fields to delete. These are the fields present in `last-applied-configuration` and missing from the configuration file.
2. Calculate the fields to add or set. These are the fields present in the configuration file whose values don't match the live configuration.

Here's an example. Suppose this is the configuration file for a Deployment object:

{{< code file="update_deployment.yaml" >}}

Also, suppose this is the live configuration for the same Deployment object:

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  annotations:
    # ...
    # note that the annotation does not contain replicas
    # because it was not updated through apply
    kubectl.kubernetes.io/last-applied-configuration: |
      {"apiVersion":"apps/v1","kind":"Deployment",
      "metadata":{"annotations":{},"name":"nginx-deployment","namespace":"default"},
      "spec":{"minReadySeconds":5,"selector":{"matchLabels":{"app":nginx}},"template":{"metadata":{"labels":{"app":"nginx"}},
      "spec":{"containers":[{"image":"nginx:1.7.9","name":"nginx",
      "ports":[{"containerPort":80}]}]}}}}
  # ...
spec:
  replicas: 2 # written by scale
  # ...
  minReadySeconds: 5
  selector:
    matchLabels:
      # ...
      app: nginx
  template:
    metadata:
      # ...
      labels:
        app: nginx
    spec:
      containers:
      - image: nginx:1.7.9
        # ...
        name: nginx
        ports:
        - containerPort: 80
      # ...
```

Here are the merge calculations that would be performed by `kubectl apply`:

1. Calculate the fields to delete by reading values from
   `last-applied-configuration` and comparing them to values in the
   configuration file. In this example, `minReadySeconds` appears in the
   `last-applied-configuration` annotation, but does not appear in the configuration file.
    **Action:** Clear `minReadySeconds` from the live configuration.
2. Calculate the fields to set by reading values from the configuration
   file and comparing them to values in the live configuration. In this example,
   the value of `image` in the configuration file does not match
    the value in the live configuration. **Action:** Set the value of `image` in the live configuration.
3. Set the `last-applied-configuration` annotation to match the value
   of the configuration file.
4. Merge the results from 1, 2, 3 into a single patch request to the API server.

Here is the live configuration that is the result of the merge:

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  annotations:
    # ...
    # The annotation contains the updated image to nginx 1.11.9,
    # but does not contain the updated replicas to 2
    kubectl.kubernetes.io/last-applied-configuration: |
      {"apiVersion":"apps/v1","kind":"Deployment",
      "metadata":{"annotations":{},"name":"nginx-deployment","namespace":"default"},
      "spec":{"selector":{"matchLabels":{"app":nginx}},"template":{"metadata":{"labels":{"app":"nginx"}},
      "spec":{"containers":[{"image":"nginx:1.11.9","name":"nginx",
      "ports":[{"containerPort":80}]}]}}}}
    # ...
spec:
  selector:
    matchLabels:
      # ...
      app: nginx
  replicas: 2 # Set by `kubectl scale`.  Ignored by `kubectl apply`.
  # minReadySeconds cleared by `kubectl apply`
  # ...
  template:
    metadata:
      # ...
      labels:
        app: nginx
    spec:
      containers:
      - image: nginx:1.11.9 # Set by `kubectl apply`
        # ...
        name: nginx
        ports:
        - containerPort: 80
        # ...
      # ...
    # ...
  # ...
```

{{< comment >}}
TODO(1.6): For 1.6, add the following bullet point to 1.

- clear fields explicitly set to null in the local object configuration file regardless of whether they appear in the last-applied-configuration
{{< /comment >}}

### How different types of fields are merged

How a particular field in a configuration file is merged with
the live configuration depends on the
type of the field. There are several types of fields:

- *primitive*: A field of type string, integer, or boolean.
  For example, `image` and `replicas` are primitive fields. **Action:** Replace.

- *map*, also called *object*: A field of type map or a complex type that contains subfields. For example, `labels`,
  `annotations`,`spec` and `metadata` are all maps. **Action:** Merge elements or subfields.

- *list*: A field containing a list of items that can be either primitive types or maps.
  For example, `containers`, `ports`, and `args` are lists. **Action:** Varies.

When `kubectl apply` updates a map or list field, it typically does
not replace the entire field, but instead updates the individual subelements.
For instance, when merging the `spec` on a Deployment, the entire `spec` is
not replaced. Instead the subfields of `spec`, such as `replicas`, are compared
and merged.

### Merging changes to primitive fields

Primitive fields are replaced or cleared.

{{< note >}}
**Note:** '-' is used for "not applicable" because the value is not used.
{{< /note >}}

| Field in object configuration file  | Field in live object configuration | Field in last-applied-configuration | Action                                    |
|-------------------------------------|------------------------------------|-------------------------------------|-------------------------------------------|
| Yes                                 | Yes                                | -                                   | Set live to configuration file value.  |
| Yes                                 | No                                 | -                                   | Set live to local configuration.           |
| No                                  | -                                  | Yes                                 | Clear from live configuration.            |
| No                                  | -                                  | No                                  | Do nothing. Keep live value.             |

### Merging changes to map fields

Fields that represent maps are merged by comparing each of the subfields or elements of the map:

{{< note >}}
**Note:** '-' is used for "not applicable" because the value is not used.
{{< /note >}}

| Key in object configuration file    | Key in live object configuration   | Field in last-applied-configuration | Action                           |
|-------------------------------------|------------------------------------|-------------------------------------|----------------------------------|
| Yes                                 | Yes                                | -                                   | Compare sub fields values.        |
| Yes                                 | No                                 | -                                   | Set live to local configuration.  |
| No                                  | -                                  | Yes                                 | Delete from live configuration.   |
| No                                  | -                                  | No                                  | Do nothing. Keep live value.     |

### Merging changes for fields of type list

Merging changes to a list uses one of three strategies:

* Replace the list.
* Merge individual elements in a list of complex elements.
* Merge a list of primitive elements.

The choice of strategy is made on a per-field basis.

#### Replace the list

Treat the list the same as a primitive field. Replace or delete the
entire list. This preserves ordering.

**Example:** Use `kubectl apply` to update the `args` field of a Container in a Pod. This sets
the value of `args` in the live configuration to the value in the configuration file.
Any `args` elements that had previously been added to the live configuration are lost.
The order of the `args` elements defined in the configuration file is
retained in the live configuration.

```yaml
# last-applied-configuration value
    args: ["a, b"]

# configuration file value
    args: ["a", "c"]

# live configuration
    args: ["a", "b", "d"]

# result after merge
    args: ["a", "c"]
```

**Explanation:** The merge used the configuration file value as the new list value.

#### Merge individual elements of a list of complex elements:

Treat the list as a map, and treat a specific field of each element as a key.
Add, delete, or update individual elements. This does not preserve ordering.

This merge strategy uses a special tag on each field called a `patchMergeKey`. The
`patchMergeKey` is defined for each field in the Kubernetes source code:
[types.go](https://git.k8s.io/api/core/v1/types.go#L2565)
When merging a list of maps, the field specified as the `patchMergeKey` for a given element
is used like a map key for that element.

**Example:** Use `kubectl apply` to update the `containers` field of a PodSpec.
This merges the list as though it was a map where each element is keyed
by `name`.

```yaml
# last-applied-configuration value
    containers:
    - name: nginx
      image: nginx:1.10
    - name: nginx-helper-a # key: nginx-helper-a; will be deleted in result
      image: helper:1.3
    - name: nginx-helper-b # key: nginx-helper-b; will be retained
      image: helper:1.3

# configuration file value
    containers:
    - name: nginx
      image: nginx:1.10
    - name: nginx-helper-b
      image: helper:1.3
    - name: nginx-helper-c # key: nginx-helper-c; will be added in result
      image: helper:1.3

# live configuration
    containers:
    - name: nginx
      image: nginx:1.10
    - name: nginx-helper-a
      image: helper:1.3
    - name: nginx-helper-b
      image: helper:1.3
      args: ["run"] # Field will be retained
    - name: nginx-helper-d # key: nginx-helper-d; will be retained
      image: helper:1.3

# result after merge
    containers:
    - name: nginx
      image: nginx:1.10
      # Element nginx-helper-a was deleted
    - name: nginx-helper-b
      image: helper:1.3
      args: ["run"] # Field was retained
    - name: nginx-helper-c # Element was added
      image: helper:1.3
    - name: nginx-helper-d # Element was ignored
      image: helper:1.3
```

**Explanation:**

- The container named "nginx-helper-a" was deleted because no container
  named "nginx-helper-a" appeared in the configuration file.
- The container named "nginx-helper-b" retained the changes to `args`
  in the live configuration. `kubectl apply` was able to identify
  that "nginx-helper-b" in the live configuration was the same
  "nginx-helper-b" as in the configuration file, even though their fields
  had different values (no `args` in the configuration file). This is
  because the `patchMergeKey` field value (name) was identical in both.
- The container named "nginx-helper-c" was added because no container
  with that name appeared in the live configuration, but one with
  that name appeared in the configuration file.
- The container named "nginx-helper-d" was retained because
  no element with that name appeared in the last-applied-configuration.

#### Merge a list of primitive elements

As of Kubernetes 1.5, merging lists of primitive elements is not supported.

{{< note >}}
**Note:** Which of the above strategies is chosen for a given field is controlled by
the `patchStrategy` tag in [types.go](https://git.k8s.io/api/core/v1/types.go#L2565)
If no `patchStrategy` is specified for a field of type list, then
the list is replaced.
{{< /note >}}

{{< comment >}}
TODO(pwittrock): Uncomment this for 1.6

- Treat the list as a set of primitives.  Replace or delete individual
  elements.  Does not preserve ordering.  Does not preserve duplicates.

**Example:** Using apply to update the `finalizers` field of ObjectMeta
keeps elements added to the live configuration.  Ordering of finalizers
is lost.
{{< /comment >}}

## Default field values

The API server sets certain fields to default values in the live configuration if they are
not specified when the object is created.

Here's a configuration file for a Deployment. The file does not specify `strategy`:

{{< code file="simple_deployment.yaml" >}}

Create the object using `kubectl apply`:

```shell
kubectl apply -f https://k8s.io/docs/concepts/overview/object-management-kubectl/simple_deployment.yaml
```

Print the live configuration using `kubectl get`:

```shell
kubectl get -f https://k8s.io/docs/concepts/overview/object-management-kubectl/simple_deployment.yaml -o yaml
```

The output shows that the API server set several fields to default values in the live
configuration. These fields were not specified in the configuration file.

```yaml
apiVersion: apps/v1
kind: Deployment
# ...
spec:
  selector:
    matchLabels:
      app: nginx
  minReadySeconds: 5
  replicas: 1 # defaulted by apiserver
  selector:
    matchLabels:
      app: nginx
  strategy:
    rollingUpdate: # defaulted by apiserver - derived from strategy.type
      maxSurge: 1
      maxUnavailable: 1
    type: RollingUpdate # defaulted apiserver
  template:
    metadata:
      creationTimestamp: null
      labels:
        app: nginx
    spec:
      containers:
      - image: nginx:1.7.9
        imagePullPolicy: IfNotPresent # defaulted by apiserver
        name: nginx
        ports:
        - containerPort: 80
          protocol: TCP # defaulted by apiserver
        resources: {} # defaulted by apiserver
        terminationMessagePath: /dev/termination-log # defaulted by apiserver
      dnsPolicy: ClusterFirst # defaulted by apiserver
      restartPolicy: Always # defaulted by apiserver
      securityContext: {} # defaulted by apiserver
      terminationGracePeriodSeconds: 30 # defaulted by apiserver
# ...
```

In a patch request, defaulted fields are not re-defaulted unless they are explicitly cleared
as part of a patch request. This can cause unexpected behavior for
fields that are defaulted based
on the values of other fields. When the other fields are later changed,
the values defaulted from them will not be updated unless they are
explicitly cleared.

For this reason, it is recommended that certain fields defaulted
by the server are explicitly defined in the configuration file, even
if the desired values match the server defaults. This makes it
easier to recognize conflicting values that will not be re-defaulted
by the server.

**Example:**

```yaml
# last-applied-configuration
spec:
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
      - name: nginx
        image: nginx:1.7.9
        ports:
        - containerPort: 80

# configuration file
spec:
  strategy:
    type: Recreate # updated value
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
      - name: nginx
        image: nginx:1.7.9
        ports:
        - containerPort: 80

# live configuration
spec:
  strategy:
    type: RollingUpdate # defaulted value
    rollingUpdate: # defaulted value derived from type
      maxSurge : 1
      maxUnavailable: 1
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
      - name: nginx
        image: nginx:1.7.9
        ports:
        - containerPort: 80

# result after merge - ERROR!
spec:
  strategy:
    type: Recreate # updated value: incompatible with rollingUpdate
    rollingUpdate: # defaulted value: incompatible with "type: Recreate"
      maxSurge : 1
      maxUnavailable: 1
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
      - name: nginx
        image: nginx:1.7.9
        ports:
        - containerPort: 80
```

**Explanation:**

1. The user creates a Deployment without defining `strategy.type`.
2. The server defaults `strategy.type` to `RollingUpdate` and defaults the
   `strategy.rollingUpdate` values.
3. The user changes `strategy.type` to `Recreate`. The `strategy.rollingUpdate`
   values remain at their defaulted values, though the server expects them to be cleared.
   If the `strategy.rollingUpdate` values had been defined initially in the configuration file,
   it would have been more clear that they needed to be deleted.
4. Apply fails because `strategy.rollingUpdate` is not cleared. The `strategy.rollingupdate`
   field cannot be defined with a `strategy.type` of `Recreate`.

Recommendation: These fields should be explicitly defined in the object configuration file:

- Selectors and PodTemplate labels on workloads, such as Deployment, StatefulSet, Job, DaemonSet,
  ReplicaSet, and ReplicationController
- Deployment rollout strategy

### How to clear server-defaulted fields or fields set by other writers

As of Kubernetes 1.5, fields that do not appear in the configuration file cannot be
cleared by a merge operation. Here are some workarounds:

Option 1: Remove the field by directly modifying the live object.

{{< note >}}
**Note:** As of Kubernetes 1.5, `kubectl edit` does not work with `kubectl apply`.
Using these together will cause unexpected behavior.
{{< /note >}}

Option 2: Remove the field through the configuration file.

1. Add the field to the configuration file to match the live object.
1. Apply the configuration file; this updates the annotation to include the field.
1. Delete the field from the configuration file.
1. Apply the configuration file; this deletes the field from the live object and annotation.

{{< comment >}}
TODO(1.6): Update this with the following for 1.6

Fields that do not appear in the configuration file can be cleared by
setting their values to `null` and then applying the configuration file.
For fields defaulted by the server, this triggers re-defaulting
the values.
{{< /comment >}}

## How to change ownership of a field between the configuration file and direct imperative writers

These are the only methods you should use to change an individual object field:

- Use `kubectl apply`.
- Write directly to the live configuration without modifying the configuration file:
for example, use `kubectl scale`.

### Changing the owner from a direct imperative writer to a configuration file

Add the field to the configuration file. For the field, discontinue direct updates to
the live configuration that do not go through `kubectl apply`.

### Changing the owner from a configuration file to a direct imperative writer

As of Kubernetes 1.5, changing ownership of a field from a configuration file to
an imperative writer requires manual steps:

- Remove the field from the configuration file.
- Remove the field from the `kubectl.kubernetes.io/last-applied-configuration` annotation on the live object.

## Changing management methods

Kubernetes objects should be managed using only one method at a time.
Switching from one method to another is possible, but is a manual process.

{{< note >}}
**Note:** It is OK to use imperative deletion with declarative management.
{{< /note >}}

{{< comment >}}
TODO(pwittrock): We need to make using imperative commands with
declarative object configuration work so that it doesn't write the
fields to the annotation, and instead.  Then add this bullet point.

- using imperative commands with declarative configuration to manage where each manages different fields.
{{< /comment >}}

### Migrating from imperative command management to declarative object configuration

Migrating from imperative command management to declarative object
configuration involves several manual steps:

1. Export the live object to a local configuration file:

     ```shell
     kubectl get <kind>/<name> -o yaml --export > <kind>_<name>.yaml
     ```

1. Manually remove the `status` field from the configuration file.

    {{< note >}}**Note:** This step is optional, as `kubectl apply` does not update the status field
    even if it is present in the configuration file.{{< /note >}}

1. Set the `kubectl.kubernetes.io/last-applied-configuration` annotation on the object:

    ```shell
    kubectl replace --save-config -f <kind>_<name>.yaml
    ```

1. Change processes to use `kubectl apply` for managing the object exclusively.

{{< comment >}}
TODO(pwittrock): Why doesn't export remove the status field?  Seems like it should.
{{< /comment >}}

### Migrating from imperative object configuration to declarative object configuration

1. Set the `kubectl.kubernetes.io/last-applied-configuration` annotation on the object:

    ```shell
    kubectl replace --save-config -f <kind>_<name>.yaml
    ```

1. Change processes to use `kubectl apply` for managing the object exclusively.

## Defining controller selectors and PodTemplate labels

{{< warning >}}
**Warning:** Updating selectors on controllers is strongly discouraged.
{{< /warning >}}

The recommended approach is to define a single, immutable PodTemplate label
used only by the controller selector with no other semantic meaning.

**Example:**

```yaml
selector:
  matchLabels:
      controller-selector: "extensions/v1beta1/deployment/nginx"
template:
  metadata:
    labels:
      controller-selector: "extensions/v1beta1/deployment/nginx"
```

## Known Issues

* Prior to Kubernetes 1.6, `kubectl apply` did not support operating on objects stored in a
  [custom resource](/docs/concepts/api-extension/custom-resources/).
  For these cluster versions, you should instead use [imperative object configuration](/docs/concepts/overview/object-management-kubectl/imperative-config/).
{{% /capture %}}

{{% capture whatsnext %}}
- [Managing Kubernetes Objects Using Imperative Commands](/docs/concepts/overview/object-management-kubectl/imperative-command/)
- [Imperative Management of Kubernetes Objects Using Configuration Files](/docs/concepts/overview/object-management-kubectl/imperative-config/)
- [Kubectl Command Reference](/docs/reference/generated/kubectl/kubectl/)
- [Kubernetes API Reference](/docs/reference/generated/kubernetes-api/{{< param "version" >}}/)
{{% /capture %}}


