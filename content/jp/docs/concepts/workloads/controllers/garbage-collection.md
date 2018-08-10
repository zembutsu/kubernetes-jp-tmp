---
title: ガベージ・コレクション
content_template: templates/concept
weight: 60
---

{{% capture overview %}}

<!--
The role of the Kubernetes garbage collector is to delete certain objects
that once had an owner, but no longer have an owner.
-->

Kubernetes ガベージ・コレクタ（garbage collector）の役割は、かつては所有者がいたものの、現在は所有者がいないオブジェクトの削除です。

<!--
**Note**: Garbage collection is a beta feature and is enabled by default in
Kubernetes version 1.4 and later.
-->

**メモ：** ガベージ・コレクションはベータ機能であり、デフォルトで有効なのは Kubernetes 1.4 以降です。

{{% /capture %}}


{{% capture body %}}

<!--
## Owners and dependents
-->
## 所有者と従属 {#owners-and-dependents}

<!--
Some Kubernetes objects are owners of other objects. For example, a ReplicaSet
is the owner of a set of Pods. The owned objects are called *dependents* of the
owner object. Every dependent object has a `metadata.ownerReferences` field that
points to the owning object.
-->
いくつかの Kubernetes オブジェクトは他のオブジェクトの所有者です。
たとえば、ReplicaSet はポッドの集まりの所有者です。
所有しているオブジェクトは、所有者オブジェクトの *dependents（従属）* と呼びます。
それぞれの従属オブジェクト（dependent object）は `metadata.ownerReferences` （メタデータ.所有者参照）フィールドを持ちます。これは所有しているオブジェクトを示します。

<!--
Sometimes, Kubernetes sets the value of `ownerReference` automatically. For
example, when you create a ReplicaSet, Kubernetes automatically sets the
`ownerReference` field of each Pod in the ReplicaSet. In 1.8, Kubernetes
automatically sets the value of `ownerReference` for objects created or adopted
by ReplicationController, ReplicaSet, StatefulSet, DaemonSet, Deployment, Job
and CronJob.
-->
時々、Kubernetes は `ownerReference` （所有者参照）の値を自動的に設定します。
たとえば、ReplicaSet の作成時、Kubernetes は自動的に ReplicaSet 内の各ポッドの `ownerReference` フィールドを更新します。
Kubernetes 1.8 では、オブジェクトの作成時または採用時（adopted）に  `ownerReference` の値が ReplicationController、ReplicaSet、StatefulSet、Deployment、Job、CronJob によって自動的に設定されます。

<!--
You can also specify relationships between owners and dependents by manually
setting the `ownerReference` field.
-->
また、自分で所有者と従属に関する指定を、手動で `ownerReference` フィールドの更新もできます。

<!--
Here's a configuration file for a ReplicaSet that has three Pods:
-->
こちらの設定ファイルは３つのポッドを持つ ReplicaSet です。

{{< codenew file="controllers/replicaset.yaml" >}}

<!--
If you create the ReplicaSet and then view the Pod metadata, you can see
OwnerReferences field:
-->
もし ReplicaSet を作成し、ポッドのメタデータを参照すると、OwnerReferences フィールドが見えるでしょう。

```shell
kubectl create -f https://k8s.io/examples/controllers/replicaset.yaml
kubectl get pods --output=yaml
```

<!--
The output shows that the Pod owner is a ReplicaSet named `my-repset`:
-->
出力結果から、ポッドの所有者が `myrepset` という名称の ReplicaSet だと分かります。

```shell
apiVersion: v1
kind: Pod
metadata:
  ...
  ownerReferences:
  - apiVersion: apps/v1
    controller: true
    blockOwnerDeletion: true
    kind: ReplicaSet
    name: my-repset
    uid: d9607e19-f88f-11e6-a518-42010a800195
  ...
```

<!--
## Controlling how the garbage collector deletes dependents
--
## ガベージ・コレクタが削除する従属を制御するには {#controlling-how-the-garbage-collector-deletes-dependents}
<!--
When you delete an object, you can specify whether the object's dependents are
also deleted automatically. Deleting dependents automatically is called *cascading
deletion*.  There are two modes of *cascading deletion*: *background* and *foreground*.
-->
オブジェクトの削除時、オブジェクトの従属を限定して、自動的な削除を行えます。
従属の自動的な削除を *cascading deletion（連鎖削除／カスケーディング削除）* と呼びます。
*cascading deletion（連鎖削除）*には *background（バックグラウンド）* と *foreground （フォアグラウンド）* があります。

