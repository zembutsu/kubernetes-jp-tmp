---
title: 概念
main_menu: true
weight: 40
---

<!---
The Concepts section helps you learn about the parts of the Kubernetes system and the abstractions Kubernetes uses to represent your cluster, and helps you obtain a deeper understanding of how Kubernetes works.
-->
概念（concept；コンセプト）のセクションは Kubernetes システムの各部（パーツ）を学ぶのに役立ちます。また、Kubernetes 抽象化は自分のクラスタを説明するのに使えます。さらに、Kubernetes がどのように動作しているか、深い理解を得るのに役立ちます。

<!--
## Overview
-->
## 概要 {#overview}

<!---
To work with Kubernetes, you use *Kubernetes API objects* to describe your cluster's *desired state*: what applications or other workloads you want to run, what container images they use, the number of replicas, what network and disk resources you want to make available, and more. You set your desired state by creating objects using the Kubernetes API, typically via the command-line interface, `kubectl`. You can also use the Kubernetes API directly to interact with the cluster and set or modify your desired state.
-->
Kubenetes を動かすためには、*Kubernetes API オブジェクト* を使ってクラスタの *期待状態（desired state）* を記述します。期待状態とは、実行しようとするアプリケーションや他のワークロード（訳注：workload = 作業負荷、仕事量）が何であるか、利用するコンテナ・イメージが何か、複製（レプリカ）数はいくつか、提供するネットワークおよびディスクリソースは何か、などです。期待状態を設定するには、 Kubernetes API を使ってオブジェクトを作成します。Kubernetes API を使うには、通常はコマンドライン・インターフェース `kubectl` を通します（via：経由します）。あるいは、Kubernetes API を直接使っても、クラスタや期待状態を設定・変更できます。

<!--
Once you've set your desired state, the *Kubernetes Control Plane* works to make the cluster's current state match the desired state. To do so, Kubernetes performs a variety of tasks automatically--such as starting or restarting containers, scaling the number of replicas of a given application, and more. The Kubernetes Control Plane consists of a collection of processes running on your cluster: 
-->
いったん期待様態を設定すると、 *Kubernetes コントロール・プレーン（Control Plane）* がクラスタの現行状態（current state）が期待状態（desired state）と一致するように動作します。そのためには、Kubernetes が様々な仕事（タスク）を自動的に処理します。例えば、コンテナの起動や再起動、あるアプリケーションの複製（レプリカ数）を増減（スケール）するなどです。Kubernetes コントロール・プレーンはクラスタ上ので動作するプロセスの集合で構成されています：

<!--
* The **Kubernetes Master** is a collection of three processes that run on a single node in your cluster, which is designated as the master node. Those processes are: [kube-apiserver](/docs/admin/kube-apiserver/), [kube-controller-manager](/docs/admin/kube-controller-manager/) and [kube-scheduler](/docs/admin/kube-scheduler/).
* Each individual non-master node in your cluster runs two processes:
  * **[kubelet](/docs/admin/kubelet/)**, which communicates with the Kubernetes Master.
  * **[kube-proxy](/docs/admin/kube-proxy/)**, a network proxy which reflects Kubernetes networking services on each node.
-->
* **Kubernetes Master（マスタ）** は３つのプロセスの集まりであり、　クラスタの中にノード（node）が１つであれば、こちらがマスタ・ノードとして指定されます。３つのプロセスとは、[kube-apiserver](/jp/docs/admin/kube-apiserver/)（api サーバ）、 [kube-controller-manager](/jp/docs/admin/kube-controller-manager/)（コントローラ・マネージャ）、 [kube-scheduler](/jp/docs/admin/kube-scheduler/)（スケジューラ） です。
* クラスタ内にあるマスタ以外の各ノードでは、２つのプロセスを実行します：
  * **[kubelet](/jp/docs/admin/kubelet/)** とは、Kubernetes マスタと通信します。
  * **[kube-proxy](/jp/docs/admin/kube-proxy/)** とはネットワーク・プロキシ（proxy：代理）であり、Kubrnetes ネットワーク・サービスを各ノードに反映します。

<!--
## Kubernetes Objects
-->
## Kubernetes オブジェクト {#kubernetes-objects}

<!--
Kubernetes contains a number of abstractions that represent the state of your system: deployed containerized applications and workloads, their associated network and disk resources, and other information about what your cluster is doing. These abstractions are represented by objects in the Kubernetes API; see the [Kubernetes Objects overview](/docs/concepts/abstractions/overview/) for more details. 
-->
システムの状態を説明するために、Kubernetes には多くの抽象概念があります。コンテナ化したアプリケーションと作業負荷の配置（デプロイ）、それらに関連したネットワークとディスク資源（リソース）、そして、クラスタが何をしているかに関するその他の情報です。これらの抽象概念を Kubernetes API のオブジェクトで表現します（説明します）。こちらの詳細については [Kubernetes オブジェクト概要](/jp/docs/concepts/abstractions/overview/) をご覧ください。

<!--
The basic Kubernetes objects include:
-->
基本 Kubernetes オブジェクトに含まれるのは：

<!--
* [Pod](/docs/concepts/workloads/pods/pod-overview/)
* [Service](/docs/concepts/services-networking/service/)
* [Volume](/docs/concepts/storage/volumes/)
* [Namespace](/docs/concepts/overview/working-with-objects/namespaces/)
-->
* [Pod（ポッド）](/jp/docs/concepts/workloads/pods/pod-overview/)
* [Service（サービス）](/jp/docs/concepts/services-networking/service/)
* [Volume（ボリューム）](/jp/docs/concepts/storage/volumes/)
* [Namespace（名前空間）](/jp/docs/concepts/overview/working-with-objects/namespaces/)

