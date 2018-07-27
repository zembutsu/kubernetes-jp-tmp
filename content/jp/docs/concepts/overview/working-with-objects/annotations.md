---
title: アノテーション
content_template: templates/concept
weight: 50
---

{{% capture overview %}}
<!--
You can use Kubernetes annotations to attach arbitrary non-identifying metadata
to objects. Clients such as tools and libraries can retrieve this metadata.
-->
Kubernetes アノテーション（annocation：注釈）を使えば、オブジェクトに対して識別用途以外のメタデータを貼り付けられます。ツールやライブラリのようなクライアントがこのメタデータを参照できます。

{{% /capture %}}

{{% capture body %}}
<!--
## Attaching metadata to objects
-->
## メタデータをオブジェクトにアタッチ {#attaching-metadata-to-object}

<!--
You can use either labels or annotations to attach metadata to Kubernetes
objects. Labels can be used to select objects and to find
collections of objects that satisfy certain conditions. In contrast, annotations
are not used to identify and select objects. The metadata
in an annotation can be small or large, structured or unstructured, and can
include characters not permitted by labels.
-->
Kubernetes オブジェクトに対して添付（アタッチ）するメタデータには、ラベルだけでなくアノテーション（annotation）があります。ラベルを使う目的は、オブジェクトの選択と、条件を満たしているオブジェクトの集まりを見つけるためです。一方のアノテーションの場合は、オブジェクトの識別や選択のためには使いません。メタデータに含まれるアノテーションで使えるのは、小さいあるいはは大きなデータ、構造化あるいは非構造化データ、あるいはラベルでは許可されない文字列も含みます。

<!--
Annotations, like labels, are key/value maps:
-->
アノテーションはラベルのように、キー／バリューで割り当てます。

```json
"metadata": {
  "annotations": {
    "key1" : "value1",
    "key2" : "value2"
  }
}
```
<!--
Here are some examples of information that could be recorded in annotations:
-->
アノテーションとして記録可能な情報の例を紹介します：

<!--
* Fields managed by a declarative configuration layer. Attaching these fields
  as annotations distinguishes them from default values set by clients or
  servers, and from auto-generated fields and fields set by
  auto-sizing or auto-scaling systems.

* Build, release, or image information like timestamps, release IDs, git branch,
  PR numbers, image hashes, and registry address.

* Pointers to logging, monitoring, analytics, or audit repositories.

* Client library or tool information that can be used for debugging purposes:
  for example, name, version, and build information.

* User or tool/system provenance information, such as URLs of related objects
  from other ecosystem components.

* Lightweight rollout tool metadata: for example, config or checkpoints.

* Phone or pager numbers of persons responsible, or directory entries that
  specify where that information can be found, such as a team web site.
-->
* フィールドは宣言型の設定レイヤで管理します。クライアントやサーバによってセットされるデフォルトの値と、自動サイジングや自動スケール・システムによって生成されるフィールと識別するため、これらのフィールドでアノテーションを使う。

* ビルド、リリース、あるいはイメージにおけるタイムスタンプ、リリース ID 、git ブランチ、PR 番号、イメージ・ハッシュ値、レジストリの場所（アドレス）。

* ログ搬出（ログ記録）、監視、懐石、リポジトリ監査のためのポインタ。

* クライアント・ライブラリやツールがデバッグ用途で使う情報。たとえば、名前、バージョン、ビルド情報。

* ツールまたはシステムの起源（出所）となる情報。たとえば他のエコシステム構成要素のオブジェクトに関連する URL のようなもの。

* 軽量な展開（ロールアウト）ツールのメタデータ。例えば設定やチェックポイントなど。

* 責任者の電話番号やページャー（ポケベル）番号、あるいはチームのウェブサイトのような何らかの全体の情報を示す情報。

<!--
Instead of using annotations, you could store this type of information in an
external database or directory, but that would make it much harder to produce
shared client libraries and tools for deployment, management, introspection,
and the like.
-->
アノテーションを使わずに、外部のデータベースやディレクトリでの情報保管もできるでしょう。しかし、クライアント・ライブラリや展開用ツール、管理、内省がより大変になるかもしれません。

{{% /capture %}}

{{% capture whatsnext %}}
<!--
Learn more about [Labels and Selectors](/docs/concepts/overview/working-with-objects/labels/).
-->
[ラベルとセレクタ](/docs/concepts/overview/working-with-objects/labels/)について学びましょう。
{{% /capture %}}


