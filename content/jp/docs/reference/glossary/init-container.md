---
title: Init Container（初期化コンテナ）
id: init-container
date: 2018-04-12
full_link: 
short_description: >
  <!--One or more initialization containers that must run to completion before any app containers run. -->
  １つまたは複数の初期化コンテナは、あらゆるアプリケーション・コンテナを実行する前に終了する必要があります。

aka: 
tags:
- fundamental
---
 <!--One or more initialization containers that must run to completion before any app containers run. -->
 １つまたは複数の初期化コンテナは、あらゆるアプリケーション・コンテナを実行する前に終了する必要があります。

<!--more--> 

<!--
Initialization (init) containers are like regular app containers, with one difference: init containers must run to completion before any app containers can start. Init containers run in series: each init container must run to completion before the next init container begins.  
-->
初期化（init）コンテナとは通常のアプリケーション・コンテナのようなものですが、１点が異なります。それは初期化コンテナはあらゆるアプリケーション・コンテナを開始するまでに実行を完了しなくてはいけません。初期化コンテナは連続して実行します。つまり、各初期化コンテナは次の初期化コンテナを実行する前に終了する必要があります。