<!--
In addition, Kubernetes contains a number of higher-level abstractions called Controllers. Controllers build upon the basic objects, and provide additional functionality and convenience features. They include:
-->
さらに、Kubernetes にはコントローラ（Controllers）と呼ばれる上位の抽象概念がたくさんあります。コントローラは基本オブジェクトを積み上げたものであり、機能的で便利な付加機能を提供します。ここに含まれるのは、以下の通りです：

<!--
* [ReplicaSet](/docs/concepts/workloads/controllers/replicaset/)
* [Deployment](/docs/concepts/workloads/controllers/deployment/)
* [StatefulSet](/docs/concepts/workloads/controllers/statefulset/)
* [DaemonSet](/docs/concepts/workloads/controllers/daemonset/)
* [Job](/docs/concepts/workloads/controllers/jobs-run-to-completion/)
-->
* [ReplicaSet（レプリカ・セット）](/jp/docs/concepts/workloads/controllers/replicaset/)
* [Deployment（デプロイメント）](/jp/docs/concepts/workloads/controllers/deployment/)
* [StatefulSet（ステートフル・セット）](/jp/docs/concepts/workloads/controllers/statefulset/)
* [DaemonSet（デーモン・セット）](/jp/docs/concepts/workloads/controllers/daemonset/)
* [Job（ジョブ）](/jp/docs/concepts/workloads/controllers/jobs-run-to-completion/)

<!--
## Kubernetes Control Plane
-->
## Kubernetes コントロール・プレーン（Control Plane） {#kubernetes-control-plane}

<!--
The various parts of the Kubernetes Control Plane, such as the Kubernetes Master and kubelet processes, govern how Kubernetes communicates with your cluster. The Control Plane maintains a record of all of the Kubernetes Objects in the system, and runs continuous control loops to manage those objects' state. At any given time, the Control Plane's control loops will respond to changes in the cluster and work to make the actual state of all the objects in the system match the desired state that you provided.
-->
Kubernetes マスタや kubelet プロセスなど、様々な部品（パーツ）で構成される Kubernetes コントロール・プレーンは、クラスタ内で Kubernetes の通信方法を司ります（govern：管理します）。コントロール・プレーンは、システム上にある Kubernetes オブジェクトすべての記録を保持します。そして各オブジェクトの状態を管理するために、制御ループを継続的に実行します。いかなる時においても、コントロール・プレーンの制御ループはクラスタの変更に対応し、システム上のオブジェクト状態すべてが、指定していた期待状態と一致するような動作をします。

<!--
For example, when you use the Kubernetes API to create a Deployment object, you provide a new desired state for the system. The Kubernetes Control Plane records that object creation, and carries out your instructions by starting the required applications and scheduling them to cluster nodes--thus making the cluster's actual state match the desired state.
-->
たとえば、Kubernetes API で Depoyment（デプロイメント）オブジェクトを作成するには、新しい期待状態をシステムに与えます。それから Kubernetes コントロール・プレーンはオブジェクトの作成を記録します。そして、（期待状態に対する）指示を実施するため、クラスタ・ノードで必要なアプリケーションの開始およびスケジューリングも行います。それゆえに、実際のクラスタ状態が期待状態と一致するのです。

<!--
### Kubernetes Master
-->
### Kubernetesマスタ {#kubernetes-master}

<!--
The Kubernetes master is responsible for maintaining the desired state for your cluster. When you interact with Kubernetes, such as by using the `kubectl` command-line interface, you're communicating with your cluster's Kubernetes master.
-->
Kubernetes マスタはクラスタに対する期待状態を維持する責任を負います。 Kubernetes とやりとりする場合は、`kubectl` コマンドライン・インターフェースを使うなどして、クラスタの Kubernetes マスタと通信をやりとりします。

<!--
> The "master" refers to a collection of processes managing the cluster state.  Typically these processes are all run on a single node in the cluster, and this node is also referred to as the master. The master can also be replicated for availability and redundancy.
-->
「マスタ」（master）として言及されるものは、クラスタ状態を管理するプロセスの集まりです。通常はクラスタにある１つのノード上で、３つのプロセスが稼働しています。そして、このノードこそがマスタとして見なされます。また、可用性（availability）と冗長性（redundancy）のためにマスタを複製できます。

<!--
### Kubernetes Nodes
-->
### Kubernetes ノード {#kubernetes-nodes}

<!--
The nodes in a cluster are the machines (VMs, physical servers, etc) that run your applications and cloud workflows. The Kubernetes master controls each node; you'll rarely interact with nodes directly.
-->
クラスタ内におけるノードとは、アプリケーションとクラウド作業負荷（ワークロード）を実行するマシン（仮想マシン、物理サーバ、など）です。Kubernetes マスタは各ノードを管理します。つまり、ノードと直接やりとりするのは稀です。

<!--
#### Object Metadata
-->
#### オブジェクト・メタデータ {#object-medatata}


* [注釈](/jp/docs/concepts/overview/working-with-objects/annotations/)

<!--
### What's next
-->
### 次は

<!--
If you would like to write a concept page, see
[Using Page Templates](/docs/home/contribute/page-templates/)
for information about the concept page type and the concept template.
-->
もしも概要ページに書きたければ、[Using Page Templates](/docs/home/contribute/page-templates/) をご覧ください。概要ページのタイプと概要テンプレートに関する情報があります。
