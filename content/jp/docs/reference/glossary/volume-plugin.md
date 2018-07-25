---
title: Volume Plugin（ボリューム・プラグイン）
id: volumeplugin
date: 2018-04-12
full_link: 
short_description: >
  <!--A Volume Plugin enables integration of storage within a Pod.-->ボリューム・プラグインはポッド内でのストレージ統合を有効化します。

aka: 
tags:
- core-object
- storage
---
<!--
 A Volume Plugin enables integration of storage within a {{< glossary_tooltip text="Pod" term_id="pod" >}}.
-->
ボリューム・プラグインは {{< glossary_tooltip text="ポッド" term_id="pod" >}} 内でのストレージ統合を有効化します。

<!--more--> 
<!--
A Volume Plugin lets you attach and mount storage volumes for use by a {{< glossary_tooltip text="Pod" term_id="pod" >}}. Volume plugins can be _in tree_ or _out of tree_. _In tree_ plugins are part of the Kubernetes code repository and follow its release cycle. _Out of tree_ plugins are developed independently.
-->
ボリューム・プラグインは  {{< glossary_tooltip text="ポッド" term_id="pod" >}}  が使うためのストレージ・ボリューム（記憶領域）をアタッチ（訳者注：領域をポッドから認識できるようにする）およびマウント（訳者注：ファイルシステムとして読み書き可能にする）します。
