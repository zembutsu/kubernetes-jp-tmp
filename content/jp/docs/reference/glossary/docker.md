---
title: docker（ドッカー）
id: docker
date: 2018-04-12
full_link: /jp/docs/reference/kubectl/docker-cli-to-kubectl/
short_description: >
  <!-Docker is a software technology providing operating-system-level virtualization also known as containers.-->
  Docker とは、コンテナとして知られているオペレーティングシステム・レベルでの仮想化を提供するソフトウェア技術です。

aka: 
tags:
- fundamental
---
 <!--Docker is a software technology providing operating-system-level virtualization also known as containers.-->
 Docker とは、コンテナとして知られているオペレーティングシステム・レベルでの仮想化を提供するソフトウェア技術です。

<!--more--> 

<!--
Docker uses the resource isolation features of the Linux kernel such as cgroups and kernel namespaces, and a union-capable file system such as OverlayFS and others to allow independent "containers" to run within a single Linux instance, avoiding the overhead of starting and maintaining virtual machines (VMs).
-->
Docker はリソースを独立（分離：isolation）する機能のために Linux カーネルの cgorup と kernel 名前空間（namespace）と、OverlayFS のような統合可能なファイルシステムと、その他により、１つの Linux インスタンス（実体）の中に独立した *コンテナ（container）* として実行できるようにするものであり、仮想マシン（VM）を起動・運用するオーバーヘッドを防ぎます。