---
title: クラウド・コントローラ・マネージャの基礎概念
content_template: templates/concept
weight: 30
---

{{% capture overview %}}
<!--
The cloud controller manager (CCM) concept (not to be confused with the binary) was originally created to allow cloud specific vendor code and the Kubernetes core to evolve independent of one another. The cloud controller manager runs alongside other master components such as the Kubernetes controller manager, the API server, and scheduler. It can also be started as a Kubernetes addon, in which case it runs on top of Kubernetes.
-->
クラウド・コントローラ・マネージャ（CCM）の概念とは（プログラムとしてのバイナリと混同しないでください）、本来はクラウド固有のベンダー・コードと Kubernetes コアがお互い独立して開発を進められるようにするためでした。
クラウド・コントローラ・マネージャは Kubernetes コントローラ・マネージャ、API サーバ、スケジューラのような、他のマスタ構成要素と一緒に実行します。
また、Kubernetes アドオンとしても起動できるため、Kubernetes 上で実行する場合もあります。

<!--
The cloud controller manager's design is based on a plugin mechanism that allows new cloud providers to integrate with Kubernetes easily by using plugins. There are plans in place for on-boarding new cloud providers on Kubernetes and for migrating cloud providers from the old model to the new CCM model.
-->
クラウド・コントローラ・マネージャの設計はプラグイン機構に基づいた設計です。
そのため、クラウド事業者が Kubernetes と統合するにはプラグインを使うのが簡単です。

<!--
This document discusses the concepts behind the cloud controller manager and gives details about its associated functions.
-->
このドキュメントが扱うのはクラウド・コントローラ・マネージャの背後にある概念と、関連する機能に関する詳細を提供します。

<!--
Here's the architecture of a Kubernetes cluster without the cloud controller manager:
-->
ここは Kubernetes クラスタの構造（アーキテクチャ）を使いますが、クラウド・コントローラ・マネージャについては扱いません。

![Pre CCM Kube Arch](/images/docs/pre-ccm-arch.png)

{{% /capture %}}

{{< toc >}}

{{% capture body %}}

<!--
## Design
-->
## 設計 {#design}

<!--
In the preceding diagram, Kubernetes and the cloud provider are integrated through several different components:
-->
先ほどの図にあるように、Kubernetes とクラウド事業者は複数の異なる構成要素（コンポーネント）を通して統合されています。

<!--
* Kubelet
* Kubernetes controller manager
* Kubernetes API server
-->
* Kubelet
* Kubernetes コントローラ・マネージャ
* Kubernetes API サーバ

<!--
The CCM consolidates all of the cloud-dependent logic from the preceding three components to create a single point of integration with the cloud. The new architecture with the CCM looks like this:
-->
先ほどの３つの構成要素から、クラウドに１つの統合地点を作成するために、クラウドに依存する仕組み（ロジック）全てを CCM が集約します。

![CCM Kube Arch](/images/docs/post-ccm-arch.png)

<!--
## Components of the CCM
-->
## CCM の構成要素 {#components-of-the-ccm}

<!--
The CCM breaks away some of the functionality of Kubernetes controller manager (KCM) and runs it as a separate process. Specifically, it breaks away those controllers in the KCM that are cloud dependent. The KCM has the following cloud dependent controller loops:
-->
CCM は Kubernetes コントローラ・マネージャ（KCM）の複数ある機能から切り離されており、独立した機能として動作します。
特に、クラウドに依存する KCM 内のコントローラからは切り離されています。
KCM は以下のクラウドに依存する制御ループがあります：

<!--
 * Node controller
 * Volume controller
 * Route controller
 * Service controller
-->
 * ノード・コントローラ（Node controller）
 * ボリューム・コントローラ（Volume controller）
 * 径路・コントローラ（Route controller）
 * サービス・コントローラ（Service Controller）

<!--
In version 1.9, the CCM runs the following controllers from the preceding list:
-->
バージョン 1.9 では、CCM は先の一覧から、以下のコントローラを実行します。

<!--
* Node controller
* Route controller 
* Service controller
-->
 * ノード・コントローラ（Node controller）
 * 径路・コントローラ（Route controller）
 * サービス・コントローラ（Service Controller）

<!--
Additionally, it runs another controller called the PersistentVolumeLabels controller. This controller is responsible for setting the zone and region labels on PersistentVolumes created in GCP and AWS clouds. 
-->
さらに、PersistentVolumeLabels と呼ぶ別のコントローラも動かします。
このコントローラは GCP と AWS クラウド上で、 PersistentVolumes（持続ボリューム） 上にゾーンとリージョンのラベルを設定する役割があります。

