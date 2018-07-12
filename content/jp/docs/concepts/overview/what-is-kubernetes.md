---
reviewers:
- bgrant0607
- mikedanese
title: Kubernetes とは何か？
content_template: templates/concept
weight: 10
---

{{% capture overview %}}
<!--
This page is an overview of Kubernetes.
-->
このページは Kubernetes 概要です。
{{% /capture %}}

{{% capture body %}}
<!--
Kubernetes is a portable, extensible open-source platform for managing
containerized workloads and services, that facilitates both
declarative configuration and automation. It has a large, rapidly
growing ecosystem. Kubernetes services, support, and tools are widely available.
-->
Kubernetes とはコンテナ化した作業負荷（ワークロード）やサービスを管理するために、持ち運びでき（portable）拡張性がある（extensible）オープンソース基盤（プラットフォーム）です。Kubernetes は宣言型の設定（declarative configuration）と自動化の両方を容易にします。大きく、急成長したエコシステムがあります。Kubernetes サービス、サポート、ツールは広範囲にわたって利用できます。

<!--
Google open-sourced the Kubernetes project in 2014. Kubernetes builds upon
a [decade and a half of experience that Google has with running
production workloads at
scale](https://research.google.com/pubs/pub43438.html), combined with
best-of-breed ideas and practices from the community.
-->
Google は Kubernetes プロジェクトを 2014 年にオープンソース化しました。Kubernetes は [decade and a half of experience that Google has with running production workloads at scale（Google が大規模な本番用ワークロードを実行した、15年にわたる経験）](https://research.google.com/pubs/pub43438.html)を踏まえて構築されており、コミュニティによる最善のアイディアと実践が組み合わさっています。

<!--
## Why do I need Kubernetes and what can it do?
-->
## なぜ Kubernetes が必要で、何ができるのか？ {#why-do-i-need}

<!--
Kubernetes has a number of features. It can be thought of as:
-->
Kubernetes は多くの機能があります。考えられ得るのは：

<!--
- a container platform
- a microservices platform
- a portable cloud platform
and a lot more.
-->
- コンテナ基盤（container platform）
- マイクロサービス基盤（microservices platform）
- 移植できるクラウド基盤（portable cloud platform）など様々

<!--
Kubernetes provides a **container-centric** management environment. It
orchestrates computing, networking, and storage infrastructure on
behalf of user workloads. This provides much of the simplicity of
Platform as a Service (PaaS) with the flexibility of Infrastructure as
a Service (IaaS), and enables portability across infrastructure
providers.
-->
Kubernetes は **コンテナ中心（container-centric）** の管理環境を備えています。ユーザが負担している作業をするのに代わり、Kubernetes が計算（computing：コンピューティング）、ネットワーク形成（networking：ネットワーキング）、記憶装置基盤（storage infrastructure：ストレージ基盤）を指揮（orchestrate：オーケストレート）します。

<!--
## How is Kubernetes a platform?
-->
## Kubernetes 基盤はどうですか？ {#how-is-kubernetes-a-platform}

<!--
Even though Kubernetes provides a lot of functionality, there are
always new scenarios that would benefit from new
features. Application-specific workflows can be streamlined to
accelerate developer velocity. Ad hoc orchestration that is acceptable
initially often requires robust automation at scale. This is why
Kubernetes was also designed to serve as a platform for building an
ecosystem of components and tools to make it easier to deploy, scale,
and manage applications.
-->
Kubernetes は多くの機能性を提供するのですが、新機能の利点がもたらす新しいシナリオが常にあります。開発者の速度を迅速化するために、アプリケーション固有の作業の流れ（workflows：ワークフロー）は、無駄を無くして合理化できます。臨機応変な指揮（orchestration：オーケストレーション）のためには、しっかりとした自動化が大規模に必要なのを、初期の段階から受け入れなくてはいけません。また、これこそが、どうして Kubernetes が基盤（プラットフォーム）として役立つように設計されたかでもあります。この基盤を構築するのは、アプリケーションの配置（deploy：デプロイ）、規模の拡大縮小（scale：スケール）、管理を簡単にするための、構成要素（component：コンポーネント）やツールのエコシステムです。

<!--
[Labels](/docs/concepts/overview/working-with-objects/labels/) empower
users to organize their resources however they
please. [Annotations](/docs/concepts/overview/working-with-objects/annotations/)
enable users to decorate resources with custom information to
facilitate their workflows and provide an easy way for management
tools to checkpoint state.
-->
[Labels（ラベル）](/jp/docs/concepts/overview/working-with-objects/labels/) は資源（リソース）の整理によって、ユーザが常に満足して（資源を）活用できるようにします。[Annotations（注釈、アノテーション）](/jp/docs/concepts/overview/working-with-objects/annotations/) により、ユーザは資源（リソース）に何らかの情報を付与可能にします。これは業務の流れを容易にするためで、状態のチェックポイントを管理ツールが簡単になる方法を提供します。

<!--
Additionally, the [Kubernetes control
plane](/docs/concepts/overview/components/) is built upon the same
[APIs](/docs/reference/using-api/api-overview/) that are available to developers
and users. Users can write their own controllers, such as
[schedulers](https://github.com/kubernetes/community/blob/{{< param "githubbranch" >}}/contributors/devel/scheduler.md),
with [their own
APIs](/docs/concepts/api-extension/custom-resources/)
that can be targeted by a general-purpose [command-line
tool](/docs/user-guide/kubectl-overview/).
-->
加えて、 [Kubernetes コントロール・プレーン](/jp/docs/concepts/overview/components/) が構築する基盤となっているのは、開発者とユーザが利用可能な [API](/jp/docs/reference/using-api/api-overview/) と同じです。ユーザは自分で[スケジューラ](https://github.com/kubernetes/community/blob/{{< param "githubbranch" >}}/contributors/devel/scheduler.md)のようなコントローラを [自身の API](/docs/concepts/api-extension/custom-resources/) で書けます。あるいは汎用的な [コマンドライン・ツール](/jp/docs/user-guide/kubectl-overview/) も対象にできます。

<!--
This
[design](https://git.k8s.io/community/contributors/design-proposals/architecture/architecture.md)
has enabled a number of other systems to build atop Kubernetes.
-->
このような [設計](https://git.k8s.io/community/contributors/design-proposals/architecture/architecture.md) をしているため、Kubernetes の上で、数々の他システムを構築できます。

<!--
## What Kubernetes is not
-->
## Kubernetes は何ではないか {#what-kubernetes-is-not}

<!--
Kubernetes is not a traditional, all-inclusive PaaS (Platform as a
Service) system. Since Kubernetes operates at the container level
rather than at the hardware level, it provides some generally
applicable features common to PaaS offerings, such as deployment,
scaling, load balancing, logging, and monitoring. However, Kubernetes
is not monolithic, and these default solutions are optional and
pluggable. Kubernetes provides the building blocks for building developer
platforms, but preserves user choice and flexibility where it is
important.
-->
Kubernetes は旧来からの包括的な PaaS（Platform as a Service：サービスとしての基盤）システムではありません。Kubernetes はハードウェア面というよりは、コンテナ面での操作をするからです。いくつかの機能は PaaS が提供するものと共通でしょう。たとえば配置（デプロイ）、規模の拡大縮小（スケール）、負荷分散、ログ記録、監視です。しかしながら、Kubernetes は一枚岩的な構造（monolithic：モノリシック）ではありません。また、これら標準的な（機能に対する）解決法は任意（オプション）であり取り付け・取り外しが自由（pluggable：プラガブル）です。Kubernetes が提供するのは開発者基盤を構築するための基本的要素ですが、ユーザによる選択と柔軟性の保持する所こそが重要です。

Kubernetes:

<!--
* Does not limit the types of applications supported. Kubernetes aims
  to support an extremely diverse variety of workloads, including
  stateless, stateful, and data-processing workloads. If an
  application can run in a container, it should run great on
  Kubernetes.
* Does not deploy source code and does not build your
  application. Continuous Integration, Delivery, and Deployment
  (CI/CD) workflows are determined by organization cultures and preferences
  as well as technical requirements.
* Does not provide application-level services, such as middleware
  (e.g., message buses), data-processing frameworks (for example,
  Spark), databases (e.g., mysql), caches, nor cluster storage systems (e.g.,
  Ceph) as built-in services. Such components can run on Kubernetes, and/or
  can be accessed by applications running on Kubernetes through portable
  mechanisms, such as the Open Service Broker.
* Does not dictate logging, monitoring, or alerting solutions. It provides
  some integrations as proof of concept, and mechanisms to collect and
  export metrics.
* Does not provide nor mandate a configuration language/system (e.g.,
  [jsonnet](https://github.com/google/jsonnet)). It provides a declarative
  API that may be targeted by arbitrary forms of declarative specifications.
* Does not provide nor adopt any comprehensive machine configuration,
  maintenance, management, or self-healing systems.
-->
* 扱うアプリケーションの種類に制限はありません。Kubernetes は非常に様々な多様性のある作業負荷（ワークロード）のサポートを目標としています。作業付加にはステートレス（stateless：状態を保持しない）、ステートフル（statefull：状態を保持）、データ処理を含みます。アプリケーションをコンテナとして実行可能であれば、大いに Kubernetes で実行すべきでしょう。
* ソースコードの配置（デプロイ）やアプリケーションの構築をしません。継続的インテグレーション、デリバリ、デプロイ（CI/CD）の作業手順を決めるのは、組織文化と優先度だけでなく、技術的な要件も同様です。
* アプリケーション面でのサービスは提供しません。たとえば、ミドルウェア（例：メッセージ・バス）、データ処理フレームワーク（例：Spark）、データベース（例：mysql）、キャッシュだけでなく、クラスタ・ストレージ・システム（例：Cepth）を内蔵サービス（built-in services）として提供しません。構成要素（コンポーネント）によっては Kubernetes 上で実行できないか、あるいは、Kubernets 上で動作するアプリケーションが Open Service Broker のようなポータブルな仕組みを通さないと通信できません。
* ログ記録、監視、通報（アラート）に対処する解決策（ソリューション）はありません。、いくつかは概念実証（PoC：Proof of Concept）としての機能統合と、監視用指標（metric）の収集と出力に関する仕組みがあります。
* 設定言語システム（ 例：[jsonnet](https://github.com/google/jsonnet)）を提供しませんし、委任もしません。提供するのは宣言型 API であり、宣言型仕様によって形成されているものも対象になる場合があります。

<!--
Additionally, Kubernetes is not a mere *orchestration system*. In
fact, it eliminates the need for orchestration. The technical
definition of *orchestration* is execution of a defined workflow:
first do A, then B, then C. In contrast, Kubernetes is comprised of a
set of independent, composable control processes that continuously
drive the current state towards the provided desired state. It
shouldn't matter how you get from A to C. Centralized control is also
not required. This results in a system that is easier to use and more
powerful, robust, resilient, and extensible.
-->
さらに付け加えると、Kubernetes は単なる *オーケストレーション・システム（orchestration system）* ではありません。実際にはオーケストレーションの必要性を排除します。 *オーケストレーション（orchestration）* の技術的な定義は、定義した作業手順（ワークフロー）の実行です。つまり、まず A を処理し、つぎは B、それから C です。これに対し、Kubernetes は独立した制御プロセスの集まりで構成されており、現在の状態が指定した期待状態に向かうよう継続的に駆動します。どのように A から C に至るかは問題としません。また、制御を中心に集中する必要もありません。この結果、簡単に使用でき、より強力で、堅牢、弾力的かつ拡張可能なシステムになりました。

<!--
## Why containers?
-->
## なぜコンテナですか？ {#why-containers}
<!--
Looking for reasons why you should be using containers?
-->
なぜコンテナを使うべきかの理由をお探しですか？

![Why Containers?](/images/docs/why_containers.svg)

<!--
The *Old Way* to deploy applications was to install the applications
on a host using the operating-system package manager. This had the
disadvantage of entangling the applications' executables,
configuration, libraries, and lifecycles with each other and with the
host OS. One could build immutable virtual-machine images in order to
achieve predictable rollouts and rollbacks, but VMs are heavyweight
and non-portable.
-->
アプリケーションを配置（デプロイ）する *古い手法* とは、オペレーティングシステムのパッケージ・マネージャを使い、ホスト上にアプリケーションをインストールすることでした。これはアプリケーションとして実行するもの、設定ファイル、ライブラリ、ライフサイクルがホスト OS やその他に巻き込まれる不都合がありました。ロールアウト（展開）とロールバック（巻き戻し）を予測可能にするために、内容が変わらない（immutable：イミュータブルな）仮想マシン・イメージを使う方法がありました。しかし、仮想マシンは巨大で重く、可搬性がありません。

<!--
The *New Way* is to deploy containers based on operating-system-level
virtualization rather than hardware virtualization. These containers
are isolated from each other and from the host: they have their own
filesystems, they can't see each others' processes, and their
computational resource usage can be bounded. They are easier to build
than VMs, and because they are decoupled from the underlying
infrastructure and from the host filesystem, they are portable across
clouds and OS distributions.
-->
*新しい手法* とは、オペレーティングシステム面での仮想化に基づくコンテナの配置（デプロイ）であり、ハードウェア仮想化ではありません。これらのコンテナは相互かつホストからも隔てられています（isolated）。コンテナは各々でファイルシステムを持つため、他のプロセスのものを見られません。それに、計算資源の使用率が跳ね上がっても分かりません。コンテナは仮想マシンを構築するよりも簡単です。それに、基礎をなす基盤（インフラ）とホスト・ファイルシステムからは切り離されるため、クラウドと OS ディストリビューションを横断して移動できます。

<!--
Because containers are small and fast, one application can be packed
in each container image. This one-to-one application-to-image
relationship unlocks the full benefits of containers. With containers,
immutable container images can be created at build/release time rather
than deployment time, since each application doesn't need to be
composed with the rest of the application stack, nor married to the
production infrastructure environment. Generating container images at
build/release time enables a consistent environment to be carried from
development into production.  Similarly, containers are vastly more
transparent than VMs, which facilitates monitoring and
management. This is especially true when the containers' process
lifecycles are managed by the infrastructure rather than hidden by a
process supervisor inside the container. Finally, with a single
application per container, managing the containers becomes tantamount
to managing deployment of the application.
-->
コンテナは小さくて速いので、あるアプリケーションを各コンテナ・イメージに



Summary of container benefits:

* **Agile application creation and deployment**:
    Increased ease and efficiency of container image creation compared to VM image use.
* **Continuous development, integration, and deployment**:
    Provides for reliable and frequent container image build and
    deployment with quick and easy rollbacks (due to image
    immutability).
* **Dev and Ops separation of concerns**:
    Create application container images at build/release time rather
    than deployment time, thereby decoupling applications from
    infrastructure.
* **Observability**
    Not only surfaces OS-level information and metrics, but also application
    health and other signals.
* **Environmental consistency across development, testing, and production**:
    Runs the same on a laptop as it does in the cloud.
* **Cloud and OS distribution portability**:
    Runs on Ubuntu, RHEL, CoreOS, on-prem, Google Kubernetes Engine, and anywhere else.
* **Application-centric management**:
    Raises the level of abstraction from running an OS on virtual
    hardware to run an application on an OS using logical resources.
* **Loosely coupled, distributed, elastic, liberated [micro-services](https://martinfowler.com/articles/microservices.html)**:
    Applications are broken into smaller, independent pieces and can
    be deployed and managed dynamically -- not a fat monolithic stack
    running on one big single-purpose machine.
* **Resource isolation**:
    Predictable application performance.
* **Resource utilization**:
    High efficiency and density.

## What does Kubernetes mean? K8s?

The name **Kubernetes** originates from Greek, meaning *helmsman* or
*pilot*, and is the root of *governor* and
[cybernetic](http://www.etymonline.com/index.php?term=cybernetics). *K8s*
is an abbreviation derived by replacing the 8 letters "ubernete" with
"8".

{{% /capture %}}

{{% capture whatsnext %}}
*   Ready to [Get Started](/docs/setup/)?
*   For more details, see the [Kubernetes Documentation](/docs/home/).
{{% /capture %}}