<!--
If you delete an object without deleting its dependents
automatically, the dependents are said to be *orphaned*.
-->
オブジェクトの削除時に従属を自動自動的に削除しなければ、従属は *orphaned（孤立化）*  すると呼ばれます。

<!--
### Foreground cascading deletion
-->
### フォアグラウンド連鎖削除（Foreground cascading deletion） {#forground-cascading-deletion}

<!--
In *foreground cascading deletion*, the root object first
enters a "deletion in progress" state. In the "deletion in progress" state,
the following things are true:
-->
*フォアグラウンド連鎖削除* において、元になるオブジェクトがまず先に "deletion in progress"（削除進行中） 状態となります。
"deletion progress"  状態においては、以下の状態が true（正）です：

<!--
 * The object is still visible via the REST API
 * The object's `deletionTimestamp` is set
 * The object's `metadata.finalizers` contains the value "foregroundDeletion".
-->
 * オブジェクトは REST API を経由して見えるまま
 * オブジェクトの `deletionTimestamp` がセットされる
 * オブジェクトの `metadata.finalizers` には "foregroundDeletion" の値が含まれる

<!--
Once the "deletion in progress" state is set, the garbage
collector deletes the object's dependents. Once the garbage collector has deleted all
"blocking" dependents (objects with `ownerReference.blockOwnerDeletion=true`), it delete
the owner object.
-->

一度 "deletion in progress" 状態がセットされると、ガベージ・コレクションはオブジェクトの従属を削除します。
ガベージ・コレクタが全ての "blocking" （ブロックされた）従属（ `ownerReference.blockOwnerDeletion=true` のオブジェクト）を削除すると、所有者オブジェクトを削除します。

