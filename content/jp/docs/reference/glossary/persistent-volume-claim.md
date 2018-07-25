---
title: Persistent Volume Claim（持続型領域の要求）
id: persistent-volume-claim
date: 2018-04-12
full_link: /jp/docs/concepts/storage/persistent-volumes/
short_description: >
  <!--Claims storage resources defined in a PersistentVolume so that it can be mounted as a volume in a container.-->
  記憶領域（ストレージ）リソースを PersistentVolume（持続型領域）として要求すると、コンテナではボリューム（領域）としてマウントできます。

aka: 
tags:
- core-object
- storage
---
 <!--Claims storage resources defined in a PersistentVolume so that it can be mounted as a volume in a container.-->
 記憶領域（ストレージ）リソースを PersistentVolume（持続型領域）として要求すると、コンテナではボリューム（領域）としてマウントできます。

<!--more--> 

<!--
Specifies the amount of storage, how the storage will be accessed (read-only, read-write and/or exclusive) and how it is reclaimed (retained, recycled or deleted). Details of the storage itself are in the PersistentVolume specification.
-->
記憶領域（ストレージ）の容量を指定するには、記憶領域（ストレージ）にどのように接続するか（読み込み専用、読み書き可能、あるいは両方いずれでもない）と、どのように要求するか（保持し続ける、再利用する、削除するか）によります。記憶領域（ストレージ）自身の詳細については PersistentVolume の仕様にあります。
