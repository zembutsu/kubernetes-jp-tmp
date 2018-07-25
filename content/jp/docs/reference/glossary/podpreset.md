---
title: PodPreset（ポッド・プリセット）
id: podpreset
date: 2018-04-12
full_link: 
short_description: >
  <!--An API object that injects information such as secrets, volume mounts, and environment variables into pods at creation time.-->
  シークレットやボリューム・マウントや環境変数といった情報を、ポッドの作成時に投入するための API オブジェクトです。

aka: 
tags:
- operation
---
 <!--An API object that injects information such as secrets, volume mounts, and environment variables into pods at creation time.-->
 シークレットやボリューム・マウントや環境変数といった情報を、ポッドの作成時に投入するための API オブジェクトです。

<!--more--> 

<!--
This object chooses the pods to inject information into using standard selectors. This allows the podspec definitions to be nonspecific, decoupling the podspec from environment specific configuration.
-->
このオブジェクトでは標準セレクタを使って情報を導入するためのポッドを選択します。これにより podspec 定義は環境に依存せず、環境固有の設定とは切り離せるようにします。