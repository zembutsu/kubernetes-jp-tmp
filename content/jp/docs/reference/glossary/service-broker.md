---
title: Service Broker（サービス・ブローカー）
id: service-broker
date: 2018-04-12
full_link: 
short_description: >
  <!--An endpoint for a set of Managed Services offered and maintained by a third-party.-->
  サードパーティによって提供および保守されているマネージド・サービス一式のエンドポイントです。

aka: 
tags:
- extension
---
 <!--An endpoint for a set of {{< glossary_tooltip text="Managed Services" term_id="managed-service" >}} offered and maintained by a third-party.-->
 サードパーティによって提供および保守されている{{< glossary_tooltip text="マネージド・サービス" term_id="managed-service" >}}一式のエンドポイントです。

<!--more--> 

<!--
{{< glossary_tooltip text="Service Brokers" term_id="service-broker" >}} implement the [Open Service Broker API spec](https://github.com/openservicebrokerapi/servicebroker/blob/v2.13/spec.md) and provide a standard interface for applications to use their Managed Services. [Service Catalog](/docs/concepts/service-catalog/) provides a way to list, provision, and bind with Managed Services offered by Service Brokers.
-->
{{< glossary_tooltip text="サービス・ブローカー" term_id="service-broker" >}} は [Open Service Broker API spec](https://github.com/openservicebrokerapi/servicebroker/blob/v2.13/spec.md) を実装し、マネージド・サービスで使うためのアプリケーションのために共通するインターフェースを提供します。[サービス・カタログ](/jp/docs/concepts/service-catalog/) はサービス・ブローカーによって提供されているマネージド・サービスを一覧表示、環境の自動構築（プロビジョン）利用可能にするための手段を提供します。