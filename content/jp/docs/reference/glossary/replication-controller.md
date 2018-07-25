---
title: Replication Controller（レプリケーション・コントローラ）
id: replication-controller
date: 2018-04-12
full_link: 
short_description: >
  <!--Kubernetes service that ensures a specific number of instances of a pod are always running.-->
  ポットで指定したインスタンス数を常に実行中にするための Kubernetes サービスです。

aka: 
tags:
- workload
- core-object
---
 <!--Kubernetes service that ensures a specific number of instances of a pod are always running.-->
 ポッドで指定したインスタンス数を常に実行中にするための Kubernetes サービスです。

<!--more--> 

<!--
Will automatically add or remove running instances of a pod, based on a set value for that pod. Allows the pod to return to the defined number of instances if pods are deleted or if too many are started by mistake.
-->
ポッドに対して設定されている値に基づいて、実行しているポッドのインスタンスを自動的に追加または削除します。これにより、ポッドを削除するか多くのポッドを間違って作ったとしても、指定した数のインスタンスを常に返します。
