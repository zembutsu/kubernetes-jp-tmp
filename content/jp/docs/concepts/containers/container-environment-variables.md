---
reviewers:
- mikedanese
- thockin
title: コンテナ環境変数
content_template: templates/concept
weight: 20
---

{{% capture overview %}}
<!--
This page describes the resources available to Containers in the Container environment. 
-->
このページはコンテナで利用できるコンテナ環境変数について説明します。
{{% /capture %}}

{{< toc >}}

{{% capture body %}}

<!--
## Container environment
-->
## コンテナ環境（Container environment） {#container-environment}

<!--
The Kubernetes Container environment provides several important resources to Containers:
-->
Kubernetes コンテナ環境変数は、コンテナに対して複数の重要なリソースを提供します：

<!--
* A filesystem, which is a combination of an [image](/docs/concepts/containers/images/) and one or more [volumes](/docs/concepts/storage/volumes/).
* Information about the Container itself.
* Information about other objects in the cluster.
-->
- [イメージ](/jp/docs/concepts/containers/images/) と１つまたは複数の [ボリューム](/jp/docs/concepts/storage/volumes/) を組み合わせたファイルシステム。
- コンテナ自身に関する情報。
- クラスタ内の他のオブジェクトに関する情報。

<!--
### Container information
-->
### コンテナ情報 {#container-information}

<!--
The *hostname* of a Container is the name of the Pod in which the Container is running.
It is available through the `hostname` command or the
[`gethostname`](http://man7.org/linux/man-pages/man2/gethostname.2.html)
function call in libc.
-->
コンテナの *ホスト名* とは、コンテナを実行しているポッドの名前です。これは `hostname` コマンドを通してか、 libc の関数コール [`gethostname`](http://man7.org/linux/man-pages/man2/gethostname.2.html) で取得できます。

<!--
The Pod name and namespace are available as environment variables through the
[downward API](/docs/tasks/inject-data-application/downward-api-volume-expose-pod-information/).
-->
[downward API](/jp/docs/tasks/inject-data-application/downward-api-volume-expose-pod-information/) を通して、ポッド名と名前空間は環境変数として利用できます。

<!--
User defined environment variables from the Pod definition are also available to the Container,
as are any environment variables specified statically in the Docker image.
-->
また、ユーザはポッドの定義時にもコンテナに対する環境変数を定義できます。また、Docker イメージで指定されている環境変数と共に利用できます。

<!--
### Cluster information
-->
### クラスタ情報 {#cluster-infomation}

<!--
A list of all services that were running when a Container was created is available to that Container as environment variables.
Those environment variables match the syntax of Docker links.
-->

コンテナ作成時に実行中のサービス一覧は、コンテナの環境変数として利用できます。これら環境変数の利用形態は、 Docker の link 構文に相当します。

<!--
For a service named *foo* that maps to a Container named *bar*,
the following variables are defined:
-->
サービス名 *foo*  がコンテナ名 *bar* に割り当てられている場合、以下の環境変数が定義されます：

<!--
```shell
FOO_SERVICE_HOST=<the host the service is running on>
FOO_SERVICE_PORT=<the port the service is running on>
```
-->
```shell
FOO_SERVICE_HOST=<サービスを実行しているホスト>
FOO_SERVICE_PORT=<サービスを実行しているポート>
```

<!--
Services have dedicated IP addresses and are available to the Container via DNS,
if [DNS addon](http://releases.k8s.io/{{< param "githubbranch" >}}/cluster/addons/dns/) is enabled. 
-->
[DNS アドオン](http://releases.k8s.io/{{< param "githubbranch" >}}/cluster/addons/dns/) を有効化すると、サービスは専用の IP アドレスをもち、DNS を通してコンテナにアクセスできるようになります。

{{% /capture %}}

{{% capture whatsnext %}}

<!--
* Learn more about [Container lifecycle hooks](/docs/concepts/containers/container-lifecycle-hooks/).
* Get hands-on experience
  [attaching handlers to Container lifecycle events](/docs/tasks/configure-pod-container/attach-handler-lifecycle-event/).
-->
* [コンテナ・ライフサイクル・フック](/jp/docs/concepts/containers/container-lifecycle-hooks/) について学ぶ。
* [コンテナ・ライフサイクル・イベントを取り扱う](/docs/tasks/configure-pod-container/attach-handler-lifecycle-event/) ハンズオン練習。


{{% /capture %}}


