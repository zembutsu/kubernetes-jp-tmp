---
title: Controller（コントローラ）
id: controller
date: 2018-04-12
full_link: /jp/docs/admin/kube-controller-manager/
short_description: >
  <!--A control loop that watches the shared state of the cluster through the apiserver and makes changes attempting to move the current state towards the desired state.-->
  クラスタ共有状態の監視を通しながら、apiserver とクラスタの現在状態を、期待状態へと移行を試みる制御ループ（control loop）です。

aka: 
tags:
- architecture
- fundamental
---
 <!--A control loop that watches the shared state of the cluster through the {{< glossary_tooltip text="apiserver" term_id="kube-apiserver" >}} and makes changes attempting to move the current state towards the desired state.-->
 クラスタ共有状態の監視を通しながら、 {{< glossary_tooltip text="apiserver" term_id="kube-apiserver" >}} とクラスタの現在状態を、期待状態へと移行を試みる制御ループ（control loop）です。

<!--more--> 

<!--
Examples of controllers that ship with Kubernetes today are the replication controller, endpoints controller, namespace controller, and serviceaccounts controller.
-->
現在 Kubernetes が提供しているコントローラには、replication controller（レプリケーション・コントローラ）、endpoints controller（エンドポイント・コントローラ）、namespace controller（名前空間コントローラ）、serviceaccounts controler（サービスアカウント・コントローラ）があります。