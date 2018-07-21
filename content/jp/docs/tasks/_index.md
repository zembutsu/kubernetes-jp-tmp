---
title: タスク
main_menu: true
weight: 50
content_template: templates/concept
---

{{< toc >}}

{{% capture overview %}}
<!--
This section of the Kubernetes documentation contains pages that
show how to do individual tasks. A task page shows how to do a
single thing, typically by giving a short sequence of steps.
-->
Kubernetes ドキュメントの本セクションは、個々のタスク（作業）の仕方を指示します。タスクのページでは１つのことをどのように行うかを指示します。短い手順が順番通りにならんでいるのがほとんどです。

{{% /capture %}}

{{% capture body %}}

<!--
## Web UI (Dashboard)
-->
## Web UI（ダッシュボード） {#web-ui-dashboard}

<!--
Deploy and access the Dashboard web user interface to help you manage and monitor containerized applications in a Kubernetes cluster.
-->
ダッシュボード・ウェブ・ユーザインターフェイスの展開と接続です。これは Kubernetes クラスタでコンテナ化したアプリケーションの管理と監視に役立ちます。

<!--
## Using the kubectl Command-line
-->
## kubectl コマンドラインを使う {#using-the-kubectl-command-line}

<!--
Install and setup the `kubectl` command-line tool used to directly manage Kubernetes clusters.
-->
Kubernetes クラスタを直接管理するために、 `kubectl` コマンドライン・ツールのインストールとセットアップをします。

<!--
## Configuring Pods and Containers
-->
## ポッドとコンテナの設定 {#configuring-pods-and-containers}

<!--
Perform common configuration tasks for Pods and Containers.
-->
ポッドとコンテナに対する共通設定タスクを行います。

<!--
## Running Applications
-->
## アプリケーション実行 {#running-application}

<!--
Perform common application management tasks, such as rolling updates, injecting information into pods, and horizontal Pod autoscaling.
-->
ローリング・アップデート、ポッドに対する情報の投入、水平ポッド自動スケーリングなど、アプリケーション管理タスクを行います。

<!--
## Running Jobs
-->
## ジョブの実行 {#running-jobs}

<!--
Run Jobs using parallel processing.
-->
並列プロセス処理を使ってジョブを実行します。

<!--
## Accessing Applications in a Cluster
-->
## クラスタ内のアプリケーションに接続 {#accessing-application-in-a-cluster}

<!--
Configure load balancing, port forwarding, or setup firewall or DNS configurations to access applications in a cluster.
-->
クラスタ内のアプリケーションに接続するために、負荷分散の設定、ポート転送、ファイアウォールや DNS 設定を行います。

<!--
## Monitoring, Logging, and Debugging
-->
## 監視、ログ記録、デバッグ {#monitoring-logging-and-debugging}

<!--
Setup monitoring and logging to troubleshoot a cluster or debug a containerized application.
-->
クラスタのトラブルシュートやコンテナ化アプリケーションのセットアップ監視やログの記録をします。

<!--
## Accessing the Kubernetes API
-->
## Kubernetes API に接続 {#accessing-the-kubernetes-api}

<!--
Learn various methods to directly access the Kubernetes API.
-->
Kubernetes API に直接接続するための様々な手法を学びます。

<!--
## Using TLS
-->
## TLS を使う {#using-tls}

<!--
Configure your application to trust and use the cluster root Certificate Authority (CA).
-->
アプリケーションを信頼して使うために、クラスタのルート認証局（CA）を設定します。

<!--
## Administering a Cluster
-->
## クラスタ管理 {#administrering-a-cluster}

<!--
Learn common tasks for administering a cluster.
-->
クラスタを管理する共通したタスクを学びます。

<!--
## Administering Federation
-->
## 統合管理 {#administering-federation}

<!--
Configure components in a cluster federation.
-->
クラスタを統合するコ構成要素（コンポーネント）を設定します。

<!--
## Managing Stateful Applications
-->
## ステートフルなアプリケーションを管理 {#managing-stateful-applications}

<!--
Perform common tasks for managing Stateful applications, including scaling, deleting, and debugging StatefulSets.
-->
スケーリング、削除、StatefulSets のデバッグを含む、ステートフルなアプリケーションに共通するタスクを行います。

<!--
## Cluster Daemons
-->
## クラスタ・デーモン {#cluster-daemons}

<!--
Perform common tasks for managing a DaemonSet, such as performing a rolling update.
-->
ローリング・アップデートの処理を含む DaemonSets に共通するタスクを行います。

<!--
## Managing GPUs
-->
## GPU の管理

<!--
Configure and schedule NVIDIA GPUs for use as a resource by nodes in a cluster.
-->
クラスタ内にあるノードのリソースを使い、NVIDIA GPU の設定とスケジュールをします。

<!--
## Managing HugePages
-->
## HugePages の管理 {#maaging-hugepages}

<!--
Configure and schedule huge pages as a schedulable resource in a cluster.
-->
huge page をクラスタでスケジュール可能なリソースとして使い、設定とスケジュールをします。

{{% /capture %}}

{{% capture whatsnext %}}

<!--
If you would like to write a task page, see
[Creating a Documentation Pull Request](/docs/home/contribute/create-pull-request/).
-->
タスクページを記述したい場合は、 [ドキュメントのプルリクエスト作成](/docs/home/contribute/create-pull-request/) をご覧ください。


{{% /capture %}}
