---
title: Cloud Controller Manager（クラウド・コントローラ・マネージャ）
id: cloud-controller-manager
date: 2018-04-12
full_link: /jp/docs/tasks/administer-cluster/running-cloud-controller/
short_description: >
  <!--Cloud Controller Manager is an alpha feature in 1.8. In upcoming releases it will be the preferred way to integrate Kubernetes with any cloud.-->クラウド・コントローラ・マネージャは 1.8 ではアルファ機能（訳者注：実験的な導入）です。今後のリリースでは、あらゆるクラウドで Kubernetes と統合するのに望ましい手法となります。

aka: 
tags:
- core-object
- architecture
- operation
---
 <!--Cloud Controller Manager is an alpha feature in 1.8. In upcoming releases it will be the preferred way to integrate Kubernetes with any cloud.-->クラウド・コントローラ・マネージャは 1.8 ではアルファ機能（訳者注：実験的な導入）です。今後のリリースでは、あらゆるクラウドで Kubernetes と統合するのに望ましい手法となります。

<!--more--> 

<!--
Kubernetes v1.6 contains a new binary called cloud-controller-manager. cloud-controller-manager is a daemon that embeds cloud-specific control loops.  These cloud-specific control loops were originally in the kube-controller-manager. Since cloud providers develop and release at a different pace compared to the Kubernetes  project, abstracting the provider-specific code to the cloud-controller-manager binary allows cloud vendors to evolve independently from the core Kubernetes code.
-->
Kubernetes v1.6 から cloud-controller-manager という名前の新しいバイナリが入っています。cloud-controller-manager はクラウド特有の制御ループを組み込むためのデーモンです。クラウド固有のコントロール・ループとは、元々は kube-controller-manager に入っていたものでした。クラウド事業者の開発やリリースは Kubernetes プロジェクトとは異なるため、クラウド固有のコードを cloud-controller-manager バイナリに抽象化することで、クラウド事業者はコアの Kubernetes コードとは独立して進化できるようになります。

