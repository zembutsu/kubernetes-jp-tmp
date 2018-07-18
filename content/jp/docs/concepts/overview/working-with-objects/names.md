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

{{< glossary_definition term_id="name" length="all" >}}
クライアントが提供する文字列であり、オブジェクトを `/api/v1/pods/some-name` のようなリソース URL で参照します。

特定のオブジェクトに対して、名前が指定できるのは１度だけです。しかしながら、

<!--
By convention, the names of Kubernetes resources should be up to maximum length of 253 characters and consist of lower case alphanumeric characters, `-`, and `.`, but certain resources have more specific restrictions.
-->

## UIDs

{{< glossary_definition term_id="uid" length="all" >}}

{{% /capture %}}