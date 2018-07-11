---
reviewers:
- justinsb
- clove
title: AWS EC2 で Kubernetes を動かす
---

{{< toc >}}



<!--
## Supported Production Grade Tools
-->
## サポートされている製品級（Production Grade）のツール

<!--
* [conjure-up](/docs/getting-started-guides/ubuntu/) is an open-source installer for Kubernetes that creates Kubernetes clusters with native AWS integrations on Ubuntu.
* [Kubernetes Operations](https://github.com/kubernetes/kops) - Production Grade K8s Installation, Upgrades, and Management. Supports running Debian, Ubuntu, CentOS, and RHEL in AWS.
* [CoreOS Tectonic](https://coreos.com/tectonic/) includes the open-source [Tectonic Installer](https://github.com/coreos/tectonic-installer) that creates Kubernetes clusters with Container Linux nodes on AWS.
* CoreOS originated and the Kubernetes Incubator maintains [a CLI tool, `kube-aws`](https://github.com/kubernetes-incubator/kube-aws), that creates and manages Kubernetes clusters with [Container Linux](https://coreos.com/why/) nodes, using AWS tools: EC2, CloudFormation and Autoscaling.

-->
* [conjure-up](/jp/docs/getting-started-guides/ubuntu/) は Kubernetes 用のオープンソースのインストーラーです。ネイティブに AWS と統合した Ubuntu 上に Kubernetes クラスタを構築します。
* [Kubernetes Operations](https://github.com/kubernetes/kops) - 製品級の K8s インストール、更新、管理をします。動作のサポートは AWS の Debian、Ubuntu、CentOS、RHEL です。
* [CoreOS Tectonic](https://coreos.com/tectonic/) には、 AWS 上で Container Linux ノードによる Kubernetes クラスタを構築するための、オープンソースの [Tectonic Installer](https://github.com/coreos/tectonic-installer) を含みます。
* CoreOS が考案し、Kubernetes incubator が保守している [CLI ツール `kube-aws`](https://github.com/kubernetes-incubator/kube-aws) は、[Container Linux](https://coreos.com/why/) による Kubernetes クラスタの構築・管理を行い、EC2、クラウドフォーメーション、オートスケーリングの AWS ツールを使います。

---

<!--
## Getting started with your cluster
-->
## クラスタの構築を始める


<!--
### Command line administration tool: `kubectl`
-->
### コマンドラインの管理ツール： `kubectl`

<!--
The cluster startup script will leave you with a `kubernetes` directory on your workstation.
Alternately, you can download the latest Kubernetes release from [this page](https://github.com/kubernetes/kubernetes/releases).
-->
クラスタ・スタートアップスクリプトはワークステーションで `kubernetes` を直接触れる必要がありません。あるいは、[こちらのページ](https://github.com/kubernetes/kubernetes/releases)から最新の Kubenetes リリースをダウンロードできます。


<!--
Next, add the appropriate binary folder to your `PATH` to access kubectl:
-->
それから、 kubectl にアクセスするため、 `PATH` に適切なバイナリ・フォルダを追加します。

```shell
# OS X
export PATH=<path/to/kubernetes-directory>/platforms/darwin/amd64:$PATH

# Linux
export PATH=<path/to/kubernetes-directory>/platforms/linux/amd64:$PATH
```

<!--
An up-to-date documentation page for this tool is available here: [kubectl manual](/docs/user-guide/kubectl/)
-->
このツールの最新のドキュメント用ページはこちらです：[kubectl マニュアル](/jp/docs/user-guide/kubectl/)

<!--
By default, `kubectl` will use the `kubeconfig` file generated during the cluster startup for authenticating against the API.
For more information, please read [kubeconfig files](/docs/tasks/access-application-cluster/configure-access-multiple-clusters/)
-->
デフォルトの `kubectl`は `kubeconfig` ファイルを用います。これはクラスタを立ち上げる間に、API 間で認証するために生成されるものです。詳しい情報については、[kubeconfig ファイル](/jp/docs/tasks/access-application-cluster/configure-access-multiple-clusters/)をご覧ください。

<!--
### Examples
-->
### 例


<!--
See [a simple nginx example](/docs/tasks/run-application/run-stateless-application-deployment/) to try out your new cluster.
-->
新しいクラスタで [簡単な nginx 例](/jp/docs/tasks/run-application/run-stateless-application-deployment/)をご覧ください。

<!--
The "Guestbook" application is another popular example to get started with Kubernetes: [guestbook example](https://github.com/kubernetes/examples/tree/{{< param "githubbranch" >}}/guestbook/)
-->
"Guestbook"（ゲストブック）アプリケーションは Kubernets を始めるうえで、別の有名な例です：[guestbook 例](https://github.com/kubernetes/examples/tree/{{< param "githubbranch" >}}/guestbook/)

<!--
For more complete applications, please look in the [examples directory](https://github.com/kubernetes/examples/tree/{{< param "githubbranch" >}}/)
-->
他のアプリケーションについては、[例のディレクトリ](https://github.com/kubernetes/examples/tree/{{< param "githubbranch" >}}/)をご覧ください。

<!--
## Scaling the cluster
-->
## クラスタのスケーリング（拡大縮小）

<!--
Adding and removing nodes through `kubectl` is not supported. You can still scale the amount of nodes manually through adjustments of the 'Desired' and 'Max' properties within the [Auto Scaling Group](http://docs.aws.amazon.com/autoscaling/latest/userguide/as-manual-scaling.html), which was created during the installation.
-->
`kubectl` を通したノードの追加と削除はサポートされていません。 インストール時に作成される [オートスケーリング・グループ](http://docs.aws.amazon.com/autoscaling/latest/userguide/as-manual-scaling.html) 内の `Desired` と `Max`プロパティの調整を通してノードをスケールできます。

<!--
## Tearing down the cluster
-->
## クラスタのティアダウン（解体）

<!--
Make sure the environment variables you used to provision your cluster are still exported, then call the following script inside the
`kubernetes` directory:
-->
構築して使おうとしているクラスタの環境変数がエクスポートされているかどうか確認してください。それから `kubernetes` ディレクトリでスクリプトを実行します。

```shell
cluster/kube-down.sh
```

<!--
## Support Level
-->
## サポートレベル

<!--
IaaS Provider        | Config. Mgmt | OS            | Networking  | Docs                                          | Conforms | Support Level
-------------------- | ------------ | ------------- | ----------  | --------------------------------------------- | ---------| ----------------------------
AWS                  | kops         | Debian        | k8s (VPC)   | [docs](https://github.com/kubernetes/kops)    |          | Community ([@justinsb](https://github.com/justinsb))
AWS                  | CoreOS       | CoreOS        | flannel     | [docs](/docs/getting-started-guides/aws)      |          | Community
AWS                  | Juju         | Ubuntu        | flannel, calico, canal     | [docs](/docs/getting-started-guides/ubuntu)      | 100%     | Commercial, Community
-->
IaaS 事業者        | 構成管理 | OS            | ネットワーク  | 文章                                          | 適合率 | サポートレベル
-------------------- | ------------ | ------------- | ----------  | --------------------------------------------- | ---------| ----------------------------
AWS                  | kops         | Debian        | k8s (VPC)   | [docs](https://github.com/kubernetes/kops)    |          | コミュニティ ([@justinsb](https://github.com/justinsb))
AWS                  | CoreOS       | CoreOS        | flannel     | [docs](/docs/getting-started-guides/aws)      |          | コミュニティ
AWS                  | Juju         | Ubuntu        | flannel, calico, canal     | [docs](/docs/getting-started-guides/ubuntu)      | 100%     | 商用、コミュニティ


<!--
For support level information on all solutions, see the [Table of solutions](/docs/getting-started-guides/#table-of-solutions) chart.
-->
全てのソリューションに対するサポートレベルは、 [ソリューション一覧](/jp/docs/getting-started-guides/#table-of-solutions) 表をご覧ください。

<!--
## Further reading
-->
## さらに詳しく

<!--
Please see the [Kubernetes docs](/docs/) for more details on administering
and using a Kubernetes cluster.
-->
Kubernetes クラスタの管理と利用に関する詳細は [Kubernetes ドキュメント](/jp/docs/)をご覧ください。
