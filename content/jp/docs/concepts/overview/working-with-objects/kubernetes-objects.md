---
title: Kubernetes オブジェクトの理解
content_template: templates/concept
weight: 10
---

{{% capture overview %}}
<!--
This page explains how Kubernetes objects are represented in the Kubernetes API, and how you can express them in `.yaml` format.
-->
このページは Kubernetes オブジェクトが Kubernetes API でどのような役割を持つのかを説明します。そして、これらを `.yaml` 形式でも表現できます。
{{% /capture %}}

{{% capture body %}}
<!--
## Understanding Kubernetes Objects
-->
## Kubernetes オブジェクトの理解 {#understanding-kubernetes-object}

<!--
*Kubernetes Objects* are persistent entities in the Kubernetes system. Kubernetes uses these entities to represent the state of your cluster. Specifically, they can describe:
-->
Kubernetes システム内では、*Kubernetes オブジェクト* は永続的な存在です。Kubernetes はこれらの存在を使い、クラスタの状態を説明します。特に、これらで説明できるのは：

<!--
* What containerized applications are running (and on which nodes)
* The resources available to those applications
* The policies around how those applications behave, such as restart policies, upgrades, and fault-tolerance
-->
* 実行中のコンテナ化されたアプリケーションは何か（そして、どのノードか）
* これらアプリケーションが利用可能な資源（リソース）
* 再起動ポリシー、更新、耐障害性（fault-tolerance）のようなアプリケーションがどのような挙動なのかに関連するポリシー。

<!--
A Kubernetes object is a "record of intent"--once you create the object, the Kubernetes system will constantly work to ensure that object exists. By creating an object, you're effectively telling the Kubernetes system what you want your cluster's workload to look like; this is your cluster's **desired state**.
-->
Kubernetes オブジェクトとは "目的の記録"（record of intent）です。ひとたびオブジェクトを作成したら、Kubernetes システムはオブジェクトが確実に存在し続けるよう、絶え間なく動作します。オブジェクトの作成によって、Kubernetes システムに対してクラスタの作業負荷（ワークロード）をどうしたいのかを効率的に伝えられます。これがクラスタの **期待状態（desired state）** です。

<!--
To work with Kubernetes objects--whether to create, modify, or delete them--you'll need to use the [Kubernetes API](/docs/concepts/overview/kubernetes-api/). When you use the `kubectl` command-line interface, for example, the CLI makes the necessary Kubernetes API calls for you. You can also use the Kubernetes API directly in your own programs using one of the [Client Libraries](/docs/reference/using-api/client-libraries/).
-->
Kubernetes オブジェクトを取り扱うには、たとえばオブジェクトを作成、変更、削除をするには [Kubernetes API](/jp/docs/concepts/overview/kubernetes-api/) を使う必要があります。例として `kubectl` コマンドライン・インターフェースを使うとすると、CLI は必要になる Kubernetes API コールを作成します。また、[クライアント・ライブラリ](/jp/docs/reference/using-api/client-libraries/)のどれかを使う自分のプログラムを用い、Kubernetes API で直接の操作も可能です。

<!--
### Object Spec and Status
-->
### オブジェクト仕様と状態 {#object-spec-and-status}

<!--
Every Kubernetes object includes two nested object fields that govern the object's configuration: the object *spec* and the object *status*. The *spec*, which you must provide, describes your *desired state* for the object--the characteristics that you want the object to have. The *status* describes the *actual state* of the object, and is supplied and updated by the Kubernetes system. At any given time, the Kubernetes Control Plane actively manages an object's actual state to match the desired state you supplied.
-->
すべての Kubernetes オブジェクトには２つの入れ子になった（ネストした）オブジェクト・フィールドを含んでおり、これでオブジェクトの設定を管理します。これはオブジェクトの *spec（仕様）* とオブジェクトの *status（状態）* です。 *spec* とは、指定が必須であり、オブジェクトに対する *期待状態（desired state）* を記述します。 *status* にはオブジェクトの *実際の状態（actual state）* を記述し、こちらは Kubernetes システムによって提供および更新されます。あらゆる時間があれば、Kubernetes はコントロール・プレーンの管理を活性化し、オブジェクトの実際の状態が、指定した期待状態と一致するようにします。

<!--
For example, a Kubernetes Deployment is an object that can represent an application running on your cluster. When you create the Deployment, you might set the Deployment spec to specify that you want three replicas of the application to be running. The Kubernetes system reads the Deployment spec and starts three instances of your desired application--updating the status to match your spec. If any of those instances should fail (a status change), the Kubernetes system responds to the difference between spec and status by making a correction--in this case, starting a replacement instance.
-->
たとえば、Kubernetes

<!--
For more information on the object spec, status, and metadata, see the [Kubernetes API Conventions](https://git.k8s.io/community/contributors/devel/api-conventions.md).
-->

### Describing a Kubernetes Object

When you create an object in Kubernetes, you must provide the object spec that describes its desired state, as well as some basic information about the object (such as a name). When you use the Kubernetes API to create the object (either directly or via `kubectl`), that API request must include that information as JSON in the request body. **Most often, you provide the information to `kubectl` in a .yaml file.** `kubectl` converts the information to JSON when making the API request.

Here's an example `.yaml` file that shows the required fields and object spec for a Kubernetes Deployment:

{{< code file="nginx-deployment.yaml" >}}

One way to create a Deployment using a `.yaml` file like the one above is to use the [`kubectl create`](/docs/reference/generated/kubectl/kubectl-commands#create) command in the `kubectl` command-line interface, passing the `.yaml` file as an argument. Here's an example:

```shell
$ kubectl create -f https://k8s.io/docs/concepts/overview/working-with-objects/nginx-deployment.yaml --record
```

The output is similar to this:

```shell
deployment "nginx-deployment" created
```

### Required Fields

In the `.yaml` file for the Kubernetes object you want to create, you'll need to set values for the following fields:

* `apiVersion` - Which version of the Kubernetes API you're using to create this object
* `kind` - What kind of object you want to create
* `metadata` - Data that helps uniquely identify the object, including a `name` string, UID, and optional `namespace`

You'll also need to provide the object `spec` field. The precise format of the object `spec` is different for every Kubernetes object, and contains nested fields specific to that object. The [Kubernetes API Reference](/docs/reference/generated/kubernetes-api/{{< param "version" >}}/) can help you find the spec format for all of the objects you can create using Kubernetes.
For example, the `spec` format for a `Pod` object can be found
[here](/docs/reference/generated/kubernetes-api/{{< param "version" >}}/#podspec-v1-core),
and the `spec` format for a `Deployment` object can be found
[here](/docs/reference/generated/kubernetes-api/{{< param "version" >}}/#deploymentspec-v1-apps).

{{% /capture %}}

{{% capture whatsnext %}}
* Learn about the most important basic Kubernetes objects, such as [Pod](/docs/concepts/workloads/pods/pod-overview/).
{{% /capture %}}


