---
title: Network Policy（ネットワーク・ポリシー）
id: network-policy
date: 2018-04-12
full_link: /jp/docs/concepts/services-networking/network-policies/
short_description: >
  <!--A specification of how groups of Pods are allowed to communicate with each other and with other network endpoints.-->
  ポッドのグループがどのようにしてお互いに通信するか、あるいは、他のネットワーク・エンドポイントと通信するかの指定です。

aka: 
tags:
- networking
- architecture
- extension
---
 <!--A specification of how groups of Pods are allowed to communicate with each other and with other network endpoints.-->
 ポッドのグループがどのようにしてお互いに通信するか、あるいは、他のネットワーク・エンドポイントと通信するかの指定です。

<!--more--> 

<!--
Network Policies help you declaratively configure which Pods are allowed to connect to each other, which namespaces are allowed to communicate, and more specifically which port numbers to enforce each policy on. `NetworkPolicy` resources use labels to select Pods and define rules which specify what traffic is allowed to the selected Pods. Network Policies are implemented by a supported network plugin provided by a network provider. Be aware that creating a network resource without a controller to implement it will have no effect.
-->
どのポッドがお互いに通信できるかどうか、どの名前空間との通信が許可されているか、どのポート番号に対して各ポリシーを適用するかといった設定をするために、ネットワーク・ポリシーの宣言型の設定が役立ちます。 `NetworkPolicy` リソースは対象ポッドを選択するのにラベルを使い、選択したポッドに対しては、何の通信を許可するかを指定するルールを定義します。ネットワーク・ポリシーの実装はネットワーク・プロバイダが提供するネットワーク・プラグインがサポートしているものです。ネットワーク・リソースの作成にあたり、コントローラの実装をしていしなければ、何の影響も及ぼさないのでご注意ください。
