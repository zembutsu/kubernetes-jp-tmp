---
title: Container Environment Variables（コンテナ環境変数）
id: container-env-variables
date: 2018-04-12
full_link: /jp/docs/concepts/containers/container-environment-variables.md
short_description: >
  <!--Container environment variables are name/value pairs that provide useful information into containers running in a Pod.-->
  コンテナ環境変数は名前と値の組であり、ポッドの中で実行するコンテナ内で使う情報を提供します。

aka: 
tags:
- fundamental
---
 <!--Container environment variables are name/value pairs that provide useful information into containers running in a Pod.-->
 コンテナ環境変数は名前と値の組であり、ポッドの中で実行するコンテナ内で使う情報を提供します。

<!--more--> 

<!--
Container environment variables provide information that is required by the running containerized applications along with information about important resources to the [Containers] {{< glossary_tooltip text="Containers" term_id="container" >}}. For example, file system, information about the container itself and other cluster resources such as service endpoints, etc.
-->
コンテナ環境変数が提供する情報とは、[コンテナ] {{< glossary_tooltip text="コンテナ" term_id="container" >}} に対して、コンテナ化したアプリケーションの実行に必要な、リソースに関する重要な情報と共に、

コンテナ環境変数が提供する情報とは、


コンテナ化したアプリケーションの実行に必要な情報と共に、[コンテナ] {{< glossary_tooltip text="コンテナ" term_id="container" >}} に対する重要な情報をコンテナ環境変数を通して提供します。たとえば、ファイルシステムや、コンテナ自身の情報や、サービスのendポイントなど他のクラスタ・リソースなどです。