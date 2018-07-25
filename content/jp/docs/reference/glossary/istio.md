---
title: Istio（イスティオ）
id: istio
date: 2018-04-12
full_link: https://istio.io/docs/concepts/what-is-istio/overview.html
short_description: >
  <!--An open platform (not Kubernetes-specific) that provides a uniform way to integrate microservices, manage traffic flow, enforce policies, and aggregate telemetry data.-->
  マイクロサービスを統合し、トラフィックの流れを管理し、ポイシーを適用し、遠隔測定したデータを統合するために、統一した手法を提供する オープンなプラットフォーム（Kubernetes 固有ではありません）です。

aka: 
tags:
- networking
- architecture
- extension
---
 <!--An open platform (not Kubernetes-specific) that provides a uniform way to integrate microservices, manage traffic flow, enforce policies, and aggregate telemetry data.-->
 マイクロサービスを統合し、トラフィックの流れを管理し、ポイシーを適用し、遠隔測定したデータを統合するために、統一した手法を提供する オープンなプラットフォーム（Kubernetes 固有ではありません）です。
<!--more--> 

<!--
Adding Istio does not require changing application code. It is a layer of infrastructure between a service and the network, which when combined with service deployments, is commonly referred to as a service mesh. Istio's control plane abstracts away the underlying cluster management platform, which may be Kubernetes, Mesosphere, etc.
-->
Istio の追加にはアプリケーション・コードの変更を必要としません。サービスとネットワークの間にある基盤（インフラ）のレイヤとは、サービスを展開（デプロイ）する時に組み合わせるものであり、通常はサービス・メッシュと呼ばれます。Istio はコントロール・プレーンで根底にあるクラスタを管理するプラットフォームを抽象化します。