{{< note >}}
<!--
**Note:** Volume controller was deliberately chosen to not be a part of CCM. Due to the complexity involved and due to the existing efforts to abstract away vendor specific volume logic, it was decided that volume controller will not be moved to CCM. 
-->
**メモ:** ボリューム・コントローラは CCM の一部としてではなく、意図的に選択する必要があります。
複雑さに伴う原因と、ベンダ固有のボリューム・ロジックを抽象化して切り離す既存の努力により、ボリューム・コントローラは CCM によって移動しないのが決定されました。

{{< /note >}}
<!--
The original plan to support volumes using CCM was to use Flex volumes to support pluggable volumes. However, a competing effort known as CSI is being planned to replace Flex. 
-->
本来はボリュームをサポートするために CCM を使う計画でした。
これはプラガブル（取り付け・取り外し可能）なボリュームをサポートするために、柔軟なボリュームを使うためでした。
ですが、CSI として知られている競合する取り組みは、柔軟なものへと置き換える計画がされました。

<--
Considering these dynamics, we decided to have an intermediate stop gap measure until CSI becomes ready.
-->
これらの動きを考慮し、CSI が利用可能になるよりも前に、私たちはただちに活動を停止しました。

<!--
## Functions of the CCM
-->
## CCM の機能 {#functions-of-the-ccm}

<!--
The CCM inherits its functions from components of Kubernetes that are dependent on a cloud provider. This section is structured based on those components.
-->
CCM は Kubernetes の構成要素から、クラウド事業者に依存する機能を継承します。
このセクションは構造化された各構成要素に基づいた構成です。

<!--
### 1. Kubernetes controller manager
-->
### 1. Kubernetes コントローラ・マネージャ

<!--
The majority of the CCM's functions are derived from the KCM. As mentioned in the previous section, the CCM runs the following control loops:
-->

<!--
* Node controller
* Route controller 
* Service controller
* PersistentVolumeLabels controller
-->
* ノード・コントローラ（Node controller）
* 径路（Route）・コントローラ（controller ）
* サービス・コントローラ（Service controller）
* 持続ボリューム・ラベル（PersistentVolumeLabels controller）

<!--
#### Node controller
-->
#### ノード・コントローラ（Node controller） {#node-controller}

<!--
The Node controller is responsible for initializing a node by obtaining information about the nodes running in the cluster from the cloud provider. The node controller performs the following functions:
-->
クラウド事業者からクラスタ内で実行しているノードに関する情報を取得するために、ノード・コントローラはノードの初期化に責任を持ちます。
ノード・コントローラは以下の機能を処理します：

<!--
1. Initialize a node with cloud specific zone/region labels.
2. Initialize a node with cloud specific instance details, for example, type and size.
3. Obtain the node's network addresses and hostname.
4. In case a node becomes unresponsive, check the cloud to see if the node has been deleted from the cloud.
If the node has been deleted from the cloud, delete the Kubernetes Node object.
-->
1. クラウド固有のゾーンやリージョンのラベルと共に、ノードを初期化します。
2. クラウド固有のインスタンスの詳細と共に、ノードを初期化します。例えば、種類（タイプ）や容量（サイズ）です。
3. ノードのネットワーク・アドレスとホスト名を取得します。
4. ノードの反応が無くなれば、クラウドでノードがクラウドから削除されていないかどうかを調べます。もしもクラウドからノードが削除されていれば、Kubernetes ノード・オブジェクトを削除します。

<!--
#### Route controller
-->
#### 径路コントローラ（Route controller）{#route-controller}

<!--
The Route controller is responsible for configuring routes in the cloud appropriately so that containers on different nodes in the Kubernetes cluster can communicate with each other. The route controller is only applicable for Google Compute Engine clusters.
-->
径路コントローラはクラウド内で適切な径路設定をする責任があります。
これにより、Kubernetes クラスタ内で異なったノード上にあるコンテナが、お互いに通信可能となります。
径路コントローラが適用できるのは Google Compute Engine クラスタのみです。

<!--
#### Service Controller
-->
#### サービス・コントローラ（Service Controller） {#service-controller}

<!--
The Service controller is responsible for listening to service create, update, and delete events. Based on the current state of the services in Kubernetes, it configures cloud load balancers (such as ELB or Google LB) to reflect the state of the services in Kubernetes. Additionally, it ensures that service backends for cloud load balancers are up to date.
-->
サービス・コントローラは、サービスの作成、更新、削除のイベントを受け付ける（listening）責任を持ちます。
Kubernetes 内にあるサービスの現在の状態（current state）に基づき、クラウド負荷分散装置（ロードバランサ）（ELB や Google LB）に Kubernetes 内のサービス状況を反映するための設定をします。
さらに、クラウド負荷分散装置（ロードバランサ）が更新されても、サービスのバックエンドを安全にします（確保します）。

