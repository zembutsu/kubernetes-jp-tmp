---
title: Deployment（デプロイメント）
id: deployment
date: 2018-04-12
full_link: /jp/docs/concepts/workloads/controllers/deployment/
short_description: >
  <!--An API object that manages a replicated application.-->
  複製されたアプリケーションを管理する API オブジェクトです。

aka: 
tags:
- fundamental
- core-object
- workload
---
 <!--An API object that manages a replicated application.-->
 複製されたアプリケーションを管理する API オブジェクトです。

<!--more--> 

<!--
Each replica is represented by a {{< glossary_tooltip term_id="pod" >}}, and the Pods are distributed among the nodes of a cluster.
-->
 {{< glossary_tooltip text="ポッド" term_id="pod" >}} の複製（レプリカ）とポッドは、クラスタのノード全体にわたって配布されます。
