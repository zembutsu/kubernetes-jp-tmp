---
title: Kubernetes API
id: kubernetes-api
date: 2018-04-12
full_link: /jp/docs/concepts/overview/kubernetes-api/
short_description: >
  <!--The application that serves Kubernetes functionality through a RESTful interface and stores the state of the cluster.-->
  Kubernetes の機能性を提供する RESTful インターフェースの提供と、クラスタの状態を保管するアプリケーションです。

aka: 
tags:
- fundamental
- architecture
---
 <!--The application that serves Kubernetes functionality through a RESTful interface and stores the state of the cluster.-->
 Kubernetes の機能性を提供する RESTful インターフェースの提供と、クラスタの状態を保管するアプリケーションです。

<!--more--> 

<!--
Kubernetes resources and "records of intent" are all stored as API objects, and modified via RESTful calls to the API. The API allows configuration to be managed in a declarative way. Users can interact with the Kubernetes API directly, or via tools like `kubectl`. The core Kubernetes API is flexible and can also be extended to support custom resources.
-->
Kubernetes リソースと "records of intent"（意図の記録）がすべて API オブジェクトに保管され、さらに RESTful な API コールを経由して変更可能です。API は宣言型の手法によって設定を管理できます。ユーザは Kubernetes API と直接やりとりをするか、 `kubectl` のようなツールを経由してやりとりできます。コアの Kubernetes API は柔軟性がアリ、また、カスタム・リソースをサポートするために拡張可能です。