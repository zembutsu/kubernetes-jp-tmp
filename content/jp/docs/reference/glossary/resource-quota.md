---
title: Resource Quotas（リソース制限）
id: resource-quota
date: 2018-04-12
full_link: /jp/docs/concepts/policy/resource-quotas/
short_description: >
  <!--Provides constraints that limit aggregate resource consumption per namespace.-->
  名前空間ごとに、リソース使用量の合計に上限を設けます。

aka: 
tags:
- fundamental
- operation
- architecture
---
 <!--Provides constraints that limit aggregate resource consumption per {{< glossary_tooltip term_id="namespace" >}}.-->
 {{< glossary_tooltip text="名前空間" term_id="namespace" >}} ごとに、リソース使用量の合計に上限を設けます。

<!--more--> 

<!--
Limits the quantity of objects that can be created in a namespace by type, as well as the total amount of compute resources that may be consumed by resources in that project.
-->
オブジェクトの量の上限とは、名前空間内で作成されるものだけでなく、プロジェクト内のリソースによって消費される可能性がある合計リソース量も含みます。
