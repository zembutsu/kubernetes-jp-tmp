---
title: ReplicaSet（レプリカ・セット）
id: replica-set
date: 2018-04-12
full_link: /jp/docs/concepts/workloads/controllers/replicaset/
short_description: >
  <!--ReplicaSet is the next-generation Replication Controller.-->
  ReplicaSet（レプリカ・セット）は次世代のレプリケーション・コントローラ（複製制御）です。

aka: 
tags:
- fundamental
- core-object
- workload
---
 <!--ReplicaSet is the next-generation Replication Controller.-->
 ReplicaSet（レプリカ・セット）は次世代のレプリケーション・コントローラ（複製制御）です。

<!--more--> 

<!--
ReplicaSet, like ReplicationController, ensures that a specified number of pods replicas are running at one time. ReplicaSet supports the new set-based selector requirements as described in the labels user guide, whereas a Replication Controller only supports equality-based selector requirements.
-->
ReplicaSet はReplicationController（レプリケーション・コントローラ）のように、指定したポッドの複製（レプリカ）を同時に実行中にします。Replication Contrller（レプリケーション・コントローラ）がサポートしているのは equality-based selector 要求であるのに対し、ReplicaSet はラベル・ユーザ・ガイドに説明がある新しい set-based selector 要求をサポートしています。

