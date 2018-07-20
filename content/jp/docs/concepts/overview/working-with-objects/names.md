---
reviewers:
- mikedanese
- thockin
title: 名前
content_template: templates/concept
weight: 20
---

{{% capture overview %}}
<!--
All objects in the Kubernetes REST API are unambiguously identified by a Name and a UID.
-->
Kubernetes REST API のすべてのオブジェクトは、名前と UID によって明確に識別します。

<!--
For non-unique user-provided attributes, Kubernetes provides [labels](/docs/user-guide/labels) and [annotations](/docs/concepts/overview/working-with-objects/annotations/).
-->
ユニークではないユーザが指定する属性のためには、Kubernetes は [ラベル（label）](/jp/docs/user-guide/labels) と [注釈（annotations）](/jp/docs/concepts/overview/working-with-objects/annotations/) を提供しています

<!--
See the [identifiers design doc](https://git.k8s.io/community/contributors/design-proposals/architecture/identifiers.md) for the precise syntax rules for Names and UIDs.
-->
名前と UID に対する厳密な文法規則は [identifiers design doc](https://git.k8s.io/community/contributors/design-proposals/architecture/identifiers.md) をご覧ください。

{{% /capture %}}

{{< toc >}}

{{% capture body %}}

<!--
## Names
-->
## 名前 {#names}

<!--
{{< glossary_definition term_id="name" length="all" >}}

A client-provided string that refers to an object in a resource URL, such as /api/v1/pods/some-name.

Only one object of a given kind can have a given name at a time. However, if you delete the object, you can make a new object with the same name.
-->

クライアントが提供する文字列であり、オブジェクトを `/api/v1/pods/some-name` のようなリソース URL で参照します。

特定のオブジェクトに対して、名前を指定できるのは１度だけです。しかし、オブジェクトを削除した後であれば、同じ名前を持つ新しいオブジェクトを作成できます。

<!--
By convention, the names of Kubernetes resources should be up to maximum length of 253 characters and consist of lower case alphanumeric characters, `-`, and `.`, but certain resources have more specific restrictions.
-->
慣例として、Kubernetes のリソース名は最大 253 文字で、アルファベットの小文字、 `-` 、 `.` を含みますが、特定のリソースでは特別な制限を持ちます

<!--
## UIDs
-->
## UID

<!--
{{< glossary_definition term_id="uid" length="all" >}}
A Kubernetes systems-generated string to uniquely identify objects.

Every object created over the whole lifetime of a Kubernetes cluster has a distinct UID. It is intended to distinguish between historical occurrences of similar entities.
-->
Kubernetes はユニークにオブジェクトを識別するため、システムが生成した文字列を使います。


作成されたすべてのオブジェクトは Kubernetes クラスタ上に存在し続ける間、個別に UID を持ちます。これは似たような過去のものとを区別するためです。

{{% /capture %}}