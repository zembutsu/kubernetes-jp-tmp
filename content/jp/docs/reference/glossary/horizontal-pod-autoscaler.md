---
title: Horizontal Pod Autoscaler（水平ポッド自動スケール機能：HPA）
id: horizontal-pod-autoscaler
date: 2018-04-12
full_link: /jp/docs/tasks/run-application/horizontal-pod-autoscale/
short_description: >
  <!--An API resource that automatically scales the number of pod replicas based on targeted CPU utilization or custom metric targets.-->
  CPU 使用率の目標や任意のメトリック（計測指標）目標に基づき、ポッドの複製数を自動的にスケール（拡大）する API リソースです。

aka: 
tags:
- operation
---
 <!--An API resource that automatically scales the number of pod replicas based on targeted CPU utilization or custom metric targets.-->
 CPU 使用率の目標や任意のメトリック（計測指標）目標に基づき、ポッドの複製数を自動的にスケール（拡大）する API リソースです。

<!--more--> 

<!--
HPA is typically used with {{< glossary_tooltip text="Replication Controllers" term_id="replication-controller" >}}, {{< glossary_tooltip text="Deployments" term_id="deployment" >}}, or Replica Sets. It cannot be applied to objects that cannot be scaled, for example {{< glossary_tooltip text="DaemonSets" term_id="daemonset" >}}.
-->
HPA は {{< glossary_tooltip text="Replication Controller（レプリケーション・コントローラ）" term_id="replication-controller" >}}、 {{< glossary_tooltip text="Deployments（デプロイメント）" term_id="deployment" >}}やr Replica Set（レプリカ・セット）と通常は使います。ただし、 {{< glossary_tooltip text="DaemonSets（デーモンセット）" term_id="daemonset" >}}. のようなオブジェクトはスケールできないので割り当てできません。
