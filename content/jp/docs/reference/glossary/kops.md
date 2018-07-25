---
title: Kops
id: kops
date: 2018-04-12
full_link: /jp/docs/getting-started-guides/kops/
short_description: >
  <!--A CLI tool that helps you create, destroy, upgrade and maintain production-grade, highly available, Kubernetes clusters. *NOTE&#58; Officially supports AWS only, with GCE and VMware vSphere in alpha*.--->
  本番段間での高可用性 Kubernetes クラスタを作成、破棄、更新、運用に役立つコマンドライン・ツールです。メモ：公式にサポートしているのは AWS のみであり、GCE と VMware vSphere はアルファ版です。

aka: 
tags:
- tool
- operation
---
 <!--A CLI tool that helps you create, destroy, upgrade and maintain production-grade, highly available, Kubernetes clusters. *NOTE&#58; Officially supports AWS only, with GCE and VMware vSphere in alpha*.-->
 本番段間での高可用性 Kubernetes クラスタを作成、破棄、更新、運用に役立つコマンドライン・ツールです。メモ：公式にサポートしているのは AWS のみであり、GCE と VMware vSphere はアルファ版です。

<!--more--> 

<!--
`kops` provisions your cluster with&#58;

  * Fully automated installation
  * DNS-based cluster identification
  * Self-healing&#58; everything runs in Auto-Scaling Groups
  * Limited OS support (Debian preferred, Ubuntu 16.04 supported, early support for CentOS & RHEL)
  * High availability (HA) support
  * The ability to directly provision, or generate terraform manifests

You can also build your own cluster using {{< glossary_tooltip term_id="kubeadm" >}} as a building block. `kops` builds on the kubeadm work.
-->
`kops` はクラスタに次のものを自動構築（プロビジョン）します：

  * 完全にインストールを自動化
  * DNS をベースとしたクラスタ認識
  * 自己修復（self-healing）：オートスケーリング・グループ内のすべてに対応
  * 限定的な OS のサポート（Debian が推奨、Ubuntu 16.04 はサポートされており、CentOS & RHEL は初期段階のサポート）
  * 高可用性（HA）のサポート
  * 直に自動構築（プロビジョン）するか Terrafrom マニフェストを作成する機能がある

{{< glossary_tooltip term_id="kubeadm" >}} を使って構築ブロック（building block）として自分でクラスタを構築できます。 `kops` は kubeadm の動作の上に成り立っています。