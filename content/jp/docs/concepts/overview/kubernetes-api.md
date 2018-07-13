---
reviewers:
- chenopis
title: Kubernetes API
content_template: templates/concept
weight: 30
---

{{% capture overview %}}
<!--
Overall API conventions are described in the [API conventions doc](https://git.k8s.io/community/contributors/devel/api-conventions.md).
-->
すべての API 仕様の記述は [API 仕様ドキュメント](https://git.k8s.io/community/contributors/devel/api-conventions.md) にあります。

<!--
API endpoints, resource types and samples are described in [API Reference](/docs/reference).
-->
API エンドポイント、リソース・タイプ、サンプルは [API リファレンス](/docs/reference) に記述があります。

<!--
Remote access to the API is discussed in the [access doc](/docs/admin/accessing-the-api).
-->
API に対するリモート・アクセスは [access doc](/docs/admin/accessing-the-api) で議論されました。。

<!--
The Kubernetes API also serves as the foundation for the declarative configuration schema for the system. The [kubectl](/docs/reference/kubectl/overview/) command-line tool can be used to create, update, delete, and get API objects.
-->
Kubernetes API もまた、システムと同じ宣言型設定スキーム（declarative configuration schema）を提供します。[kubectl](/jp/docs/reference/kubectl/overview/) コマンドライン・ツールを API オブジェクトの作成、更新、削除、取得に使えます。

<!--
Kubernetes also stores its serialized state (currently in [etcd](https://coreos.com/docs/distributed-configuration/getting-started-with-etcd/)) in terms of the API resources.
-->
また、Kubernetes も逐次状態を保管するため（現時点では  [etcd](https://coreos.com/docs/distributed-configuration/getting-started-with-etcd/) ）に API リソースを使います。

<!--
Kubernetes itself is decomposed into multiple components, which interact through its API.
-->
複数の構成要素に分割された Kubernetes 自身も、API を通してやりとりします。


{{% /capture %}}

{{< toc >}}

{{% capture body %}}

<!--
## API changes
-->
## API 変更 {#api-changes}

<!--
In our experience, any system that is successful needs to grow and change as new use cases emerge or existing ones change. Therefore, we expect the Kubernetes API to continuously change and grow. However, we intend to not break compatibility with existing clients, for an extended period of time. In general, new API resources and new resource fields can be expected to be added frequently. Elimination of resources or fields will require following the [API deprecation policy](/docs/reference/using-api/deprecation-policy/).
-->
私たちの経験によれば、あらゆるシステムが成功するために必要なのは、成長と変化であり、新しい利用方法が出てくるか、既存の使い方が変わります。したがって、私たちは Kubernetes API が変化と成長を続けると予想しています。しかしながら、一定の期間は既存クライアントの互換性を壊すつもりはありません。一般に、新しい API リソースと新しいリソース・フィールドが定期的に追加されるのが予想されます。リソースやフィールドの削除には、 [API  機能廃止ポリシー（deprecation policy）](/jp/docs/reference/using-api/deprecation-policy/) に従う必要があります。

<!--
What constitutes a compatible change and how to change the API are detailed by the [API change document](https://git.k8s.io/community/contributors/devel/api_changes.md).
-->
互換性の変更と API の変更の仕方については、詳細が [API 変更ドキュメント](https://git.k8s.io/community/contributors/devel/api_changes.md) にあります。

<!--
## OpenAPI and Swagger definitions
-->
## OpenAPI と Swagger 定義 {#openapi-and-swagger-definitions}

<!--
Complete API details are documented using [Swagger v1.2](http://swagger.io/) and [OpenAPI](https://www.openapis.org/). The Kubernetes apiserver (aka "master") exposes an API that can be used to retrieve the Swagger v1.2 Kubernetes API spec located at `/swaggerapi`.
-->
API の完全な詳細 は [Swagger v1.2](http://swagger.io/) と [OpenAPI](https://www.openapis.org/) を使ってドキュメント化されています。Kubernetes api サーバ（別名 "マスタ"）が API を露出（expose）しています。そのため、Swagger v1.2 Kubernetes API 仕様は `/swaggerapi` にあるものを取得できます。

<!--
Starting with Kubernetes 1.10, OpenAPI spec is served in a single `/openapi/v2` endpoint. The format-separated endpoints (`/swagger.json`, `/swagger-2.0.0.json`, `/swagger-2.0.0.pb-v1`, `/swagger-2.0.0.pb-v1.gz`) are deprecated and will get removed in Kubernetes 1.14.
-->
Kubernetes 1.10 から `/openapi/v2` エンドポイントで OpenAPI 仕様の提供を始めました。仕様によって分けられているエンドポイント（`/swagger.json`、 `/swagger-2.0.0.json`、 `/swagger-2.0.0.pb-v1`、 `/swagger-2.0.0.pb-v1.gz`）は廃止予定であり、Kubernets 1.14 で削除されます。

<!--
Requested format is specified by setting HTTP headers:
-->
要求する書式（フォーマット）は HTTP ヘッダの設定で指定します。

<!--
Header | Possible Values
------ | ---------------
Accept | `application/json`, `application/com.github.proto-openapi.spec.v2@v1.0+protobuf` (the default content-type is `application/json` for `*/*` or not passing this header)
Accept-Encoding | `gzip` (not passing this header is acceptable)
-->
ヘッダ | 候補値
------ | ---------------
Accept | `application/json`, `application/com.github.proto-openapi.spec.v2@v1.0+protobuf` (the default content-type is `application/json` for `*/*` or not passing this header)
Accept-Encoding | `gzip` (not passing this header is acceptable)


<!--
**Examples of getting OpenAPI spec**:
-->
**OpenAPI 仕様で取得する例**：

<!--
Before 1.10 | Starting with Kubernetes 1.10
----------- | -----------------------------
GET /swagger.json | GET /openapi/v2 **Accept**: application/json
GET /swagger-2.0.0.pb-v1 | GET /openapi/v2 **Accept**: application/com.github.proto-openapi.spec.v2@v1.0+protobuf
GET /swagger-2.0.0.pb-v1.gz | GET /openapi/v2 **Accept**: application/com.github.proto-openapi.spec.v2@v1.0+protobuf **Accept-Encoding**: gzip
-->
1.10 以前 | Kubernetes 1.10 以降
----------- | -----------------------------
GET /swagger.json | GET /openapi/v2 **Accept**: application/json
GET /swagger-2.0.0.pb-v1 | GET /openapi/v2 **Accept**: application/com.github.proto-openapi.spec.v2@v1.0+protobuf
GET /swagger-2.0.0.pb-v1.gz | GET /openapi/v2 **Accept**: application/com.github.proto-openapi.spec.v2@v1.0+protobuf **Accept-Encoding**: gzip


<!--
Kubernetes implements an alternative Protobuf based serialization format for the API that is primarily intended for intra-cluster communication, documented in the [design proposal](https://github.com/kubernetes/community/blob/master/contributors/design-proposals/api-machinery/protobuf.md) and the IDL files for each schema are located in the Go packages that define the API objects.
-->

<!--
## API versioning
-->
## API バージョン指定 {#api-versioning}

<!--
To make it easier to eliminate fields or restructure resource representations, Kubernetes supports
multiple API versions, each at a different API path, such as `/api/v1` or
`/apis/extensions/v1beta1`.
-->
リソースを表すフィールドや構造を、簡単に削除できるようにするために、Kubernetes は複数の API バージョンをサポートします。それぞれ `/api/v1` や `/apis/extentions/v1beta1` のように API パスが異なります。

<!--
We chose to version at the API level rather than at the resource or field level to ensure that the API presents a clear, consistent view of system resources and behavior, and to enable controlling access to end-of-lifed and/or experimental APIs. The JSON and Protobuf serialization schemas follow the same guidelines for schema changes - all descriptions below cover both formats.
-->
私たちが選択したのは、リソースやフィールドレベルで指定するのではなく、 API レベルのバージョンで表す方法です。これはAPI の存在が明確であり、システム・リソースと挙動の見方が一貫し、さらに提供終了なり実験的な API に対するアクセスも制御可能だからです。JSON と Protobuf シリアル化スキームに従い、スキームの変更はガイドラインと同じです。以下で書式についても同様に説明します。

<!--
Note that API versioning and Software versioning are only indirectly related.  The [API and release
versioning proposal](https://git.k8s.io/community/contributors/design-proposals/release/versioning.md) describes the relationship between API versioning and
software versioning.
-->
API バージョンとソフトウェアのバージョンが関連しているのは、間接的のみなのでご注意ください。  [API and release versioning proposal](https://git.k8s.io/community/contributors/design-proposals/release/versioning.md) で API バージョンとソフトウェア・バージョン間の関係を説明しています。

<!--
Different API versions imply different levels of stability and support.  The criteria for each level are described
in more detail in the [API Changes documentation](https://git.k8s.io/community/contributors/devel/api_changes.md#alpha-beta-and-stable-versions).  They are summarized here:
-->
API バージョンがコト名rと、安定性とサポートのレベルが異なるのを示唆します。各レベルで重要となる基準については、 [API Changes documentation](https://git.k8s.io/community/contributors/devel/api_changes.md#alpha-beta-and-stable-versions) に詳細な記述があります。概要をまとめると：

<!--
- Alpha level:
  - The version names contain `alpha` (e.g. `v1alpha1`).
  - May be buggy.  Enabling the feature may expose bugs.  Disabled by default.
  - Support for feature may be dropped at any time without notice.
  - The API may change in incompatible ways in a later software release without notice.
  - Recommended for use only in short-lived testing clusters, due to increased risk of bugs and lack of long-term support.
- Beta level:
  - The version names contain `beta` (e.g. `v2beta3`).
  - Code is well tested.  Enabling the feature is considered safe.  Enabled by default.
  - Support for the overall feature will not be dropped, though details may change.
  - The schema and/or semantics of objects may change in incompatible ways in a subsequent beta or stable release.  When this happens,
    we will provide instructions for migrating to the next version.  This may require deleting, editing, and re-creating
    API objects.  The editing process may require some thought.   This may require downtime for applications that rely on the feature.
  - Recommended for only non-business-critical uses because of potential for incompatible changes in subsequent releases.  If you have
    multiple clusters which can be upgraded independently, you may be able to relax this restriction.
  - **Please do try our beta features and give feedback on them!  Once they exit beta, it may not be practical for us to make more changes.**
- Stable level:
  - The version name is `vX` where `X` is an integer.
  - Stable versions of features will appear in released software for many subsequent versions.
-->
- アルファ・レベル：
  - バージョン名に `alpha` を含める（例： `v1alpha1` ）
  - バグがある可能性。機能を有効化するとバグが顕わになる可能性。デフォルトでは無効化。
  - 機能に対するサポートは常に無く、警告も無し。
  - 予告なく API は最新のソフトウェア・リリースとの互換性が無くなる可能性。
  - 短期間のテスト用クラスタのみでの利用を推奨。バグによる危険性が増加し、長期間のサポートが困難。
- ベータ・レベル：
  - バージョン名に `beta` を含める（例： `v2beta3` ）
  - コードはしっかりテスト済み。機能の有効化は安全と考えられる。デフォルトで有効化。
  - 機能に対するサポートは常に無く、細部で変更の可能性。
  - オブジェクトの schema と semantics の片方または両方が、ベータから安定版に移行する段階で互換性をに変更を伴う可能性。このような場合、次のバージョンに移行する手順を提供。これにより、API オブジェクトの削除、編集、再作成が必要になる場合がある。プロセスの編集には考慮点が必要になる場合がある。機能によってはアプリケーションの停止時間が発生する場合がある。
  - ビジネスでクリティカルではない利用者だけ利用を推奨。時期リリースに互換性の無い変更を伴う可能性があるため。個々にアップグレードできる複数のクラスタを準備できれば、このような縛りがあってもリラックスできる。
  - **ベータ機能を試してもフィードバックを送らないでください！　なぜならまだベータを抜けていないからです。多くの変更があるかもしれないので、実用的では無いでしょう。**
- 安定版（stable）レベル：
  - バージョン名に `vX` を含み、 `X` は整数。
  - 機能の安定バージョンが、ソフトウェアのリリースで継続して搭載。

<!--
## API groups
-->
## API グループ {#api-groups}

<!--
To make it easier to extend the Kubernetes API, we implemented [*API groups*](https://git.k8s.io/community/contributors/design-proposals/api-machinery/api-group.md).
The API group is specified in a REST path and in the `apiVersion` field of a serialized object.
-->
Kubernetes API を簡単に拡張できるようにするため、 [*API グループ*](https://git.k8s.io/community/contributors/design-proposals/api-machinery/api-group.md)を実装しました。API グループは REST のパスと `apiVersion` フィールドでオブジェクトをシリアライズ化（連番を通して扱えるように）します。

<!--
Currently there are several API groups in use:
-->
API グループには現在複数の使い方があります：

<!--
1. The *core* group, often referred to as the *legacy group*, is at the REST path `/api/v1` and uses `apiVersion: v1`.

1. The named groups are at REST path `/apis/$GROUP_NAME/$VERSION`, and use `apiVersion: $GROUP_NAME/$VERSION`
   (e.g. `apiVersion: batch/v1`).  Full list of supported API groups can be seen in [Kubernetes API reference](/docs/reference/).
-->
1. コア・グループは、ときどき *レガシー・グループ* として参照されます。REST パスは `/api/v1` と `apiVersion: v1` を使います。
1. 名前付きグループの REST パスは `/apis/$GROUP_NAME/$VERSION` と`apiVersion: $GROUP_NAME/$VERSION` を使います（例：`apiVersion: batch/v1`）。サポートしている API グループすべての一覧は [Kubernetes API リファレンス](/jp/docs/reference/) にあります。

<!--
There are two supported paths to extending the API with [custom resources](/docs/concepts/api-extension/custom-resources/):
-->
 [custom resources](/docs/concepts/api-extension/custom-resources/) で API を拡張するには、２つのサポートされているパスがあります。

<!--
1. [CustomResourceDefinition](/docs/tasks/access-kubernetes-api/extend-api-custom-resource-definitions/)
   is for users with very basic CRUD needs.
1. Coming soon: users needing the full set of Kubernetes API semantics can implement their own apiserver
   and use the [aggregator](https://git.k8s.io/community/contributors/design-proposals/api-machinery/aggregated-api-servers.md)
   to make it seamless for clients.
-->
1. [CustomResourceDefinition](/docs/tasks/access-kubernetes-api/extend-api-custom-resource-definitions/)
   は非常に基本的な CRUD が必要なユーザ向けです。
1. 間もなく登場：Kubernetes API semantics のすべてを必要とするユーザが、自身の API サーバを実装し、クライアントとシームレスにするために [aggregator](https://git.k8s.io/community/contributors/design-proposals/api-machinery/aggregated-api-servers.md) を使う。

<!--
## Enabling API groups
-->
## API グループの有効化 {#enabling-api-groups}

<!--
Certain resources and API groups are enabled by default.  They can be enabled or disabled by setting `--runtime-config`
on apiserver. `--runtime-config` accepts comma separated values. For ex: to disable batch/v1, set
`--runtime-config=batch/v1=false`, to enable batch/v2alpha1, set `--runtime-config=batch/v2alpha1`.
The flag accepts comma separated set of key=value pairs describing runtime configuration of the apiserver.
-->
あるリソースと API グループがデフォルトで有効です。これらは api サーバの  `--runtime-config` の指定で有効化または無効化できます。`--runtime-config` はカンマで値を分けられます。例えば、 batch/v1 を無効にするには、 `--runtime-config=batch/v1=false` を指定します。batch/v2alpha1 を有効にするには、 `--runtime-config=batch/v2alpha1` を指定します。フラグはカンマで分けられた key=value の組み合わせて api サーバの実行時の設定でも記述できます。

<!--
IMPORTANT: Enabling or disabling groups or resources requires restarting apiserver and controller-manager
to pick up the `--runtime-config` changes.
-->
重要：グループ・リソースの有効化・無効化で `--runtime-config` を変更するには、  api サーバと controller-manager の再起動が必要です。

<!--
## Enabling resources in the groups
-->
## グループ内でリソースの有効化 {#enabling-resources-in-the-groups}

<!--
DaemonSets, Deployments, HorizontalPodAutoscalers, Ingress, Jobs and ReplicaSets are enabled by default.
Other extensions resources can be enabled by setting `--runtime-config` on
apiserver. `--runtime-config` accepts comma separated values. For example: to disable deployments and ingress, set
`--runtime-config=extensions/v1beta1/deployments=false,extensions/v1beta1/ingress=false`
-->
デフォルトで有効化されているのは、DaemonSets、 Deployments、 HorizontalPodAutoscalers、 Ingress、 Jobs、 ReplicaSets です。他の拡張リソースを有効にするには、api サーバで `--runtime-config` を設定します。 `--runtime-config` はカンマで分けた値を指定できます。例えば、 deployment と ingress を無効にするには、 `--runtime-config=extensions/v1beta1/deployments=false,extensions/v1beta1/ingress=false` を設定します。

{{% /capture %}}