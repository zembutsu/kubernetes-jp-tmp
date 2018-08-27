---
title: kubeconfig ファイルを使うクラスタ接続
content_template: templates/concept
weight: 60
---

{{% capture overview %}}
<!--
Use kubeconfig files to organize information about clusters, users, namespaces, and
authentication mechanisms. The `kubectl` command-line tool uses kubeconfig files to
find the information it needs to choose a cluster and communicate with the API server
of a cluster.
-->
クラスタ、ユーザ、名前空間、認証機構に関する情報を kubeconfig ファイルにまとめられます。
`kubectl` コマンドライン・ツールは kubeconfig ファイルを使って、クラスタの選択やクラスタの API サーバとの通信に必要な情報を見つけます。

{{< note >}}
<!--
**Note:** A file that is used to configure access to clusters is called
a *kubeconfig file*. This is a generic way of referring to configuration files.
It does not mean that there is a file named `kubeconfig`.
-->
**メモ：** クラスタに接続する設定情報のために使うファイルのことを *kubeconfig ファイル*  と呼びます。
これは設定情報ファイルを参照するための、一般的な手法です。
`kubeconfig` という名前のファイルがあるわけではありません。
{{< /note >}}

<!--
By default, `kubectl` looks for a file named `config` in the `$HOME/.kube` directory.
You can specify other kubeconfig files by setting the `KUBECONFIG` environment
variable or by setting the
[`--kubeconfig`](/docs/reference/generated/kubectl/kubectl/) flag.
-->
デフォルトでは `kubectl` は `$HOME/.kube` ディレクトリにある `config` という名前のファイルを探します。
`KUBECONFIG` 環境変数か [`--kubeconfig`](/jp/docs/reference/generated/kubectl/kubectl/) フラグを指定して、他の kubeconfig ファイルを指定できます。

<!--
For step-by-step instructions on creating and specifying kubeconfig files, see
[Configure Access to Multiple Clusters](/docs/tasks/access-application-cluster/configure-access-multiple-clusters).
-->
kubeconfig ファイルを作成して指定するステップ・バイ・ステップの手順は、[Configure Access to Multiple Clusters](/jp/docs/tasks/access-application-cluster/configure-access-multiple-clusters) をご覧ください。

{{% /capture %}}


{{% capture body %}}

<!--
## Supporting multiple clusters, users, and authentication mechanisms
-->
## 複数のクラスタ、ユーザ、認証機構のサポート {#supporting-multiple-clusters-users-and-authentication-mechanisms}

<!--
Suppose you have several clusters, and your users and components authenticate
in a variety of ways. For example:
-->
複数のクラスタを持ち、複数のユーザとコンポーネントの認証方法があると仮定します。たとえば：

<!--
- A running kubelet might authenticate using certificates.
- A user might authenticate using tokens.
- Administrators might have sets of certificates that they provide to individual users.
-->
- kubelet は証明書を使って実行している可能性がある
- ユーザはトークンを使った認証を行う可能性がある
- 管理者は証明書のセットを使いって個々のユーザを識別する

<!--
With kubeconfig files, you can organize your clusters, users, and namespaces.
You can also define contexts to quickly and easily switch between
clusters and namespaces.
-->
kubeconfig ファイルでは、クラスタとユーザと名前空間をまとめられます。
また、コンテキストを定義し、複数のクラスタと名前空間を素早くかつ簡単に切り替えられます。

<!--
## Context
-->
## コンテキスト（context） {#context]

<!--
A *context* element in a kubeconfig file is used to group access parameters
under a convenient name. Each context has three parameters: cluster, namespace, and user.
By default, the `kubectl` command-line tool uses parameters from
the *current context* to communicate with the cluster. 
-->
kubeconfig ファイルの *コンテキスト（context）* 要素は、便利な名前を使ってアクセス・パラメータのグループを指定するのに使います。
各コンテキストには３つのパラメータがあります。クラスタ、名前空間、ユーザです。
デフォルトでは、 `kubectl` コマンドライン・ツールは *現在のコンテキスト（current context）* として、クラスタとの通信にパラメータを使います。

<!--
To choose the current context:
-->
現在のコンテキストを選択するには：
```
kubectl config use-context
```

<!--
## The KUBECONFIG environment variable
-->
## KUBECONFIG 環境変数 {#the-kubeonfig-environment-variable}

