---
title: Volume（ボリューム）
id: volume
date: 2018-04-12
full_link: /docs/concepts/storage/volumes/
short_description: >
  <!--A directory containing data, accessible to the containers in a pod.-->データを収容するディレクトリであり、ポッド内のコンテナが利用できます。

aka: 
tags:
- core-object
- fundamental
---
<!--
 A directory containing data, accessible to the containers in a {{< glossary_tooltip text="pod" term_id="pod" >}}.
-->
データを収容するディレクトリであり、{{< glossary_tooltip text="ポッド" term_id="pod" >}} 内のコンテナが利用できます。

<!--more--> 
<!--
A Kubernetes volume lives as long as the {{< glossary_tooltip text="pod" term_id="pod" >}} that encloses it. Consequently, a volume outlives any {{< glossary_tooltip text="containers" term_id="container" >}} that run within the {{< glossary_tooltip text="pod" term_id="pod" >}}, and data is preserved across {{< glossary_tooltip text="container" term_id="container" >}} restarts. 
-->
Kubernetes ボリュームは {{< glossary_tooltip text="ポッド" term_id="pod" >}} に取り込まれている限り存続します。従って、ボリュームはどの {{< glossary_tooltip text="コンテナ" term_id="container" >}} よりも長く存在します。そのため、 {{< glossary_tooltip text="ポッド" term_id="pod" >}} 内にあるボリュームは、{{< glossary_tooltip text="コンテナ" term_id="container" >}} を再起動する間もデータを維持します。
