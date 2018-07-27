---
title: Kubernetes オブジェクト管理
content_template: templates/concept
weight: 10
---

{{% capture overview %}}
<!--
The `kubectl` command-line tool supports several different ways to create and manage
Kubernetes objects. This document provides an overview of the different
approaches.
-->
`kubectl` コマンドライン・ツールは Kubernetes オブジェクトを作成・管理する複数の方法をサポートします。このドキュメントは異なる手法の概要を説明します。
{{% /capture %}}

{{% capture body %}}
<!--
## Management techniques
-->
## 管理手法（マネジメント・テクニック） {#management-techniques}

{{< warning >}}
<!--
**Warning:** A Kubernetes object should be managed using only one technique. Mixing
and matching techniques for the same object results in undefined behavior.
-->
**警告:** Kubernetes オブジェクトは１つの手法だけを使って管理すべきです。同じオブジェクトに対する手法の混合や組み合わせは、定義されていない挙動を引き起こします。
{{< /warning >}}

<!--
| Management technique             | Operates on          |Recommended environment | Supported writers  | Learning curve |
|----------------------------------|----------------------|------------------------|--------------------|----------------|
| Imperative commands              | Live objects         | Development projects   | 1+                 | Lowest         |
| Imperative object configuration  | Individual files     | Production projects    | 1                  | Moderate       |
| Declarative object configuration | Directories of files | Production projects    | 1+                 | Highest        |
-->
| 管理手法             | 影響・効果          |推奨環境 | サポートしている書き手  | 学習曲線 |
|----------------------------------|----------------------|------------------------|--------------------|----------------|
| コマンド命令型（Imperative commands）              | ライブ・オブジェクト         | 開発プロジェクト   | 1+                 | 低         |
| 命令型オブジェクト設定（Imperative object configuration）  | 個々のファイル     | 本番プロジェクト    | 1                  | 中       |
| 宣言型オブジェクト設定（Declarative object configuration） | ファイルのディレクトリ of files | 本番プロジェクト    | 1+                 | 高い        |

<!--
## Imperative commands
-->
## コマンド命令型 {#imperative-commands}

<!--
When using imperative commands, a user operates directly on live objects
in a cluster. The user provides operations to
the `kubectl` command as arguments or flags.
-->
コマンド命令型の利用時、ユーザはクラスタ内のライブ・オブジェクトを直接操作します。ユーザは作業にあたって `kubectl` コマンドを使い、引数やフラグを与えます。

<!--
This is the simplest way to get started or to run a one-off task in
a cluster. Because this technique operates directly on live
objects, it provides no history of previous configurations.
-->
こちらは使い始めや１度だけのタスクをクラスタでこなすには最も簡単な手法です。この手法による操作は生のオブジェクトを直接操作するため、以前の設定に関する履歴が残りません。

<!--
### Examples
-->
### 例
<!--
Run an instance of the nginx container by creating a Deployment object:
nginx コンテナのインスタンスを実行し、デプロイメント・オブジェクトを作成：
-->

```sh
kubectl run nginx --image nginx
```

<!--
Do the same thing using a different syntax:
-->
同じことを別の構文で実行：

```sh
kubectl create deployment nginx --image nginx
```

<!--
### Trade-offs
-->
### トレードオフ（得失評価）
<!--
Advantages compared to object configuration:
-->
オブジェクト設定に対する利点を比較：

<!--
- Commands are simple, easy to learn and easy to remember.
- Commands require only a single step to make changes to the cluster.
-->
- コマンドはシンプルで、学んだり覚えたりするのが楽。
- コマンドでクラスタの変更に必要なのは１ステップのみ。

<!--
Disadvantages compared to object configuration:
-->
オブジェクト設定に対する欠点を比較：

<!--
- Commands do not integrate with change review processes.
- Commands do not provide an audit trail associated with changes.
- Commands do not provide a source of records except for what is live.
- Commands do not provide a template for creating new objects.
-->
- コマンドは変更をレビューする手順が一緒ではない。
- コマンドは変更に関連する権限を追跡する機能を提供しない。
- コマンドは稼働中のもの以外の記録を提供しない。
- コマンドは新しいオブジェクトを作成するテンプレートを提供しない。

