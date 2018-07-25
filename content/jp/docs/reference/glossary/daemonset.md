---
title: DaemonSet（デーモンセット）
id: daemonset
date: 2018-04-12
full_link: /jp/docs/concepts/workloads/controllers/daemonset
short_description: >
  <!--Ensures a copy of a Pod is running across a set of nodes in a cluster.-->
  クラスタ内のノード群を横断して Pod の複製が動くようにします。

aka: 
tags:
- fundamental
- core-object
- workload
---
 <!--Ensures a copy of a {{< glossary_tooltip text="Pod" term_id="pod" >}} is running across a set of nodes in a {{< glossary_tooltip text="cluster" term_id="cluster" >}}.-->
 {{< glossary_tooltip text="クラスタ" term_id="cluster" >}}内のノード群を横断して  {{< glossary_tooltip text="ポッド" term_id="pod" >}} の複製が動くようにします。

<!--more--> 

<!--
Used to deploy system daemons such as log collectors and monitoring agents that typically must run on every {{< glossary_tooltip term_id="node" >}}.
-->
ログ収集や監視アラート・エージェントのように展開用（デプロイ）システム・デーモンが使うもので、通常はすべての{{< glossary_tooltip  text="ノード" term_id="node" >}} 上で動作します。