<!--
#### PersistentVolumeLabels controller
-->
#### 持続ボリューム・ラベル・コントローラ（PersistentVolumeLabels controller）{#persistentvolumelabels-controller}

<!--
The PersistentVolumeLabels controller applies labels on AWS EBS/GCE PD volumes when they are created. This removes the need for users to manually set the labels on these volumes. 
-->
持続ボリューム・ラベル・コントローラは、ボリュームの作成時に AWS EBS/GCE PD ボリューム上にラベルを適用します。
ボリュームに対して設定したラベルは、ユーザの必要があれば手動で削除する必要があります。

<!--
These labels are essential for the scheduling of pods as these volumes are constrained to work only within the region/zone that they are in. Any Pod using these volumes needs to be scheduled in the same region/zone.
-->
これらのラベルは、本質的にポッドのスケジューリングのためです。各ボリュームは、リージョンまたはゾーン内でのみボリュームが動作するように制限されています。

<!_-
The PersistentVolumeLabels controller was created specifically for the CCM; that is, it did not exist before the CCM was created. This was done to move the PV labelling logic in the Kubernetes API server (it was an admission controller) to the CCM. It does not run on the KCM.
-->
持続ボリューム・ラベル・コントローラはCCM によって特別に作成されます。
つまり、CCM によって作成されるまで、これは存在しません。
この処理を行う PV ラベリング論理（ロジック）は、Kubernetes PI サーバ（以前はアドミッション・コントローラでした）から CCM に移動しました。
これは KCM 上では実行されません。

### 2. Kubelet

<!--
The Node controller contains the cloud-dependent functionality of the kubelet. Prior to the introduction of the CCM, the kubelet was responsible for initializing a node with cloud-specific details such as IP addresses, region/zone labels and instance type information. The introduction of the CCM has moved this initialization operation from the kubelet into the CCM. 
-->
ノード・コントローラには kubelet のクラウドに依存する機能を含んでいます。
CCM が導入される以前は、kubelet はノードの初期化に責任があり、ここにクラウド固有の詳細も含んでいました。たとえば IP アドレス、リージョンやゾーン、ラベル、インフタンス型などの情報です。
CCM の導入によって、初期化作業は kubelet から CCM へと移行しました。

<!--
In this new model, the kubelet initializes a node without cloud-specific information. However, it adds a taint to the newly created node that makes the node unschedulable until the CCM initializes the node with cloud-specific information. It then removes this taint.
-->
この新しいモデルでは、kubelet がノードを初期化するにあたり、クラウド固有の情報を含みません。
しかし、新しく作成されたノードに対しては故障（テイント：taint）を追加します。
これは CCM がクラウド固有の情報と共にノードを初期化するまでは、ノードがスケジュール不可能とするものです。
その後、故障（テイント）を削除します。

<!--
### 3. Kubernetes API server
-->
### 3. Kubernetes API サーバ

<!--
The PersistentVolumeLabels controller moves the cloud-dependent functionality of the Kubernetes API server to the CCM as described in the preceding sections.
-->
持続ボリューム・ラベル・コントローラの、 Kubernetes API サーバのクラウドに依存する機能を、前述の通り CCM に移行しました。

<!--
## Plugin mechanism
-->
## プラグイン構造 {#plugin-mechanism}

