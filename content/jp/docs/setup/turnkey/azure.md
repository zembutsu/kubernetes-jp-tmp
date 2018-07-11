---
reviewers:
- colemickens
- brendandburns
title: Azure で Kubernetes を動かす
---

<!--
## Azure Container Service
--->
## Azure コンテナ・サービス

<!--
The [Azure Container Service](https://azure.microsoft.com/en-us/services/container-service/) offers simple
deployments of one of three open source orchestrators: DC/OS, Swarm, and Kubernetes clusters.
--->
[Azure コンテナ・サービス](https://azure.microsoft.com/en-us/services/container-service/)は３つのオープンソース・オーケストレータ DC/OS、Swarm、Kubernetes から１つを簡単に展開（デプロイ）します。


<!--
For an example of deploying a Kubernetes cluster onto Azure via the Azure Container Service:
--->
Azure へ Azure コンテナ・サービスを通して Kubernetes クラスタを展開する例がこちらです：

**[Microsoft Azure Container Service - Kubernetes Walkthrough](https://docs.microsoft.com/en-us/azure/aks/intro-kubernetes)**

<!--
## Custom Deployments: ACS-Engine
--->
## カスタム展開：ACS-Engine

<!--
The core of the Azure Container Service is **open source** and available on GitHub for the community
to use and contribute to: **[ACS-Engine](https://github.com/Azure/acs-engine)**.
--->
Azure コンテナ・サービスの中心となるのは **オープンソース** であり、コミュニティが利用・貢献するために GitHub 上で公開されています：**[ACS-Engine](https://github.com/Azure/acs-engine)**

<!--
ACS-Engine is a good choice if you need to make customizations to the deployment beyond what the Azure Container
Service officially supports. These customizations include deploying into existing virtual networks, utilizing multiple
agent pools, and more. Some community contributions to ACS-Engine may even become features of the Azure Container Service.
--->
展開（デプロイ）のカスタマイズが必要な場合、Azure コンテナ・サービスが公式にサポートしている ACS-Engine は良い選択です。カスタマイズには既存の仮想ネットワークに対する展開、複数のエージェント・プールの活用などを含みます。ACS-Engine に対するコミュニティへの貢献は、Azure コンテナ・サービスの機能になり得ます。

<!--
The input to ACS-Engine is similar to the ARM template syntax used to deploy a cluster directly with the Azure Container Service.
The resulting output is an Azure Resource Manager Template that can then be checked into source control and can then be used
to deploy Kubernetes clusters into Azure.
--->
ACS-Engine に対する入力は ARM テンプレート構文と似ており、これで Azure コンテナ・サービスに直接クラスタを展開するのに使えます。Azure リソース・マネージャ・テンプレートの出力結果は、Azure 上に Kubernetes クラスタの展開後、ソース・コントロールで確認できます。

<!--
You can get started quickly by following the **[ACS-Engine Kubernetes Walkthrough](https://github.com/Azure/acs-engine/blob/master/docs/kubernetes.md)**.
--->
 **[ACS-Engine Kubernetes Walkthrough](https://github.com/Azure/acs-engine/blob/master/docs/kubernetes.md)** に従い、迅速に始められます。

<!--
## CoreOS Tectonic for Azure
--->
## Azure 用 CoreOS Tectonic

<!--
The CoreOS Tectonic Installer for Azure is **open source** and available on GitHub for the community to use and contribute to: **[Tectonic Installer](https://github.com/coreos/tectonic-installer)**.
--->

<!--
Tectonic Installer is a good choice when you need to make cluster customizations as it is built on [Hashicorp's Terraform](https://www.terraform.io/docs/providers/azurerm/) Azure Resource Manager (ARM) provider. This enables users to customize or integrate using familiar Terraform tooling.
--->

<!--
You can get started using the [Tectonic Installer for Azure Guide](https://coreos.com/tectonic/docs/latest/install/azure/azure-terraform.html).
-->