<!--
Note that in the "foregroundDeletion", only dependents with
`ownerReference.blockOwnerDeletion` block the deletion of the owner object.
Kubernetes version 1.7 added an [admission controller](/docs/admin/admission-controllers/#ownerreferencespermissionenforcement) that controls user access to set
`blockOwnerDeletion` to true based on delete permissions on the owner object, so that
unauthorized dependents cannot delay deletion of an owner object.
-->
"foregroundDeletion" は `ownerReference.blockOwnerDeletion` ブロックを持つ従属のみが、所有者オブジェクトの削除対象となりますのでご注意ください。
Kubernetes バージョン 1.7 は [admission controller（承認コントローラ）](/jp/docs/admin/admission-controllers/#ownerreferencespermissionenforcement) を追加しました。
これは所有者オブジェクト上の削除権限を、`blockOwnerDeletion`  が true をベースとしたユーザアクセスを制御します。
そのため、所有者オブジェクトの削除をしても、その後の従属は削除が許可されません。

<!--
If an object's `ownerReferences` field is set by a controller (such as Deployment or ReplicaSet),
blockOwnerDeletion is set automatically and you do not need to manually modify this field.
-->
オブジェクトの `ownerReferences` フィールドはコントローラ（Deployment や ReplicaSet）によって設定されますが、
blockOwnerDeletion は自動的に設定されるもので、ユーザが手動でこのフィールドを変更する必要はありません。

<!--
### Background cascading deletion
-->
### バックグラウンド連鎖削除（Background cascading deletion） {#background-cascading-deletion}

<!--
In *background cascading deletion*, Kubernetes deletes the owner object
immediately and the garbage collector then deletes the dependents in
the background.
-->
*background cascading deletion（バックグラウンド連鎖削除）* では、Kubernetes は所有者オブジェクトを直ちに削除し、ガベージ・コレクタがバックグラウンドでその従属を削除します。

<!--
### Setting the cascading deletion policy
-->
### 連鎖削除ポリシーの設定 {#setting-the-cascading-deletion-policy}

<!--
To control the cascading deletion policy, set the `propagationPolicy`
field on the `deleteOptions` argument when deleting an Object. Possible values include "Orphan",
"Foreground", or "Background".
-->
連鎖削除ポリシーを設定するには、 `propagationPolicy` （伝搬方針）フィールドにある `deleteOptions`  （削除オプション）引数を、オブジェクトの削除時に設定します。
ここで設定できる値は "Orphan" （孤立）、 "" （フォアグラウンド）、 "" （バックグラウンド）です。

<!--
Prior to Kubernetes 1.9, the default garbage collection policy for many controller resources was `orphan`.
This included ReplicationController, ReplicaSet, StatefulSet, DaemonSet, and
Deployment. For kinds in the `extensions/v1beta1`, `apps/v1beta1`, and `apps/v1beta2` group versions, unless you 
specify otherwise, dependent objects are orphaned by default. In Kubernetes 1.9, for all kinds in the `apps/v1` 
group version, dependent objects are deleted by default.
-->
Kubernetes 1.9 よりも前のバージョンでは、多くのコントローラ・リソースにおけるデフォルトのガベージ・コレクション方針は `orphan` （孤立）でした。
これには ReplicationController、ReplicaSet、StatefulSet、DaemonSet、Deployment を含みます。
`extensions/v1beta1`、 `apps/v1beta1`、 `apps/v1beta2` グループ・バージョンでは、項目の指定がなければ、従属オブジェクトはデフォルトで孤立します。
Kubernetes 1.9 では、 `apps/v1` のグループ・バージョンすべてが、デフォルトで従属オブジェクトが削除されます。

<!--
Here's an example that deletes dependents in background:
-->
こちらはバックグラウンドで従属を削除する例です：

```shell
kubectl proxy --port=8080
curl -X DELETE localhost:8080/apis/apps/v1/namespaces/default/replicasets/my-repset \
-d '{"kind":"DeleteOptions","apiVersion":"v1","propagationPolicy":"Background"}' \
-H "Content-Type: application/json"
```

<!--
Here's an example that deletes dependents in foreground:
-->
こちらはフォアグラウンドで従属を削除する例です：

```shell
kubectl proxy --port=8080
curl -X DELETE localhost:8080/apis/apps/v1/namespaces/default/replicasets/my-repset \
-d '{"kind":"DeleteOptions","apiVersion":"v1","propagationPolicy":"Foreground"}' \
-H "Content-Type: application/json"
```

<!--
Here's an example that orphans dependents:
-->
こちらは孤立した従属の例です；

```shell
kubectl proxy --port=8080
curl -X DELETE localhost:8080/apis/apps/v1/namespaces/default/replicasets/my-repset \
-d '{"kind":"DeleteOptions","apiVersion":"v1","propagationPolicy":"Orphan"}' \
-H "Content-Type: application/json"
```

<!--
kubectl also supports cascading deletion.
To delete dependents automatically using kubectl, set `--cascade` to true.  To
orphan dependents, set `--cascade` to false. The default value for `--cascade`
is true.
-->
また、kubectl も連鎖削除をサポートします。
kubectl を使って従属を自動的に削除するには、 `--cascade` を true に設定します。
従属を孤立させるには、 `--cascade` を false にします。
`--cascade` のデフォルト値は true です。

<!--
Here's an example that orphans the dependents of a ReplicaSet:
-->
こちらは ReplicaSet の従属を孤立化する例です；

```shell
kubectl delete replicaset my-repset --cascade=false
```

<!--
### Additional note on Deployments
-->
### デプロイメント上における追加注記 {#additional-note-on-deployments}

<!--
When using cascading deletes with Deployments you *must* use `propagationPolicy: Foreground`
to delete not only the ReplicaSets created, but also their Pods. If this type of _propagationPolicy_
is not used, only the ReplicaSets will be deleted, and the Pods will be orphaned.
See [kubeadm/#149](https://github.com/kubernetes/kubeadm/issues/149#issuecomment-284766613) for more information.
-->
デプロイメントを連鎖削除するときは、 `propagationPolicy: Foreground` にするのが *必須* です。
作成した Replicaset の削除だけでなく、ポッドも削除する必要があります。
_propagationPolicy_ （伝搬方針）を使わなければ、ReplicaSet のみが削除され、ポッドは孤立化します。
詳しい情報は [kubeadm/#149](https://github.com/kubernetes/kubeadm/issues/149#issuecomment-284766613) をご覧ください。

<!--
## Known issues
-->
## 衆知の問題 {#known-issue}

<!--
Tracked at [#26120](https://github.com/kubernetes/kubernetes/issues/26120)
-->
[#26120](https://github.com/kubernetes/kubernetes/issues/26120) を追跡ください。

{{% /capture %}}


{{% capture whatsnext %}}

<!--
[Design Doc 1](https://git.k8s.io/community/contributors/design-proposals/api-machinery/garbage-collection.md)

[Design Doc 2](https://git.k8s.io/community/contributors/design-proposals/api-machinery/synchronous-garbage-collection.md)
-->
[設計文書1](https://git.k8s.io/community/contributors/design-proposals/api-machinery/garbage-collection.md)

[設計文章2](https://git.k8s.io/community/contributors/design-proposals/api-machinery/synchronous-garbage-collection.md)

{{% /capture %}}



