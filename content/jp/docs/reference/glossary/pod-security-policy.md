---
title: Pod Security Policy（ポッド・セキュリティ・ポリシー）
id: pod-security-policy
date: 2018-04-12
full_link: /jp/docs/concepts/policy/pod-security-policy/
short_description: >
  <!--Enables fine-grained authorization of pod creation and updates.-->
  ポッドの作成と更新時に粒度の高い認証を有効化します。

aka: 
tags:
- core-object
- fundamental
---
 <!--Enables fine-grained authorization of {{< glossary_tooltip term_id="pod" >}} creation and updates.-->
 ポッドの作成と更新時に、粒度の高い認証を有効化します。

<!--more--> 

<!--
A cluster-level resource that controls security sensitive aspects of the Pod specification. The `PodSecurityPolicy` objects define a set of conditions that a Pod must run with in order to be accepted into the system, as well as defaults for the related fields. Pod Security Policy control is implemented as an optional admission controller.
-->
クラスタ・レベルのリソースで、セキュリティ管理に細心の注意を払うべき対象がポッドの設計です。 `PodSecurityPolicy` オブジェクトで、システムに受け入れられるようにするためにポッドの実行を必須にしたり、関連するフィールドをデフォルトにしたり、一連の状況を定義します。ポッド・セキュリティ・ポリシーの管理はオプションのコントローラと並んで実装されています。