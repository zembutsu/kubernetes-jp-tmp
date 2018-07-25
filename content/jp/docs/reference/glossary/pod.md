---
title: Pod（ポッド）
id: pod
date: 2018-04-12
full_link: /jp/docs/concepts/workloads/pods/pod-overview/
short_description: >
  <!--The smallest and simplest Kubernetes object. A Pod represents a set of running containers on your cluster.-->
  最小かつ最も単純な Kubernetes オブジェクトです。ポッドはクラスタ上で実行するコンテナの集まりを意味します。

aka: 
tags:
- core-object
- fundamental
---
 <!--The smallest and simplest Kubernetes object. A Pod represents a set of running {{< glossary_tooltip text="containers" term_id="container" >}} on your cluster.-->
  最小かつ最も単純な Kubernetes オブジェクトです。ポッドはクラスタ上で実行する {{< glossary_tooltip text="コンテナ" term_id="container" >}}の集まりを意味します。

<!--more--> 

<!--
A Pod is typically set up to run a single primary container. It can also run optional sidecar containers that add supplementary features like logging. Pods are commonly managed by a {{< glossary_tooltip term_id="deployment" >}}.
-->
ポッドは主として単一のプライマリ・コンテナが実行するセットアップします。また、オプションでサイドカー・コンテナも実行します。サイドカー・コンテナとは、ログ記録などの補助的な機能をポッドに追加します。ポッドは {{< glossary_tooltip text="Deployment（デプロイメント）"  term_id="deployment" >}} によって共通管理されます。
