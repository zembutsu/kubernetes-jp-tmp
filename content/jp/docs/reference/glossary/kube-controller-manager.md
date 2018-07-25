---
title: kube-controller-manager
id: kube-controller-manager
date: 2018-04-12
full_link: /jp/docs/reference/generated/kube-controller-manager/
short_description: >
  <!--Component on the master that runs controllers.-->
  マスタ上でコントローラを実行する構成要素（コンポーネント）です。

aka: 
tags:
- architecture
- fundamental
---
 <!--Component on the master that runs {{< glossary_tooltip text="controllers" term_id="controller" >}}.-->
 マスタ上で {{< glossary_tooltip text="コントローラ" term_id="controller" >}} を実行する構成要素（コンポーネント）です。

<!--more--> 

<!--
Logically, each {{< glossary_tooltip text="controller" term_id="controller" >}} is a separate process, but to reduce complexity, they are all compiled into a single binary and run in a single process.
-->
論理的には各 {{< glossary_tooltip text="コントローラ" term_id="controller" >}} は分離されたプロセスですが、複雑さを減らすために、すべてが１つのバイナリにコンパイルされ、１つのプロセスとして実行します。