<!--
The `KUBECONFIG` environment variable holds a list of kubeconfig files.
For Linux and Mac, the list is colon-delimited. For Windows, the list
is semicolon-delimited. The `KUBECONFIG` environment variable is not
required. If the `KUBECONFIG` environment variable doesn't exist,
`kubectl` uses the default kubeconfig file, `$HOME/.kube/config`.
-->
`KUBECONFIG` 環境変数は kubeconfig ファイル群の一覧を保持します。
Linux と Mac では、この一覧はコロン区切りです。
Windows では、この一覧はセミコロン区切りです。
`KUBECONFIG` 環境変数は必須ではありません。
もしも `KUBECONFIG` 環境変数が存在しなければ、 `kubectl` はデフォルトの kubeconfig ファイルとして `$HOME/.kube/config` を使います。

<!--
If the `KUBECONFIG` environment variable does exist, `kubectl` uses
an effective configuration that is the result of merging the files
listed in the `KUBECONFIG` environment variable.
-->
`KUBECONFIG` 環境変数が存在しなければ、 `kubectl` は、設定ファイルから得られた有効な情報を統合し、 `KUBECONFIG` 間児湯変数に入れます。


<!--
## Merging kubeconfig files
-->
## kubeconfig ファイルの統合 {#merging-kubeconfig-files}

<!--
To see your configuration, enter this command:
-->
設定情報を参照するには、こちらのコマンドを実行します：

```shell
kubectl config view
```

<!--
As described previously, the output might be from a single kubeconfig file,
or it might be the result of merging several kubeconfig files.
-->
先述の通り、出力には１つの kubeconfig ファイルの場合もあれば、複数の kubeconfig ファイルを統合した結果になります。

<!--
Here are the rules that `kubectl` uses when it merges kubeconfig files:
-->
以下は `kubectl` が kubeconfig ファイルを統合するときの規則です。

<!--
1. If the `--kubeconfig` flag is set, use only the specified file. Do not merge.
   Only one instance of this flag is allowed.
-->
1. もしも `--kubeconfig` フラグの指定があれば、指定されたファイルのみを使います。統合は行いません。このフラグで指定したものだけを許可します。
<!--
   Otherwise, if the `KUBECONFIG` environment variable is set, use it as a
   list of files that should be merged.
   Merge the files listed in the `KUBECONFIG` environment variable
   according to these rules:
-->
あるいは、もしも `KUBECONFIG` 環境半数が指定され、ファイルの一覧があれば、統合して使われます。
`KUBECONFIG` 環境変数にあるファイル一覧の統合は、以下の規則に従います。

<!--
   * Ignore empty filenames.
   * Produce errors for files with content that cannot be deserialized.
   * The first file to set a particular value or map key wins.
   * Never change the value or map key.
     Example: Preserve the context of the first file to set `current-context`.
     Example: If two files specify a `red-user`, use only values from the first file's `red-user`.
     Even if the second file has non-conflicting entries under `red-user`, discard them.
-->
  * 空のファイル名を無視
  * コンテナントを再び直線上に並べられなければ、ファイルにエラーを出力
  * 詳細な値やキーを割り当てる（マップする）ために、１つめのファイルを使う
  * 値やキーの割り当て（マップ）にあたって変更をしない。
  例：コンテキストの１行目を `current-context` に保存
  例：２つのファイルで `red-user` を指定していると、１つめのファイルの `red-user` にある値のみ使う
  たとえ２つめのファイルが `red-user` に関する競合しない情報を含んでいたとしても、これらを無視する。

