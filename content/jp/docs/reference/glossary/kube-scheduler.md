---
title: kube-scheduler
id: kube-scheduler
date: 2018-04-12
full_link: /jp/docs/reference/generated/kube-scheduler/
short_description: >
  <!--Component on the master that watches newly created pods that have no node assigned, and selects a node for them to run on.-->
  マスタの構成要素（コンポーネント）です。直近で作成されたポッドを監視し、ノードが割り当てられていなければ、ポッドを実行するノードを選んで実行します。

aka: 
tags:
- architecture
---
 <!--Component on the master that watches newly created pods that have no node assigned, and selects a node for them to run on.-->
 マスタの構成要素（コンポーネント）です。直近で作成されたポッドを監視し、ノードが割り当てられていなければ、ポッドを実行するノードを選んで実行します。

<!--more--> 

<!--
Factors taken into account for scheduling decisions include individual and collective resource requirements,  hardware/software/policy constraints, affinity and anti-affinity specifications, data locality, inter-workload interference and deadlines.
-->
スケジューリングを決めるために考慮するには、個別の要素と全体的なリソース要求が含まれます。これには、ハードウェアやソフトウエアとポリシーの制約、または親和性の指定、データのローカル保存、お互いのワークフローが干渉しない、そして、期限があげられます。