<!--
The cloud controller manager uses Go interfaces to allow implementations from any cloud to be plugged in. Specifically, it uses the CloudProvider Interface defined [here](https://github.com/kubernetes/kubernetes/blob/master/pkg/cloudprovider/cloud.go).
-->
クラウド・コントローラ・マネージャは Go インターフェースを使い、あらゆるクラウドを差し込んで（plugged in）実装します。
特に、CloudProvider インターフェースを使うものは[こちら](https://github.com/kubernetes/kubernetes/blob/master/pkg/cloudprovider/cloud.go) に定義されています。

<!--
The implementation of the four shared controllers highlighted above, and some scaffolding along with the shared cloudprovider interface, will stay in the Kubernetes core. Implementations specific to cloud providers will be built outside of the core and implement interfaces defined in the core.
-->
４つの共有コントローラの実装に関するハイライトは、前述の通りです。
そして、いくつかの足場（scaffolding）は共有クラウドプロバイダ・インタフェースと一緒に Kubernetes コアの中にあります。
クラウド事業者に対する特別な実装は、コアの外で構築されており、実装のためのインターフェースはコアの中で定義されています。

<!--
For more information about developing plugins, see [Developing Cloud Controller Manager](/docs/tasks/administer-cluster/developing-cloud-controller-manager/).
ｰｰ>
プラグイン開発に関する詳細は、[クラウド・コントローラ・マネージャの開発](/jp/docs/tasks/administer-cluster/developing-cloud-controller-manager/) をご覧ください。

<!--
## Authorization
-->
## 承認（Authorization）{#authorization}

<!--
This section breaks down the access required on various API objects by the CCM to perform its operations. 
->
このセクションは CCM が処理する動作によって、様々な API オブジェクト上でのアクセスに必要なものを掘り下げます（ブレイクダウンします）。

<!--
### Node Controller
-->
### ノード・コントローラ {#node-controller}

<!--
The Node controller only works with Node objects. It requires full access to get, list, create, update, patch, watch, and delete Node objects.
-->
ノード・コントローラが操作するのはノード・オブジェクトのみです。
ノード・コントローラはノード・オブジェクトに対する get、list、create、update、patch、watch、delete に対するフル・アクセスが必要です。

v1/Node: 

- Get
- List
- Create
- Update
- Patch
- Watch
- Delete

<!--
### Route controller
-->
### 径路コントローラ {#route-controller}

<!--
The route controller listens to Node object creation and configures routes appropriately. It requires get access to Node objects. 
-->
径路コントローラはノード・オブジェクトの作成と適切な径路設定を受け付けます。
これにはノード・オブジェクトに対する get アクセスが必要です。

v1/Node: 

- Get

<!--
### Service controller
-->
### サービス・コントローラ {#service-controller}

<!--
The service controller listens to Service object create, update and delete events and then configures endpoints for those Services appropriately. 
-->
サービス・コントローラはサービス・オブジェクトの create、update、delete イベントと、これらサービスのエンドポイントに対する適切な設定を受け付けます。

<!--
To access Services, it requires list, and watch access. To update Services, it requires patch and update access. 
-->
サービスにアクセスするためには、list と watch 権限が必要です。
サービスを更新するには、patch と update 権限が必要です。

<!--
To set up endpoints for the Services, it requires access to create, list, get, watch, and update.
-->
サービスのエンドポイントをセットアップするには、create、list、get、watch、update 権限が必要です。

v1/Service:

- List
- Get
- Watch
- Patch
- Update

<!--
### PersistentVolumeLabels controller
-->
### 持続ボリューム・ラベル・コントローラ {#persistentvolumelabels-controller}

<!--
The PersistentVolumeLabels controller listens on PersistentVolume (PV) create events and then updates them. This controller requires access to get and update PVs.
-->
持続ボリューム・ラベル・コントローラは、持続ボリューム（PV）上での create イベントと update を受け付けます。
このコントローラは持続ボリュームに対する get と update 権限が必要です。

v1/PersistentVolume:

- Get
- List
- Watch
- Update

<!--
### Others
-->
### その他 {#others}

<!--
The implementation of the core of CCM requires access to create events, and to ensure secure operation, it requires access to create ServiceAccounts.
-->
CCM のコア実装に対する必要な権限は create イベントです。安全な作業のためには ServiceAccount（サービス・アカウント）を作成する権限が必要です。

v1/Event:

- Create
- Patch
- Update

v1/ServiceAccount:

- Create

<!--
The RBAC ClusterRole for the CCM looks like this:
-->
CCM に対する RBAC ClusterRole は以下のようなものです：

```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRole
metadata:
  name: cloud-controller-manager
rules:
- apiGroups:
  - ""
  resources:
  - events
  verbs:
  - create
  - patch
  - update
- apiGroups:
  - ""
  resources:
  - nodes
  verbs:
  - '*'
- apiGroups:
  - ""
  resources:
  - nodes/status
  verbs:
  - patch
- apiGroups:
  - ""
  resources:
  - services
  verbs:
  - list
  - patch
  - update
  - watch
- apiGroups:
  - ""
  resources:
  - serviceaccounts
  verbs:
  - create
- apiGroups:
  - ""
  resources:
  - persistentvolumes
  verbs:
  - get
  - list
  - update
  - watch
- apiGroups:
  - ""
  resources:
  - endpoints
  verbs:
  - create
  - get
  - list
  - watch
  - update
```

<!--
## Vendor Implementations
-->
## ベンダ実装 {#vender-implementations}

<!--
The following cloud providers have implemented CCMs: 
-->
以下のクラウド事業者は CCM を実装しています：

* Digital Ocean
* [Oracle](https://github.com/oracle/oci-cloud-controller-manager)
* Azure
* GCE
* AWS

<!--
## Cluster Administration
-->
## クラスタ管理 {#cluster-administration}

<!--
Complete instructions for configuring and running the CCM are provided
[here](/docs/tasks/administer-cluster/running-cloud-controller/#cloud-controller-manager).
-->
CCM の設定と実行に関する全面的な手順は [こちら](/jp/docs/tasks/administer-cluster/running-cloud-controller/#cloud-controller-manager) にあります。


{{% /capture %}}
