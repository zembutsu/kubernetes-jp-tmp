---
title: ConfigMap（コンフィグマップ）
id: configmap
date: 2018-04-12
full_link: /docs/tasks/configure-pod-container/configure-pod-configmap/
short_description: >
  <!--An API object used to store non-confidential data in key-value pairs. Can be consumed as environment variables, command-line arguments, or config files in a volume.-->
  機密ではないデータをキーバリューの組で保管するための API オブジェクトです。用途は環境変数、コマンドラインの引数、ボリュームにおける設定ファイルなどです。

aka: 
tags:
- core-object
---
 <!--An API object used to store non-confidential data in key-value pairs. Can be consumed as environment variables, command-line arguments, or config files in a {{< glossary_tooltip text="volume" term_id="volume" >}}.-->
 機密ではないデータをキーバリューの組で保管するための API オブジェクトです。用途は環境変数、コマンドラインの引数、{{< glossary_tooltip text="ボリューム" term_id="volume" >}}における設定ファイルなどです。

<!--more--> 

<!--
Allows you to decouple environment-specific configuration from your {{< glossary_tooltip text="container images" term_id="container" >}}, so that your applications are easily portable. When storing confidential data use a [Secret](https://kubernetes.io/docs/concepts/configuration/secret/).
-->
 {{< glossary_tooltip text="コンテナ・イメージ" term_id="container" >}} と環境固有の設定を分離できるので、アプリケーションを簡単に移動可能（ポータブル）にします。機密データを保管する場合は [Secret（シークレット）](/jp/docs/concepts/configuration/secret/) を使います。