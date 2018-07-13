---
reviewers:
- lavalamp
title: Kubernetes の構成要素
content_template: templates/concept
weight: 20
---

{{% capture overview %}}
<!--
This document outlines the various binary components needed to
deliver a functioning Kubernetes cluster.
-->
様々なバイナリ構成要素（コンポーネント）の概要を説明します。構成要素は Kubernetes クラスタに機能を提供するために必要なものです。
{{% /capture %}}

{{% capture body %}}
<!--
## Master Components
-->
## マスタ構成要素 {#master-components}

<!--
Master components provide the cluster's control plane. Master components make global decisions about the
cluster (for example, scheduling), and detecting and responding to cluster events (starting up a new pod when a replication controller's 'replicas' field is unsatisfied).
-->
マスタを構成する要素は、クラスタのコントロール・プレーン（control plane）を提供します（訳者注；コントロール・プレーンは、通信制御と径路指定の役割を担うパーツ）。マスタ構成要素はクラスタ全体に対する決定（例：スケジューリング）を行い、かつ、クラスタでのイベントを検出・対応（レプリケーション・コントローラの 'replicas' 項目が満たさないとき、新しいポッドを起動 ）します。

<!--
Master components can be run on any machine in the cluster. However,
for simplicity, set up scripts typically start all master components on
the same machine, and do not run user containers on this machine. See
[Building High-Availability Clusters](/docs/admin/high-availability/) for an example multi-master-VM setup.
-->
マスタ構成要素はクラスタ内のどのマシン上でも実行できます。しかしながら通常のセットアップ・スクリプトは、作業を簡単にするために、全てのマスタ構成要素を同じマシン上で起動します。複数の仮想マシンでマスタをセットアップする例は [高可用性クラスタの構築](/jp/docs/setup/independent/high-availability/) をご覧ください。

### kube-apiserver

<!--
{{< glossary_definition term_id="kube-apiserver" length="all" >}}
Component on the master that exposes the Kubernetes API. It is the front-end for the Kubernetes control plane.
It is designed to scale horizontally – that is, it scales by deploying more instances. See Building High-Availability Clusters.
-->
Kubernetes API を公開する（expose：露出する）マスタの構成要素です。Kubernetes コントロール・プレーンのフロントエンドです。
水平スケールできるように設計してありますので、規模を拡大するには、より多く配置します。[高可用性クラスタの構築](/jp/docs/setup/independent/high-availability/) をご覧ください。

### etcd
<!--
{{< glossary_definition term_id="etcd" length="all" >}}
Consistent and highly-available key value store used as Kubernetes’ backing store for all cluster data.
Always have a backup plan for etcd’s data for your Kubernetes cluster. For in-depth information on etcd, see etcd documentation.
-->
一貫性と高い可用性があるキーバリュー・ストアです。 Kubernets の外部記憶装置として全てのクラスタ・データをを保管します。Kubernetes クラスタ用 etcd データのバックアップ計画は、常に持ってください。etcd の掘り下げた情報は、 [etcd ドキュメント](https://github.com/coreos/etcd/blob/master/Documentation/docs.md) をご覧ください。

### kube-scheduler
<!--
{{< glossary_definition term_id="kube-scheduler" length="all" >}}
Component on the master that watches newly created pods that have no node assigned, and selects a node for them to run on.
Factors taken into account for scheduling decisions include individual and collective resource requirements, hardware/software/policy constraints, affinity and anti-affinity specifications, data locality, inter-workload interference and deadlines.
-->
新たに作成されたポッドを監視するマスタの構成要素です。ノードが割り当てられていなければ、実行するためのノードが選ばれます。
スケジューリング決定にあたって考慮する要素に含まれるのは、個々および全体的な資源（リソース）要件、ハードウェア／ソフトウェア／ポリシー制限、親和性（affinity）と非親和性（auti-affinity）の指定、データの場所、作業負荷間の衝突や期限です。

### kube-controller-manager
<!--
{{< glossary_definition term_id="kube-controller-manager" length="all" >}}
Component on the master that runs controllers .
Logically, each controller is a separate process, but to reduce complexity, they are all compiled into a single binary and run in a single process.
-->
コントローラを実行するマスタの構成要素です。
論理的に、各コントローラはプロセスによって隔てられていますが、複雑さを緩和するため、すべてが１つのバイナリにコンパイルされており、１つのプロセスとして実行します。

<!--
These controllers include:
-->
以下をコントローラに含みます：
<!--
  * Node Controller: Responsible for noticing and responding when nodes go down.
  * Replication Controller: Responsible for maintaining the correct number of pods for every replication
  controller object in the system.
  * Endpoints Controller: Populates the Endpoints object (that is, joins Services & Pods).
  * Service Account & Token Controllers: Create default accounts and API access tokens for new namespaces.
-->
  * ノード・コントローラ（Node Controller）：ノードがダウンすると、通知と対応する責任があります。
  * レプリケーション・コントローラ（Replication Controller）：システム上の全てのレプリケーション・コントローラ・オブジェクトに対し、適切なポッド数を維持する責任があｒます。
  * エンドポイント・コントローラ（Endpoints Controller）：エンドポイント・オブジェクトを投入します（つまり、サービスとポッドの追加です）。
  * サービス・アカウント＆トークン・コントローラ（Service Account & Token）：新しい名前空間のためにデフォルトのアカウントと API アクセス・トークンを作成します。

### cloud-controller-manager

<!--
[cloud-controller-manager](/docs/tasks/administer-cluster/running-cloud-controller/) runs controllers that interact with the underlying cloud providers. The cloud-controller-manager binary is an alpha feature introduced in Kubernetes release 1.6.
-->
[cloud-controller-manager](/jp/docs/tasks/administer-cluster/running-cloud-controller/) はクラウド・プロバイダとやり取りするコントローラを実行します。cloud-controller-manager バイナリは、 Kubernetes リリース 1.6 で導入されたアルファ機能です。

<!--
cloud-controller-manager runs cloud-provider-specific controller loops only. You must disable these controller loops in the kube-controller-manager. You can disable the controller loops by setting the `--cloud-provider` flag to `external` when starting the kube-controller-manager.
-->
cloud-controller-manager は cloud-provider-specific （クラウド・プロバイダ固有の）コントローラの実行を繰り返すだけです。コントローラの繰り返しは kube-controller-manager で無効化する必要があります。kube-controller-manager の開始時に `--cloud-provider` フラグに `external` を指定すると、コントローラ・ループを無効化できます。

<!--
cloud-controller-manager allows cloud vendors code and the Kubernetes core to evolve independent of each other. In prior releases, the core Kubernetes code was dependent upon cloud-provider-specific code for functionality. In future releases, code specific to cloud vendors should be maintained by the cloud vendor themselves, and linked to cloud-controller-manager while running Kubernetes.
-->
cloud-controller-manager は、クラウド事業者のコードと Kubernetes の中核（コア）をお互いに独立した状態にできます。以前のリリースでは、クラウド・プロバイダ固有のコードに依存している機能が Kubernetes の中核にありました。今後のリリースでは、クラウド事業者に対する特定のコードは、各クラウド事業者自身によってメンテナンスされるべきです。そして、cloud-controller-manager が実行中の Kuberntes とを結び付けます（リンクします）。

<!--
The following controllers have cloud provider dependencies:
-->
以下はクラウド・プロバイダに依存するコントローラです：

<!--
  * Node Controller: For checking the cloud provider to determine if a node has been deleted in the cloud after it stops responding
  * Route Controller: For setting up routes in the underlying cloud infrastructure
  * Service Controller: For creating, updating and deleting cloud provider load balancers
  * Volume Controller: For creating, attaching, and mounting volumes, and interacting with the cloud provider to orchestrate volumes
-->
  * Node Controller（ノード・コントローラ）：クラウドが応答を停止した後、ノードが削除されたかどうかを判断するために、クラウド・プロバイダを確認します。
  * Route Controller（ルート・コントローラ）：下にあるクラウド基盤（インフラ）で径路（ルート）をセットアップします。
  * Service Controller（サービス・コントローラ）：クラウド・プロバイダのロードバランサを作成、更新、削除します。
  * Volume Controller（ボリューム・コントローラ）：ボリュームの作成・アタッチ・マウントと、ボリュームを調整する（orchestrate）ためにクラウド・プロバイダとやりとりします。

<!--
## Node Components
-->
## ノード構成要素 {#node-components}
<!--
Node components run on every node, maintaining running pods and providing the Kubernetes runtime environment.
-->
全てのノード上でノード構成要素が動き、ポッドの稼働を保守し、Kubernetes 実行環境を提供します。

### kubelet
<!--
{{< glossary_definition term_id="kubelet" length="all" >}}
An agent that runs on each node in the cluster. It makes sure that containers are running in a pod.
The kubelet takes a set of PodSpecs that are provided through various mechanisms and ensures that the containers described in those PodSpecs are running and healthy. The kubelet doesn’t manage containers which were not created by Kubernetes.
-->
クラスタ内の各ノード上で実行するエージェントであり、ポット内でコンテナが間違いなく稼働するようにします。
kubelet は PodSpecs の一式を取り込んでいます。これは PodSpecs に記述されたコンテナが確実に正常に実行できるように、様々な仕組みを提供します。kubelet は Kubernetes で作成されていないコンテナを管理しません。

### kube-proxy

<!--
[kube-proxy](/docs/admin/kube-proxy/) enables the Kubernetes service abstraction by maintaining
network rules on the host and performing connection forwarding.
-->
[kube-proxy](/p/docs/admin/kube-proxy/) は Kubernetes サービスを抽象化するため、ホスト上でネットワーク・ルールを保守し、接続を転送処理します。

<!--
### Container Runtime
-->
### コンテナ・ランタイム（Container Runtime） {#container-runtime}

<!--
The container runtime is the software that is responsible for running containers. Kubernetes supports several runtimes: [Docker](http://www.docker.com), [rkt](https://coreos.com/rkt/), [runc](https://github.com/opencontainers/runc) and any OCI [runtime-spec](https://github.com/opencontainers/runtime-spec) implementation.
-->
コンテナ・ランタイムとはコンテナの実行に責任を持つソフトウェアです。Kubernets は複数のランタイムをサポートします。[Docker](http://www.docker.com)、 [rkt](https://coreos.com/rkt/)、 [runc](https://github.com/opencontainers/runc) は、いずれも OCI [runtime-spec（ランタイム仕様）](https://github.com/opencontainers/runtime-spec) を実装しています.

<!--
## Addons
-->
## アドオン {#addons}

<!--
Addons are pods and services that implement cluster features. The pods may be managed
by Deployments, ReplicationControllers, and so on. Namespaced addon objects are created in
the `kube-system` namespace.
-->
アドオンとはポッドとサービスに対してクラスタの機能を提供するものです。ポッドは Depyoment 、ReplicationController 等によっても管理できるでしょう。名前空間アドオン・オブジェクトが `kube-system` 名前空間内に作成されます。

<!--
Selected addons are described below, for an extended list of available addons please see [Addons](/docs/concepts/cluster-administration/addons/).
-->
いくつかのアドオンを以下で説明します。利用可能なアドオンの幅広い情報は、[アドオン](/jp/docs/concepts/cluster-administration/addons/) をご覧ください。

### DNS

<!--
While the other addons are not strictly required, all Kubernetes clusters should have [cluster DNS](/docs/concepts/services-networking/dns-pod-service/), as many examples rely on it.
-->
すべての Kubernetes クラスタには  [クラスタ DNS](/jp/docs/concepts/services-networking/dns-pod-service/) があり、多くの例がこちらに依存しているため、厳密には他のアドオンは不要です。

<!--
Cluster DNS is a DNS server, in addition to the other DNS server(s) in your environment, which serves DNS records for Kubernetes services.
-->
クラスタ DNS は DNS サーバです。環境に別の DNS サーバを追加すると、Kubernetes サービス用に DNS レコードを提供します。

<!--
Containers started by Kubernetes automatically include this DNS server in their DNS searches.
-->
Kubernetes が自動的に起動するコンテナには、この DNS サーバが含まれており、Kubernetes の DNS 検索のために使います。

<!--
### Web UI (Dashboard)
-->
### Web UI （ダッシュボード）

<!--
[Dashboard](/docs/tasks/access-application-cluster/web-ui-dashboard/) is a general purpose, web-based UI for Kubernetes clusters. It allows users to manage and troubleshoot applications running in the cluster, as well as the cluster itself.
-->
[ダッシュボード](/jp/docs/tasks/access-application-cluster/web-ui-dashboard/) は Kubernetes クラスタに対する汎用的なウェブをベースとするユーザ・インタフェースです。ユーザがクラスタ上で実行しているアプリケーションの管理とトラブルシュートをできるようにするだけでなく、クラスタに対しても使えます。

<!--
### Container Resource Monitoring
-->
### コンテナ・リソース・モニタリング（資源の監視）

<!--
[Container Resource Monitoring](/docs/tasks/debug-application-cluster/resource-usage-monitoring/) records generic time-series metrics
about containers in a central database, and provides a UI for browsing that data.
-->
[コンテナ・リソース・モニタリング（Container Resource Monitoring）](/jp/docs/tasks/debug-application-cluster/resource-usage-monitoring/) はコンテナに対する包括的な時系列指標（メトリクス）を中央データベースに記録し、UI を通してデータを参照できます。


<!--
### Cluster-level Logging
-->
### クラスタ・レベルのログ記録

<!--
A [Cluster-level logging](/docs/concepts/cluster-administration/logging/) mechanism is responsible for
saving container logs to a central log store with search/browsing interface.
-->
[クラスタ・レベルのログ記録（Cluster-level logging）](/jp/docs/concepts/cluster-administration/logging/) は、コンテナのログを中央にあるログ・ストア（log store）に記録し、インターフェースを通して検索・閲覧できるようにする責任があります。

{{% /capture %}}