<!--
## Imperative object configuration
-->
## 命令型オブジェクト設定 {#imprerative-object-configuration}

<!--
In imperative object configuration, the kubectl command specifies the
operation (create, replace, etc.), optional flags and at least one file
name. The file specified must contain a full definition of the object
in YAML or JSON format.
-->
命令型オブジェクト設定で、kubectl コマンドで指定するのは、手順（作成、置き換え、等）、オプションのフラグ、そして少なくとも１つのファイル名です。ファイルで指定するのは、オブジェクトの完全な定義を含むものであり、YAML または JSON 形式です。

<!--
See the [API reference](/docs/reference/generated/kubernetes-api/{{< param "version" >}}/)
for more details on object definitions.
-->
オブジェクトの定義に関する詳しい情報は [API リファレンス](/jp/docs/reference/generated/kubernetes-api/{{< param "version" >}}/) をご覧ください。

{{< warning >}}
<!--
**Warning:** The imperative `replace` command replaces the existing
spec with the newly provided one, dropping all changes to the object missing from
the configuration file.  This approach should not be used with resource
types whose specs are updated independently of the configuration file.
Services of type `LoadBalancer`, for example, have their `externalIPs` field updated
independently from the configuration by the cluster.
-->
**警告**： 命令型 `replace` コマンドは直近で提供したものを置き換えるものであり、設定ファイルでオブジェクトに関する記述がなければ、すべてを削除します。この手法は設定ファイル上で spec（仕様）を個々に更新するリソース型によっては適さないでしょう。 たとえば `LoadBlancer` 型のサービスが持つ `externalIPs` フィールドの更新は、クラスタごとの設定ファイルによって個々に更新されます。
{{< /warning >}}

<!--
### Examples
-->
### 例

<!--
Create the objects defined in a configuration file:
-->
設定ファイルで定義したオブジェクトを作成：

```sh
kubectl create -f nginx.yaml
```

<!--
Delete the objects defined in two configuration files:
-->
２つの設定ファイルで定義したオブジェクトを削除：

```sh
kubectl delete -f nginx.yaml -f redis.yaml
```

<!--
Update the objects defined in a configuration file by overwriting
the live configuration:
-->
稼働中の設定を上書きし、設定ファイルで定義したオブジェクトに更新：

```sh
kubectl replace -f nginx.yaml
```

<!--
### Trade-offs
-->
### トレードオフ（得失評価）

<!--
Advantages compared to imperative commands:
-->
命令型コマンドに対する利点を比較：

<!--
- Object configuration can be stored in a source control system such as Git.
- Object configuration can integrate with processes such as reviewing changes before push and audit trails.
- Object configuration provides a template for creating new objects.
-->
- GIt のようなソース・コントロール・システムにオブジェクト設定を保管できる。
- push や audito trails による変更前のレビューなどのような手順とオブジェクト設定を統合できる。
- オブジェクト設定は新しいオブジェクトを作成するテンプレートになる。

<!--
Disadvantages compared to imperative commands:
-->
命令型コマンドに対する欠点を比較：

<!--
- Object configuration requires basic understanding of the object schema.
- Object configuration requires the additional step of writing a YAML file.
-->
- オブジェクト設定には、オブジェクト構造（スキーマ）の基本的な理解が必要。
- オブジェクト設定には YAML ファイルを書くための追加手順が必要。

<!--
Advantages compared to declarative object configuration:
-->
宣言型オブジェクト設定に対する利点を比較：

<!--
- Imperative object configuration behavior is simpler and easier to understand.
- As of Kubernetes version 1.5, imperative object configuration is more mature.
-->
- 命令型オブジェクト設定の挙動は簡単で、理解が簡単。
- Kubernetes バージョン 1.5 以降から、命令型オブジェクト設定が更に成熟。

<!--
Disadvantages compared to declarative object configuration:
-->
宣言型オブジェクト設定に対する欠点を比較：

