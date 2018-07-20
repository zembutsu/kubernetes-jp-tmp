---
reviewers:
- derekwaynecarr
- mikedanese
- thockin
title: 名前空間
content_template: templates/concept
weight: 30
---

{{% capture overview %}}

<!--
Kubernetes supports multiple virtual clusters backed by the same physical cluster.
These virtual clusters are called namespaces.
-->
Kubernetes は同じ物理クラスタ上で複数の仮想クラスタをサポートします。これらの仮想クラスタを名前空間（namespace）と呼びます。

{{% /capture %}}

{{< toc >}}

{{% capture body %}}
<!--
## When to Use Multiple Namespaces
-->
## 複数の名前空間を使う場合 {#when-to-user-multiple-namespaces}

<!--
Namespaces are intended for use in environments with many users spread across multiple
teams, or projects.  For clusters with a few to tens of users, you should not
need to create or think about namespaces at all.  Start using namespaces when you
need the features they provide.
-->
名前空間（namespaces）の利用を想定しているのは、多くの利用者が複数のチームやプロジェクトを広く横断している環境です。利用者が数人から10人のクラスタでは、名前空間の作成や検討する必要はほとんどないでしょう。名前空間が必要な機能が必要になった場合に、名前空間の利用を始めます。

<!--
Namespaces provide a scope for names.  Names of resources need to be unique within a namespace, but not across namespaces.
-->
名前空間が提供するのは名前の範囲（scope for namess）です。名前空間内ではリソース名がユニークである必要がありますが、名前空間は越えません。

<!--
Namespaces are a way to divide cluster resources between multiple users (via [resource quota](/docs/concepts/policy/resource-quotas/)).
-->
名前空間とは複数のユーザ間でクラスタのリソースを（[リソース・クォータ](/jp/docs/concepts/policy/resource-quotas/)を経由して）分割するための手段です。

<!--
In future versions of Kubernetes, objects in the same namespace will have the same
access control policies by default.
-->
Kubernetes の今後のバージョンでは、同じ名前空間にあるオブジェクトが、同じアクセス制御ポリシーを持つのがデフォルトになります。

<!--
It is not necessary to use multiple namespaces just to separate slightly different
resources, such as different versions of the same software: use [labels](/docs/user-guide/labels) to distinguish
resources within the same namespace.
-->
同じソフトウェアの異なるバージョンを使うなど、わずかに異なるリソースのために名前空間を使う必要はありません。そのような場合は同じ名前空間内でも [ラベル](/jp/docs/user-guide/labels) を使ってリソースを区別します。

<!--
## Working with Namespaces
-->
## 名前空間を活用 {#working-with-namespaces}

<!--
Creation and deletion of namespaces are described in the [Admin Guide documentation
for namespaces](/docs/admin/namespaces).
-->
名前空間の作成と定義については、[名前空間の管理者ガイド・ドキュメント](/jp/docs/admin/namespaces) に説明があります。

<!--
### Viewing namespaces
-->
### 名前空間を見てみる {#viewing-namespaces}

<!--
You can list the current namespaces in a cluster using:
-->
クラスタで使っている名前空間の一覧を表示できます：

```shell
$ kubectl get namespaces
NAME          STATUS    AGE
default       Active    1d
kube-system   Active    1d
kube-public   Active    1d
```

<!--
Kubernetes starts with three initial namespaces:
-->
Kubernetes は初期に３つの名前空間を準備します。

<!---
   * `default` The default namespace for objects with no other namespace
   * `kube-system` The namespace for objects created by the Kubernetes system
   * `kube-public` The namespace is created automatically and readable by all users (including those not authenticated). This namespace is mostly reserved for cluster usage, in case that some resources should be visible and readable publicly throughout the whole cluster. The public aspect of this namespace is only a convention, not a requirement.
-->
   * `default` オブジェクトが一切ないデフォルトの名前空間。
   * `kube-system` Kubernetes システムによって作成されるオブジェクト用の名前空間。
   * `kube-public` 自動的に作成され、すべてのユーザ（認証されていないユーザも含む）が読み込み可能な名前空間。この名前空間はクラスタが使うために予約されています。用途はクラスタ全体を通してパブリックに表示かつ読み込み可能な同一リソースを使う場合です。この名前空間がパブリックに見えるのは利便性のためであり、（利用は）必須ではありません。

<!--
### Setting the namespace for a request
-->
### リクエストのために名前空間を設定 {#setting-the-namespace-for-a-request}

<!--
To temporarily set the namespace for a request, use the `--namespace` flag.
--->
リクエストのために一時的な名前空間を設定するには、 `--namespace` フラグを使います。

<!--
For example:
-->
例：

```shell
$ kubectl --namespace=<insert-namespace-name-here> run nginx --image=nginx
$ kubectl --namespace=<insert-namespace-name-here> get pods
```

<!--
### Setting the namespace preference
-->
### 優先する名前空間の設定 {#setting-the-namespace-preference}

<!--
You can permanently save the namespace for all subsequent kubectl commands in that
context.
-->
kubectl コマンドに続くすべての名前空間を変わらないように保存するには、次のように実行します。

```shell
$ kubectl config set-context $(kubectl config current-context) --namespace=<insert-namespace-name-here>
# Validate it
$ kubectl config view | grep namespace:
```

<!--
## Namespaces and DNS
-->
## 名前空間と DNS {#namespaces-and-dns}

<!--
When you create a [Service](/docs/user-guide/services), it creates a corresponding [DNS entry](/docs/concepts/services-networking/dns-pod-service/).
This entry is of the form `<service-name>.<namespace-name>.svc.cluster.local`, which means
that if a container just uses `<service-name>`, it will resolve to the service which
is local to a namespace.  This is useful for using the same configuration across
multiple namespaces such as Development, Staging and Production.  If you want to reach
across namespaces, you need to use the fully qualified domain name (FQDN).
-->
[サービス](/jp/docs/user-guide/services) を作成すると、適切な [DNS エントリ](/jp/docs/concepts/services-networking/dns-pod-service/) が作成されます。このエントリは `<サービス名>.<名前空間名>.svc.cluster.local` の形式です。これが意味するのは、 `<サービス名>` にあたるのがコンテナで使われる箇所で、ローカルな名前空間におけるサービスの名前解決に使います。これは展開（デプロイメント）、ステージング、本番といった、複数の名前空間を通して同じ設定を使う場合に便利です。もしも名前空間を越えて到達したい場合は、完全修飾ドメイン名（FQDN）を使う必要があります。

<!--
## Not All Objects are in a Namespace
-->
## オブジェクトが名前空間にあるとは限らない {#not-all-obujects-are-in-a-namespace}

<!--
Most Kubernetes resources (e.g. pods, services, replication controllers, and others) are
in some namespaces.  However namespace resources are not themselves in a namespace.
And low-level resources, such as [nodes](/docs/admin/node) and
persistentVolumes, are not in any namespace.
-->
多くの Kubernetes リソース（例：ポッド、サービス、レプリケーション・コントローラ、等）は同じ名前空間内にあります。しかしながら、名前空間リソースそのものが名前空間にはありません。また、 [ノード](/docs/admin/node) や一貫性ボリュームといった低レベル・リソースは名前空間内にはありません。

{{% /capture %}}