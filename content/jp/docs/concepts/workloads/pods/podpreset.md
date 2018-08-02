---
reviewers:
- jessfraz
title: ポッド・プリセット
content_template: templates/concept
weight: 50
---

{{% capture overview %}}
<!--
This page provides an overview of PodPresets, which are objects for injecting
certain information into pods at creation time. The information can include
secrets, volumes, volume mounts, and environment variables.
-->
このページは PodPreset（ポッド・プリセット）の概要を紹介します。PodPreset とは、特定の情報をポッド作成時に投入するオブジェクトです。情報に含められるのはシークレット（機密情報）、ボリューム、ボリューム・マウント、環境変数です。
{{% /capture %}}

{{< toc >}}

{{% capture body %}}
<!--
## Understanding Pod Presets
-->
## ポッド・プリセットの理解 {#understanding-pod-presets}

<!--
A `Pod Preset` is an API resource for injecting additional runtime requirements
into a Pod at creation time.
You use [label selectors](/docs/concepts/overview/working-with-objects/labels/#label-selectors)
to specify the Pods to which a given Pod Preset applies.
-->
`PodPreset` （ポッド・プリセット）とは API リソースであり、ポッドの作成時に追加のランタイム環境変数を投入します。[ラベル・セレクタ](/jp/docs/concepts/overview/working-with-objects/labels/#label-selectors) を使い、特定のポッドに対して適用するポッド・プリセットを指定します。

<!--
Using a Pod Preset allows pod template authors to not have to explicitly provide 
all information for every pod. This way, authors of pod templates consuming a
specific service do not need to know all the details about that service.
-->
ポッド・プリセットを使うと、ポッド・テンプレートの作者はポッドごとに全ての情報を明示する必要がなくなります。ポッド・テンプレートの作者がこの方法を使うと、詳細な記述が不要な情報を対象となるサービスから削除できます。

<!--
For more information about the background, see the [design proposal for PodPreset](https://git.k8s.io/community/contributors/design-proposals/service-catalog/pod-preset.md).
-->
背景に関する詳しい情報は、 [PodPreset の設計提案](https://git.k8s.io/community/contributors/design-proposals/service-catalog/pod-preset.md) をご覧ください。

<!--
## How It Works
-->
## 動作内容 {#how-it-works}

<!--
Kubernetes provides an admission controller (`PodPreset`) which, when enabled,
applies Pod Presets to incoming pod creation requests.
When a pod creation request occurs, the system does the following:
-->
Kubernetes が提供する入場制御（アドミッション・コントローラ：admission controller）（`PodPreset`）は、有効にすると、ポッド作成要求があれば、ポッド・プリセットを作成します。ポッド作成要求が発生すると、システムは以下の処理をします：

<!--
1. Retrieve all `PodPresets` available for use.
1. Check if the label selectors of any `PodPreset` matches the labels on the
   pod being created.
1. Attempt to merge the various resources defined by the `PodPreset` into the
   Pod being created.
1. On error, throw an event documenting the merge error on the pod, and create
   the pod _without_ any injected resources from the `PodPreset`.
1. Annotate the resulting modified Pod spec to indicate that it has been
   modified by a `PodPreset`. The annotation is of the form
   `podpreset.admission.kubernetes.io/podpreset-<pod-preset name>: "<resource version>"`.
-->
1. 使うために、全ての利用可能な `PodPresets` を集めます。
1. ラベル・セレクタは、作成しようとしているポット上に `PodPreset` に一致するラベルがあるかどうか調べます。
1. 作成しようとしているポッド内に、 `PodPreset` にある様々なリソース定義の統合（マージ）を試みます。
1. エラーがあれば、ポッドに対する統合エラーがイベントに記録され、ポッドには `PodPreset` で指定したリソース投入を行いません。
1. 変更した Pod spec を目立つようにアノテーション（注釈）をつけます。これは `PodPreset` いよって変更されたものです。アノテーションの形式は    `podpreset.admission.kubernetes.io/podpreset-<ポッド・プリセット名>: "<リソース・バージョン>"` です。

<!--
Each Pod can be matched by zero or more Pod Presets; and each `PodPreset` can be
applied to zero or more pods. When a `PodPreset` is applied to one or more
Pods, Kubernetes modifies the Pod Spec. For changes to `Env`, `EnvFrom`, and
`VolumeMounts`, Kubernetes modifies the container spec for all containers in
the Pod; for changes to `Volume`, Kubernetes modifies the Pod Spec.
-->
各ポッドがゼロもしくは PodPreset を複数もつ場合があります。つまり、各 `PodPreset` にはゼロもしくは複数のポッドが適用される可能性があります。もし `PodPreset` が１つもしくは複数のポッドに割り当てられる場合、Kubernetes はポッド内にある全てのコンテナのコンテナ spec を変更します。この変更には `Volume` も含みますので、Kubernetes はポッドの spec を変更します。

{{< note >}}
<!--
**Note:** A Pod Preset is capable of modifying the `.spec.containers` field in a
Pod spec when appropriate. *No* resource definition from the Pod Preset will be 
applied to the `initContainers` field.
-->
**メモ：** ポッド・プリセットは然るべき時にポッド spec の `.spec.containers` フィールドを変更できる能力があります。ポッド・プリセットに *No* リソースを定義すると（訳者注：何も定義しなければ）、ポッド・プリセットは `initContainers` フィールドに適用されます。
{{< /note >}}

<!--
### Disable Pod Preset for a Specific Pod
-->
### 特定のポッドに対するポッド・プリセットを無効化 {#disable-pod-preset-for-a-specific-pod}

<!--
There may be instances where you wish for a Pod to not be altered by any Pod
Preset mutations. In these cases, you can add an annotation in the Pod Spec
of the form: `podpreset.admission.kubernetes.io/exclude: "true"`.
-->
ポッドに対しては、通知の無いあらゆるポッドのプリセット変更を行いたくない場合があるでしょう。そのような場合、ポッド Spec にアノテーション（注釈）を `podpreset.admission.kubernetes.io/exclude: "true"` のような形式で追加できます。

<!--
## Enable Pod Preset
-->
## ポッド・プリセットの有効化 {#enable-pod-preset}

<!--
In order to use Pod Presets in your cluster you must ensure the following:
-->
ポッド・プリセットをクラスタ内で使うためには、以下の手順を確実に行う必要があります

<!--
1.  You have enabled the API type `settings.k8s.io/v1alpha1/podpreset`. For
    example, this can be done by including `settings.k8s.io/v1alpha1=true` in
    the `--runtime-config` option for the API server. 
1.  You have enabled the admission controller `PodPreset`. One way to doing this
    is to include `PodPreset` in the `--enable-admission-plugins` option value specified
    for the API server.
1.  You have defined your Pod Presets by creating `PodPreset` objects in the
    namespace you will use.
-->
1. API タイプ `settings.k8s.io/v1alpha1/podpreset` を有効化します。たとえば、API サーバに対するオプションとして  `--runtime-config` に `settings.k8s.io/v1alpha1=true`  を指定します。
1. アドミッション・コントローラ `PodPreset` を有効化します。有効化するための１つの方法は、API サーバに対するオプション `--enable-admission-plugins` の値に `PodPreset` を含めます。
1. 使用する名前空間内で `PodPreset` オブジェクトの作成時に、ポッド・プリセットをを定義します。

{{% /capture %}}

{{% capture whatsnext %}}
<!--
* [Injecting data into a Pod using PodPreset](/docs/tasks/inject-data-application/podpreset/)
-->
* [PodPreset を使ってポッドにデータを投入](/jp/docs/tasks/inject-data-application/podpreset/)

{{% /capture %}}


