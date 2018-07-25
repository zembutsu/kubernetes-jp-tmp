---
title: Kubelet
id: kubelet
date: 2018-04-12
full_link: /jp/docs/reference/generated/kubelet
short_description: >
  <!--An agent that runs on each node in the cluster. It makes sure that containers are running in a pod.-->
  クラスタ内の各ノードで実行するエージェント。ポッドの中でコンテナを確実に実行します。

aka: 
tags:
- fundamental
- core-object
---
 <!--An agent that runs on each node in the cluster. It makes sure that containers are running in a pod.-->
  クラスタ内の各ノードで実行するエージェント。ポッドの中でコンテナを確実に実行します。

<!--more--> 

<!--
The kubelet takes a set of PodSpecs that are provided through various mechanisms and ensures that the containers described in those PodSpecs are running and healthy. The kubelet doesn’t manage containers which were not created by Kubernetes.
-->
kubelet は PodSpec のセットを取得し、PodSpec に記述されたコンテナが確実に正常に稼働し続けるようにするための、様々な仕組みを提供します。kubelet は Kubernetes によって作成されていないコンテナを管理しません。