<!--
   For an example of setting the `KUBECONFIG` environment variable, see
   [Setting the KUBECONFIG environment variable](/docs/tasks/access-application-cluster/configure-access-multiple-clusters/#set-the-kubeconfig-environment-variable).
-->
`KUBECONFIG` 環境変数の設定例については [KUBECONFIG 環境変数の設定](/jp/docs/tasks/access-application-cluster/configure-access-multiple-clusters/#set-the-kubeconfig-environment-variable) をご覧ください。

<!--
   Otherwise, use the default kubeconfig file, `$HOME/.kube/config`, with no merging.
-->
あるいは、統合されていない `$HOME/.kube/config` にあるデフォルトの設定情報ファイルを使います。

<!_-
1. Determine the context to use based on the first hit in this chain:
-->
2. 鎖（チェーン）で初めて一致するコンテキストに決定する。
<!--
    1. Use the `--context` command-line flag if it exits.
    1. Use the `current-context` from the merged kubeconfig files.
-->
    1. `--context` コマンドライン・フラグがあれば、そちらを使う
    2. 統合された kubeconfig ファイルにある `current-context` を使う
<!--
   An empty context is allowed at this point.
-->
この時点では、空のコンテキストは許可されません。

<!--
1. Determine the cluster and user. At this point, there might or might not be a context.
   Determine the cluster and user based on the first hit in this chain,
   which is run twice: once for user and once for cluster:
-->
3. クラスタとユーザを決定します。この時点では、コンテキストかもしれませんし、そうでないかもしれません。クラスタとユーザを決めるにあたって、ベースとなるのは鎖（チェーン）中ではじめて一致するものが２つあります。１つはユーザに対して、１つはクラスタに対してです。
<!--
   1. Use a command-line flag if it exists: `--user` or `--cluster`.
   1. If the context is non-empty, take the user or cluster from the context.
-->
    1. コマンドライン・フラグに `--user` や `--cluster` があれば、そちらを使う
    2. コンテキストが空でなければ、コンテキストの中からユーザとクラスタを得る
<!--
   The user and cluster can be empty at this point.
-->
この時点では、ユーザとクラスタの指定がないのは許可されません。

<!--
1. Determine the actual cluster information to use. At this point, there might or
   might not be cluster information.
   Build each piece of the cluster information based on this chain; the first hit wins:
-->
4. 実際のクラスタ情報を決定するのに使う。この時点では、クラスタの情報があるかもしれませんし、無いかもしれません。鎖（チェイン）にあるクラスタ情報から、1番目に一致するものに基づいて構築します。
<!--
   1. Use command line flags if they exist: `--server`, `--certificate-authority`, `--insecure-skip-tls-verify`.
   1. If any cluster information attributes exist from the merged kubeconfig files, use them.
   1. If there is no server location, fail.
-->
    1. コマンドライン・フラグに `--server`、 `--certificate-authority`、 `--insecure-skip-tls-verify` があれば、そちらを使う
    2. 統合された kubeconfig ファイルにクラスタ情報に関する属性があれば、そちらを使う
    3. サーバの場所に関する情報がなければ、処理を中断

<!--
1. Determine the actual user information to use. Build user information using the same
   rules as cluster information, except allow only one authentication
   technique per user:
-->
5. 実際に使うユーザ情報を決定します。ユーザ情報に使うのはクラスタ情報の規則と同じですが、ユーザごとに１つの認証方法しか許可しません。
<!--
   1. Use command line flags if they exist: `--client-certificate`, `--client-key`, `--username`, `--password`, `--token`.
   1. Use the `user` fields from the merged kubeconfig files.
   1. If there are two conflicting techniques, fail.
-->
   1. コマンドライン・フラグに  `--client-certificate`、 `--client-key`、 `--username`、 `--password`、 `--token`があれば、そちらを使う
   2. 統合された kubeconfig ファイルにある `user` フィールドを使う
   3. ２つの認証手法があれば、処理を中断

<!--
1. For any information still missing, use default values and potentially
   prompt for authentication information.
-->
6. 認証に関する情報が足りなければ、デフォルトの値を使い、場合によっては認証情報の入力を促すプロンプトを表示。

<!--
## File references
-->
## ファイルの参照先 {#file-references}

<!--
File and path references in a kubeconfig file are relative to the location of the kubeconfig file.
File references on the command line are relative to the current working directory.
In `$HOME/.kube/config`, relative paths are stored relatively, and absolute paths
are stored absolutely.
-->
kubeconfig ファイルのファイルとパスを参照するにあたり、kubeconfig ファイルのある場所を相対的な基準とします。
コマンドライン上でファイルを参照するにあたっては、現在の作業ディレクトリを基準とします。
`$HOME/.kube/config` で相対パスがあれば、ファイルのあるディレクトリからですし、絶対パスのものは絶対パスとして扱います。

{{% /capture %}}


{{% capture whatsnext %}}

<!--
* [Configure Access to Multiple Clusters](/docs/tasks/access-application-cluster/configure-access-multiple-clusters/)
* [`kubectl config`](/docs/reference/generated/kubectl/kubectl-commands#config)
-->
* [複数のクラスタに対する接続を設定](/jp/docs/tasks/access-application-cluster/configure-access-multiple-clusters/)
* [`kubectl config`](/jp/docs/reference/generated/kubectl/kubectl-commands#config)

{{% /capture %}}


