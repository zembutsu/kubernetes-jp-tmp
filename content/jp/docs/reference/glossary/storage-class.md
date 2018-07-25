---
title: Storage Class（ストレージ・クラス）
id: storageclass
date: 2018-04-12
full_link: /jp/docs/concepts/storage/storage-classes
short_description: >
  <!--A StorageClass provides a way for administrators to describe different available storage types.-->
  ストレージ・クラスは管理者のために異なる利用可能なストレージ型（タイプ）を記述する手段を提供します。

aka: 
tags:
- core-object
- storage
---
 <!--A StorageClass provides a way for administrators to describe different available storage types.-->
 ストレージ・クラスは管理者のために異なる利用可能なストレージ型（タイプ）を記述する手段を提供します。

<!--more--> 

<!--
StorageClasses can map to quality-of-service levels, backup policies, or to arbitrary policies determined by cluster administrators. Each StorageClass contains the fields `provisioner`, `parameters`, and `reclaimPolicy`, which are used when a {{< glossary_tooltip text="Persistent Volume" term_id="persistent-volume" >}} belonging to the class needs to be dynamically provisioned. Users can request a particular class using the name of a StorageClass object.
-->
クラスタ管理者がサービス品質レベル、バックアップ方針、あるいは任意のポリシーを決めるときに、ストレージ・クラスを割り当てられます。各ストレージ・クラスに  `provisioner`（プロビジョナ）、 `parameters`（パラメータ）、`reclaimPolicy`（再要求ポリシー） のフィールド（項目）を含みます。これらは  {{< glossary_tooltip text="Persistent Volume（持続型領域）" term_id="persistent-volume" >}} と一緒に使われるもので、動的にプロビジョン（自動構築）する時にクラスが必要となります。ユーザは StorageClass オブジェウトの名前を使い、個々に使うクラスを要求できます。