<!--
- Imperative object configuration works best on files, not directories.
- Updates to live objects must be reflected in configuration files, or they will be lost during the next replacement.
-->
 - 命令型設定オブジェクト設定はファイルでの扱いが優れており、ディレクトリではない。
 - 存在しているオブジェクトの更新にには、設定ファイルに変更の反映が必須であり、書き換え後は消えてしまう。

<!--
## Declarative object configuration
-->
## 宣言型オブジェクト設定 {#declarative-object-configuration}

<!--
When using declarative object configuration, a user operates on object
configuration files stored locally, however the user does not define the
operations to be taken on the files. Create, update, and delete operations
are automatically detected per-object by `kubectl`. This enables working on
directories, where different operations might be needed for different objects.
-->
宣言型オブジェクト設定の使用時は、ローカルに保存したオブジェクト設定用のファイルを使ってユーザは操作します。しかしながら、ユーザが操作する場合は、設定用のファイルを定義する必要がありません。作成、更新、削除作業は `kubectl` を使ってオブジェクトごとに自動的に処理されます。様々なオブジェクトに対して別々の操作をする必要があるため、これらはの作業はディレクトリ内で行います。

{{< note >}}
<!--
**Note:** Declarative object configuration retains changes made by other
writers, even if the changes are not merged back to the object configuration file.
This is possible by using the `patch` API operation to write only
observed differences, instead of using the `replace`
API operation to replace the entire object configuration.
-->
**メモ：** あるオブジェクト設定ファイルに影響のある設定が他の書き手によって追加されても、宣言型オブジェクト設定では、それが元のファイルに反映（マージ）されなくても、設定変更は保持され続けます。これにより、オブジェクト設定の全体を置き換えるために `replace` API を使うのではなく、影響のある範囲を見えるように設定ファイルに書き出し、  `patch` API 処理をするのも可能です。
{{< /note >}}

<!--
### Examples
-->
### 例

<!---
Process all object configuration files in the `configs` directory, and
create or patch the live objects:
-->
すべてのオブジェクト設定ファイルが `config` ディレクトリに置き、ライブ・オブジェクトを作成または追加（パッチ）します。

```sh
kubectl apply -f configs/
```

<!--
Recursively process directories:
-->
処理用のディレクトリで繰り返します：

```sh
kubectl apply -R -f configs/
```

<!--
### Trade-offs
-->
### トレードオフ（損失評価）

<!--
Advantages compared to imperative object configuration:
-->
命令型オブジェクト設定に対する利点：

<!--
- Changes made directly to live objects are retained, even if they are not merged back into the configuration files.
- Declarative object configuration has better support for operating on directories and automatically detecting operation types (create, patch, delete) per-object.
-->
- 設定ファイルに対する変更が反映されていなくても、ライブ・オブジェクトに対する変更は維持されます。
- 宣言型オブジェクト設定は、ディレクトリでの操作に適しており、オブジェクト毎に操作タイプ（作成、パッチ、削除）を自動的に検出します。

<!--
Disadvantages compared to imperative object configuration:
-->
命令型オブジェクト設定に対する欠点：

<!--
- Declarative object configuration is harder to debug and understand results when they are unexpected.
- Partial updates using diffs create complex merge and patch operations.
-->
- 宣言型オブジェクト設定は予期しない状況でのデバッグや理解が大変。
- 部分的な更新のために差分作成を使うのは、マージやパッチ操作が複雑。


{{% /capture %}}

{{% capture whatsnext %}}
- [命令型コマンドを使う Kubernetes オブジェクト管理](/jp/docs/concepts/overview/object-management-kubectl/imperative-command/)
- [オブジェクト設定を使う (命令型) Kubernetes オブジェクト管理](/jp/docs/concepts/overview/object-management-kubectl/imperative-config/)
- [オブジェクト設定 を使う(宣言型) Kubernetes オブジェクト管理](/jp/docs/concepts/overview/object-management-kubectl/declarative-config/)
- [Kubectl コマンド・リファレンス（参考資料）](/jp/docs/reference/generated/kubectl/kubectl-commands/)
- [Kubernetes API リファレンス（参考資料）](/docs/reference/generated/kubernetes-api/{{< param "version" >}}/)

{{< comment >}}
{{< /comment >}}
{{% /capture %}}


