---
title: Persistent Volume（持続型領域：パーシステント・ボリューム）
id: persistent-volume
date: 2018-04-12
full_link: /jp/docs/concepts/storage/persistent-volumes/
short_description: >
  <!--An API object that represents a piece of storage in the cluster. Available as a general, pluggable resource that persists beyond the lifecycle of any individual Pod.-->
  クラスタ内で記憶領域（ストレージ）の構成要素を表す API オブジェクトです。個々のポッドのライフサイクルとは切り離して一般的に利用できるプラガブル（着脱可能）なリソースです。

aka: 
tags:
- core-object
- storage
---
 <!--An API object that represents a piece of storage in the cluster. Available as a general, pluggable resource that persists beyond the lifecycle of any individual {{< glossary_tooltip text="Pod" term_id="pod" >}}.-->
 クラスタ内で記憶領域（ストレージ）の構成要素を表す API オブジェクトです。個々の {{< glossary_tooltip text="Pod" term_id="pod" >}}のライフサイクルとは切り離して一般的に利用できるプラガブル（着脱可能）なリソースです

<!--more--> 

<!--
PersistentVolumes (PVs) provide an API that abstracts details of how storage is provided from how it is consumed.
PVs are used directly in scenarios where storage can be created ahead of time (static provisioning).
For scenarios that require on-demand storage (dynamic provisioning), PersistentVolumeClaims (PVCs) are used instead.
-->
持続型領域（PersistentVolue：PV）は API を提供しており、どのように記憶領域を提供するか、どのように消費するかの詳細を抽象化します。記憶領域を事前に作成しておく場合（静的な環境自動構築）には、持続型領域が直接使われます。あるいはオンデマンドに（必要に応じて）記憶領域が必要な場合には、持続型領域の要求（PersistentVolumeClaims）が代わりに使われます。
