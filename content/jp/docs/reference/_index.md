---
title: 参照ドキュメント（リファレンス）
approvers:
- chenopis
linkTitle: "リファレンス"
main_menu: true
weight: 70
---

<!--
## API Reference
-->
## API リファレンス {#api-reference}

<!--
* [Kubernetes API Overview](/docs/reference/using-api/api-overview/) - Overview of the API for Kubernetes.
* Kubernetes API Versions
  * [1.11](/docs/reference/generated/kubernetes-api/v1.11/)
  * [1.10](https://v1-10.docs.kubernetes.io/docs/reference/)
  * [1.9](https://v1-9.docs.kubernetes.io/docs/reference/)
  * [1.8](https://v1-8.docs.kubernetes.io/docs/reference/)
  * [1.7](https://v1-7.docs.kubernetes.io/docs/reference/)
-->
* [Kubernetes API 概要](/jp/docs/reference/using-api/api-overview/) - Kubernetes 用 API の概要
* Kubernetes API バージョン
  * [1.11](/jp/docs/reference/generated/kubernetes-api/v1.11/)
  * [1.10](https://v1-10.docs.kubernetes.io/docs/reference/)
  * [1.9](https://v1-9.docs.kubernetes.io/docs/reference/)
  * [1.8](https://v1-8.docs.kubernetes.io/docs/reference/)
  * [1.7](https://v1-7.docs.kubernetes.io/docs/reference/)

<!--
## API Client Libraries
-->
## API クライアント・ライブラリ {#api-client-libraries}

<!--
To call the Kubernetes API from a programming language, you can use
[client libraries](/docs/reference/using-api/client-libraries/). Officially supported
client libraries:
-->
プログラミング言語から Kubernetes API を呼び出すためには、[クライアント・ライブラリ](/jp/docs/reference/using-api/client-libraries/)を利用できます。公式にサポートされているクライアント・ライブラリ：

- [Kubernetes Go client library](https://github.com/kubernetes/client-go/)
- [Kubernetes Python client library](https://github.com/kubernetes-client/python)

<!--
## CLI Reference
-->
## CLI リファレンス {#cli-reference}

<!--
* [kubectl](/docs/user-guide/kubectl-overview) - Main CLI tool for running commands and managing Kubernetes clusters.
    * [JSONPath](/docs/user-guide/jsonpath/) - Syntax guide for using [JSONPath expressions](http://goessner.net/articles/JsonPath/) with kubectl.
* [kubeadm](/docs/admin/kubeadm/) - CLI tool to easily provision a secure Kubernetes cluster. 
* [kubefed](/docs/admin/kubefed/) - CLI tool to help you administrate your federated clusters.
-->
* [kubectl](/jp/docs/user-guide/kubectl-overview) - メインの CLI ツールであり、コマンドの実行と Kubernetes クラスタを管理します。
    * [JSONPath](/docs/user-guide/jsonpath/) - kubectl が使う [JSONPath expressions](http://goessner.net/articles/JsonPath/) 構文ガイドです。
* [kubeadm](/jp/docs/admin/kubeadm/) - 安全な Kubernetes クラスタを簡単に自動構築（プロビジョン）するための CLI ツールです。
* [kubefed](/jp/docs/admin/kubefed/) - クラスタを統合管理するのに役立つ CLI ツールです。

<!--
## Config Reference
-->
## 設定リファレンス {#config-reference}

<!--
* [kubelet](/docs/admin/kubelet/) - The primary *node agent* that runs on each node. The kubelet takes a set of PodSpecs and ensures that the described containers are running and healthy.
* [kube-apiserver](/docs/admin/kube-apiserver/) - REST API that validates and configures data for API objects such as  pods, services, replication controllers.
* [kube-controller-manager](/docs/admin/kube-controller-manager/) - Daemon that embeds the core control loops shipped with Kubernetes.
* [kube-proxy](/docs/admin/kube-proxy/) - Can do simple TCP/UDP stream forwarding or round-robin TCP/UDP forwarding across a set of back-ends.
* [kube-scheduler](/docs/admin/kube-scheduler/) - Scheduler that manages availability, performance, and capacity.
* [federation-apiserver](/docs/admin/federation-apiserver/) - API server for federated clusters.
* [federation-controller-manager](/docs/admin/federation-controller-manager/) - Daemon that embeds the core control loops shipped with Kubernetes federation.
-->
* [kubelet](/jp/docs/admin/kubelet/) -各ノード上で稼働するプライマリの *ノード・エージェント* です。kubelet は PodSpec の集まりで、指定したコンテナの実行と正常性を請け負います。
* [kube-apiserver](/jp/docs/admin/kube-apiserver/) - ポッド、サービス、レプリケーション・コントローラのような API オブジェクトに対する確認と設定のための REST API です。
* [kube-controller-manager](/jp/docs/admin/kube-controller-manager/) - コントロール・ループを Kubernetes で提供できるようにするために中心となるデーモンです。
* [kube-proxy](/jp/docs/admin/kube-proxy/) - シンプルな TCP/UDP ストリーム転送やラウンド・ロビン TCP/UDP 転送をバックエンドを横断して可能にします。
* [kube-scheduler](/jp/docs/admin/kube-scheduler/) - 可用性、性能、機能を管理するスケジューラです。
* [federation-apiserver](/jp/docs/admin/federation-apiserver/) - クラスタを統合するための（federated） API サーバです。
* [federation-controller-manager](/jp/docs/admin/federation-controller-manager/) -  コントロール・ループを Kubernetes で統合して提供できるようにするために中心となるデーモンです。

<!--
## Design Docs
-->
## 設計文章 {#design-docs}

<!--
An archive of the design docs for Kubernetes functionality. Good starting points are [Kubernetes Architecture](https://git.k8s.io/community/contributors/design-proposals/architecture/architecture.md) and [Kubernetes Design Overview](https://git.k8s.io/community/contributors/design-proposals).
-->
Kubernetes の機能性に関する設計用ドキュメントのアーカイブがあります。読み始めるには [Kubernetes アーキテクチャ（英語）](https://git.k8s.io/community/contributors/design-proposals/architecture/architecture.md) and [Kubernetes 設計概要（英語） ](https://git.k8s.io/community/contributors/design-proposals) が良いでしょう。
