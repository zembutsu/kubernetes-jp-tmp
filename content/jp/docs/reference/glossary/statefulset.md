---
title: StatefulSet（ステートフルセット）
id: statefulset
date: 2018-04-12
full_link: /jp/docs/concepts/workloads/controllers/statefulset/
short_description: >
  <!--Manages the deployment and scaling of a set of Pods, *and provides guarantees about the ordering and uniqueness* of these Pods.-->
  ポッドの集まりの展開（デプロイ）と拡大縮小（スケーリング）を管理し、 *かつ、順序づけ（ordering）と一意性（uniqueness）の保証* をポッドに対して行います。

aka: 
tags:
- fundamental
- core-object
- workload
- storage
---
 <!--Manages the deployment and scaling of a set of {{< glossary_tooltip text="Pods" term_id="pod" >}}, *and provides guarantees about the ordering and uniqueness* of these Pods.-->
 {{< glossary_tooltip text="ポッド（Pod）" term_id="pod" >}}の集まりの展開（デプロイ）と拡大縮小（スケーリング）を管理し、 *かつ、順序づけ（ordering）と一意性（uniqueness）の保証* をポッドに対して行います。

<!--more--> 

<!--
Like a {{< glossary_tooltip term_id="deployment" >}}, a StatefulSet manages Pods that are based on an identical container spec. Unlike a Deployment, a StatefulSet maintains a sticky identity for each of their Pods. These pods are created from the same spec, but are not interchangeable&#58; each has a persistent identifier that it maintains across any rescheduling.

A StatefulSet operates under the same pattern as any other Controller. You define your desired state in a StatefulSet *object*, and the StatefulSet *controller* makes any necessary updates to get there from the current state.
-->
{{< glossary_tooltip text="Deployment（デプロイメント）" term_id="deployment" >}} のように、StatefulSet は同一のコンテナ spec をベースにしているポッドを管理します。Deployment と違うのは、StatefulSet は各ポッドの厄介な自己同一性を管理します。ポッドは同じ spec から作成されていますが、お互いを交換できません。つまり、それぞれのポッドは再スケジューリングの間でも一貫した識別子を持ちます。

StetefulSet は他のあらゆる Controller（コントローラ）と同じパターンの元で稼働します。StatefulSet *オブジェクト* で期待状態（desired state）を定義し、 現在の状態からそこ（期待状態）に到達するよう必要な更新を StatefulSet *コントローラ*  が行います。