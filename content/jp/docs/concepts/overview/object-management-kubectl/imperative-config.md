---
title: Kubernetes オブジェクト管理に命令型の設定ファイルを使う
content_template: templates/concept
weight: 30
---

{{% capture overview %}}
<!--
Kubernetes objects can be created, updated, and deleted by using the `kubectl`
command-line tool along with an object configuration file written in YAML or JSON.
This document explains how to define and manage objects using configuration files.
-->
`kubectl` コマンドライン・ツールを使うと、YAML または JSON で書かれたオブジェクト設定情報の記述に従い、Kubernetes オブジェクトを作成、更新、削除できます。このドキュメントは設定情報ファイルを使ったオブジェクトの定義と管理方法を説明します。


{{% /capture %}}

{{% capture body %}}

<!--
## Trade-offs
-->
## トレードオフ {#trade-offs}

<!--
The `kubectl` tool supports three kinds of object management:
-->
`kubectl` ツールは３種類のオブジェクト管理をサポートしています：

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
## How to create objects
-->
## オブジェクトの作成方法 {#how-to-create-objects}

<!--
You can use `kubectl create -f` to create an object from a configuration file.
Refer to the [kubernetes API reference](/docs/reference/generated/kubernetes-api/{{< param "version" >}}/)
for details.
-->
`kubectl create -f` を使い、設定情報ファイルにあるオブジェクトを作成できます。詳細は [kubernetes API リファレンス（参考情報）](/jp/docs/reference/generated/kubernetes-api/{{< param "version" >}}/) をご覧ください。

<!--
- `kubectl create -f <filename|url>`
-->
- `kubectl create -f <ファイル名|url>`

<!--
## How to update objects
-->
## オブジェクトの更新方法 {#how-to-update-objects}

{{< warning >}}
<!--
**Warning:** Updating objects with the `replace` command drops all
parts of the spec not specified in the configuration file.  This
should not be used with objects whose specs are partially managed
by the cluster, such as Services of type `LoadBalancer`, where
the `externalIPs` field is managed independently from the configuration
file.  Independently managed fields must be copied to the configuration
file to prevent `replace` from dropping them.
-->
**警告：** オブジェクトの更新に `replace` （置換）コマンドを使うと、設定情報ファイル内に spec （仕様）の記述がないパーツをすべて削除します。そのため、サービス型 `LoadBalancer` のようなクラスタによって管理される部分を specs のオブジェクトで管理すべきではないでしょう。この `externalIPs` フィールドは設定情報ファイルとは独立して管理すべきです。独立して管理するフィールドは `replace` によって削除されるのを防ぐため、別の設定情報ファイルにコピーしておく必要があります。
{{< /warning >}}

<!--
You can use `kubectl replace -f` to update a live object according to a
configuration file.
-->
`kubectl replace -f` を使い、設定情報ファイルに従って稼働中のオブジェクトを更新します。

<!--
- `kubectl replace -f <filename|url>`
-->
- `kubectl replace -f <ファイル名|url>`

<!--
## How to delete objects
-->
## オブジェクトの削除方法 {#how-to-delete-object}

<!--
You can use `kubectl delete -f` to delete an object that is described in a
configuration file.
-->
`kubectl delete -f` を使って設定情報ファイルに記述があるオブジェクトを削除します。

<!--
- `kubectl delete -f <filename|url>`
-->
- `kubectl delete -f <ファイル名|url>`

<!--
## How to view an object
-->
## オブジェクトを表示するには

<!--
You can use `kubectl get -f` to view information about an object that is
described in a configuration file.
-->
 `kubectl get -f` を使い、設定情報ファイル内に記述があるオブジェクトの情報を表示できます。

<!--
- `kubectl get -f <filename|url> -o yaml`
-->
- `kubectl get -f <ファイル名|url> -o yaml`

<!--
The `-o yaml` flag specifies that the full object configuration is printed.
Use `kubectl get -h` to see a list of options.
-->
`-o yaml` フラグを指定すると、オブジェクト設定情報のすべてを表示します。 `kubectl get -h` でオプションの一覧を表示します。

<!--
## Limitations
-->
## 制限事項 {#limitations}

<!--
The `create`, `replace`, and `delete` commands work well when each object's
configuration is fully defined and recorded in its configuration
file. However when a live object is updated, and the updates are not merged
into its configuration file, the updates will be lost the next time a `replace`
is executed. This can happen if a controller, such as
a HorizontalPodAutoscaler, makes updates directly to a live object. Here's
an example:
-->
`create` 、 `replace` 、 `delete` コマンドは各オブジェクトの設定情報を完全に定義し、それを設定情報ファイルに記録している場合のみ動作します。しかし、稼働中のオブジェクトが更新するか、更新したものを設定情報ファイルに反映しなければ、更新を次回実行すると `replace` （置換）が処理されます。これによって起こるのは、 HorizontalPodAutoscaler（水平ポッド・スケーラ）のようなコントローラは稼働中のオブジェクトに対して直接更新を試みます。以下は例です：

