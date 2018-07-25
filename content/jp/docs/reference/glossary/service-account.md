---
title: Service Account（サービス・アカウント）
id: service-account
date: 2018-04-12
full_link: /jp/docs/tasks/configure-pod-container/configure-service-account/
short_description: >
  <!--Provides an identity for processes that run in a Pod.-->
  ポッド内で実行するプロセスの同一性を提供します。

aka: 
tags:
- fundamental
- core-object
---
 <!--Provides an identity for processes that run in a {{< glossary_tooltip text="Pod" term_id="pod" >}}.-->
 {{< glossary_tooltip text="ポッド" term_id="pod" >}} 内で実行するプロセスの同一性を提供します。

<!--more--> 

<!--
When processes inside Pods access the cluster, they are authenticated by the API server as a particular service account, for example, `default`. When you create a Pod, if you do not specify a service account, it is automatically assigned the default service account in the same namespace {{< glossary_tooltip text="Namespace" term_id="namespace" >}}.
-->
ポッド内のプロセスがクラスタへ接続する時は、API サーバによって個々のサービス・アカウントとして認証されます。例えば `default` です。ポッドの作成時にサービス・アカウントを指定しなければ、自動的に {{< glossary_tooltip text="名前空間" term_id="namespace" >}} と同じサービス・アカウントがデフォルトで割り当てられます。