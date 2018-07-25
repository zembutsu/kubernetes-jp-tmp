---
title: Dynamic Volume Provisioning（動的なボリュームの自動準備）
id: dynamicvolumeprovisioning
date: 2018-04-12
full_link: /jp/docs/concepts/storage/dynamic-provisioning
short_description: >
  <!--Allows users to request automatic creation of storage  Volumes.-->
  ユーザは記憶領域の容量（ボリューム）自動作成を要求できます。

aka: 
tags:
- core-object
- storage
---
 <!--Allows users to request automatic creation of storage  {{< glossary_tooltip text="Volumes" term_id="volume" >}}.-->
  ユーザは記憶領域の<!--Allows users to request automatic creation of storage  {{< glossary_tooltip text="容量（Volumes）" term_id="volume" >}}.-->自動作成を要求できます。

<!--more--> 

<!--
Dynamic provisioning eliminates the need for cluster administrators to pre-provision storage. Instead, it automatically provisions storage by user request. Dynamic volume provisioning is based on an API object, {{< glossary_tooltip text="StorageClass" term_id="storage-class" >}}, referring to a {{< glossary_tooltip text="Volume Plugin" term_id="volume-plugin" >}} that provisions a {{< glossary_tooltip text="Volume" term_id="volume" >}} and the set of parameters to pass to the Volume Plugin.
-->
動的な自動準備（プロビジョニング）によって、クラスタの管理者は記憶領域（ストレージ）の事前準備を不要にします。そのかわりに、ユーザの要求によって記憶量期を自動的に準備します。動的なボリュームの事前準備は API オブジェクト {{< glossary_tooltip text="StorageClass" term_id="storage-class" >}} をベースとし、{{< glossary_tooltip text="ボリューム・プラグイン" term_id="volume-plugin" >}} が {{< glossary_tooltip text="ボリューム" term_id="volume" >}} の自動展開とボリューム・プラグインによって与えられたパラメータを指定する時に参照するために使います。
