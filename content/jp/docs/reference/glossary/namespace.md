---
title: Namespace（名前空間：ネームスペース）
id: namespace
date: 2018-04-12
full_link: /jp/docs/concepts/overview/working-with-objects/namespaces
short_description: >
  <!--An abstraction used by Kubernetes to support multiple virtual clusters on the same physical cluster.-->
  Kubernetes が使う抽象概念であり、同じ物理クラスタ上に複数の仮想クラスタをサポートするためです。

aka: 
tags:
- fundamental
---
 <!--An abstraction used by Kubernetes to support multiple virtual clusters on the same physical {{< glossary_tooltip text="cluster" term_id="cluster" >}}.-->
 Kubernetes が使う抽象概念であり、同じ物理{{< glossary_tooltip text="クラスタ" term_id="cluster" >}}上に複数の仮想クラスタをサポートするためです。

<!--more--> 

<!--
Namespaces are used to organize objects in a cluster and provide a way to divide cluster resources. Names of resources need to be unique within a namespace, but not across namespaces.
-->
名前空間の用途はクラスタ内にあるオブジェクトの体系化と、クラスタのリソースを分割する手法の提供です。名前空間内でのリソースの名前はユニーク（一意）である必要がありますが、名前空間は越えての影響は及ぼしません。