<!--
1. You create an object from a configuration file.
1. Another source updates the object by changing some field.
1. You replace the object from the configuration file. Changes made by
the other source in step 2 are lost.
-->
1. オブジェクトを構成情報ファイルから作成します。
1. （ファイルではない）別の方法を使い、オブジェクトのフィールドをいくつか変更します。
1. 設定情報ファイルでオブジェクトを置き換えます（replace）。手順２で作成した別の方法による変更は消えます。

<!--
If you need to support multiple writers to the same object, you can use
`kubectl apply` to manage the object.
-->
同じオブジェクトを複数回書けるようにするには、オブジェクトの管理に `kubectl apply` で管理する必要があります。

<!--
## Creating and editing an object from a URL without saving the configuration
-->
## 設定情報を保存せずに URL でオブジェクトの作成と更新{#creating-and-editing-an-object-from-a-url-without-saving-the-configuration}

<!--
Suppose you have the URL of an object configuration file. You can use
`kubectl create --edit` to make changes to the configuration before the
object is created. This is particularly useful for tutorials and tasks
that point to a configuration file that could be modified by the reader.
-->
オブジェクト構成情報ファイルの URL があると想定します。`kubectl create --edit` を使い、巣でイン作成したオブジェクトに対する設定情報を変更可能です。これが特に役立つのはチュートリアルとタスクです。読み手に対し、設定情報ファイルを使って変更内容を指示できます。

```sh
kubectl create -f <url> --edit
```

<!--
## Migrating from imperative commands to imperative object configuration
-->
## 命令型コマンドから命令型オブジェクト設定への移行 {#migrating-from-imperative-commands-to-imperative-object-configuration}

<!--
Migrating from imperative commands to imperative object configuration involves
several manual steps.
-->
命令型コマンドから命令型オブジェクト設定に移行するには、いくつかの手動手順があります。

<!--
1. Export the live object to a local object configuration file:
```sh
kubectl get <kind>/<name> -o yaml --export > <kind>_<name>.yaml
```
-->
1. 稼働中のオブジェクトを設定情報ファイルに出力（export）します。
```sh
kubectl get <種類>/<名前> -o yaml --export > <種類>_<名前>.yaml
```

<!--
1. Manually remove the status field from the object configuration file.
-->
1. オブジェクト設定情報ファイルから、手動で status フィールドを削除します。

<!--
1. For subsequent object management, use `replace` exclusively.
```sh
kubectl replace -f <kind>_<name>.yaml
```
-->
1. 以降のオブジェクト管理には `replace` のみを使います。
```sh
kubectl replace -f <種類>_<名前>.yaml
```

<!--
## Defining controller selectors and PodTemplate labels
-->

## コントローラ・セレクタとポッド・テンプレートの定義 {#defining-controller-selectors-and-podtemplate-labels}

{{< warning >}}
<!--
**Warning:** Updating selectors on controllers is strongly discouraged.
-->
**警告：** コントローラ上のセレクタの更新は、非常に非推奨です。
{{< /warning >}}

<!--
The recommended approach is to define a single, immutable PodTemplate label
used only by the controller selector with no other semantic meaning.
-->
推奨する手法は、全く手を加えない（イミュータブルな）ポッド・テンプレート（PodTemplate）ラベルを１つだけ定義し、何も意味を持たないコントローラ・セレクタのために使う方法です。

<!--
Example label:
-->
サンプル・ラベル：

```yaml
selector:
  matchLabels:
      controller-selector: "extensions/v1beta1/deployment/nginx"
template:
  metadata:
    labels:
      controller-selector: "extensions/v1beta1/deployment/nginx"
```

{{% /capture %}}

{{% capture whatsnext %}}
<!--
- [Managing Kubernetes Objects Using Imperative Commands](/docs/concepts/overview/object-management-kubectl/imperative-command/)
- [Managing Kubernetes Objects Using Object Configuration (Declarative)](/docs/concepts/overview/object-management-kubectl/declarative-config/)
- [Kubectl Command Reference](/docs/reference/generated/kubectl/kubectl/)
- [Kubernetes API Reference](/docs/reference/generated/kubernetes-api/{{< param "version" >}}/)
-->
- [Kubernetes オブジェクトを宣言型コマンドで管理](/jp/docs/concepts/overview/object-management-kubectl/imperative-command/)
- [Kubernetes オブジェクトを設定叙法（宣言型）で管理](/jp/docs/concepts/overview/object-management-kubectl/declarative-config/)
- [Kubectl コマンドリファレンス（参考情報）](/jp/docs/reference/generated/kubectl/kubectl/)
- [Kubernetes API リファレンス（参考情報）](/jp/docs/reference/generated/kubernetes-api/{{< param "version" >}}/)

{{% /capture %}}


