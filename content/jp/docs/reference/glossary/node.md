---
title: Node（ノード）
id: node
date: 2018-04-12
full_link: /jp/docs/concepts/architecture/nodes/
short_description: >
  <!--A node is a worker machine in Kubernetes.-->
  ノードは Kubernetes におけるワーカ・マシン（作業用マシン）です。

aka: 
tags:
- fundamental
---
 <!--A node is a worker machine in Kubernetes.-->
 ノードは Kubernetes におけるワーカ・マシン（作業用マシン）です。

<!--more--> 

<!--
A worker machine may be a VM or physical machine, depending on the cluster. It has the {{< glossary_tooltip text="Services" term_id="service" >}} necessary to run {{< glossary_tooltip text="Pods" term_id="pod" >}} and is managed by the master components. The {{< glossary_tooltip text="Services" term_id="service" >}} on a node include Docker, kubelet and kube-proxy.
-->
ワーカ・マシン（作業用マシン）は仮想マシンもしくは物理マシンであり、クラスタに依存します。{{< glossary_tooltip text="ポッド" term_id="pod" >}}  を実行するために必要な {{< glossary_tooltip text="サービス" term_id="service" >}} を保持し、マスタ構成要素（コンポーネント）によって管理されます。ノード上の {{< glossary_tooltip text="サービス" term_id="service" >}} には Docker 、kubelet、kube-proxy を含みます。