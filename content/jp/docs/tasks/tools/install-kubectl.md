---
reviewers:
- bgrant0607
- mikedanese
title: kubectl のインストールとセットアップ
content_template: templates/task
weight: 10
---

{{% capture overview %}}
<!--
Use the Kubernetes command-line tool, [kubectl](/docs/user-guide/kubectl/), to deploy and manage applications on Kubernetes. Using kubectl, you can inspect cluster resources; create, delete, and update components; and look at your new cluster and bring up example apps.
-->
Kubernetes コマンドライン・ツールの [kubectl](/jp/docs/reference/kubectl/kubectl/) を使い、Kubernetes 上のアプリケーションを展開（デプロイ）・管理できます。kubectl でクラスタのリソースを調査し、構成要素（コンポーネント）の作成、削除、更新ができます。そして、新しいクラスタでサンプル・アプリを起動する方法を見ていきましょう。

{{% /capture %}}

{{% capture prerequisites %}}
<!--
Use a version of kubectl that is the same version as your server or later. Using an older kubectl with a newer server might produce validation errors.
-->
kubectl で利用するバージョンは、サーバと同じか以降のバージョンをお使いください。古い kubectl で新しいサーバを操作すると妥当性のエラーを引き起こすでしょう。
{{% /capture %}}


{{% capture steps %}}
<!--
## Install kubectl
-->
## kubectl のインストール {#install-kubectl}

<!--
Here are a few methods to install kubectl.
-->
kubectl をインストールするには複数の方法があります。

<!--
## Install kubectl binary via native package management
-->
## パッケージ管理を通して kubectl バイナリをインストール

{{< tabs name="kubectl_install" >}}
{{< tab name="Ubuntu、Debian、HypriotOS" codelang="bash" >}}
sudo apt-get update && sudo apt-get install -y apt-transport-https
curl -s https://packages.cloud.google.com/apt/doc/apt-key.gpg | sudo apt-key add -
sudo touch /etc/apt/sources.list.d/kubernetes.list 
echo "deb http://apt.kubernetes.io/ kubernetes-xenial main" | sudo tee -a /etc/apt/sources.list.d/kubernetes.list
sudo apt-get update
sudo apt-get install -y kubectl
{{< /tab >}}
{{< tab name="CentOS、RHEL、Fedora" codelang="bash" >}}cat <<EOF > /etc/yum.repos.d/kubernetes.repo
[kubernetes]
name=Kubernetes
baseurl=https://packages.cloud.google.com/yum/repos/kubernetes-el7-x86_64
enabled=1
gpgcheck=1
repo_gpgcheck=1
gpgkey=https://packages.cloud.google.com/yum/doc/yum-key.gpg https://packages.cloud.google.com/yum/doc/rpm-package-key.gpg
EOF
yum install -y kubectl
{{< /tab >}}
{{< /tabs >}}

<!--
## Install with snap on Ubuntu
-->
## Ubuntu に snap でインストール {#install-with-snap-on-ubuntu}

<!--
kubectl is available as a [snap](https://snapcraft.io/) application.
-->
kubectl は [snap](https://snapcraft.io/) アプリケーションとして利用可能です。

<!--
1. If you are on Ubuntu or one of other Linux distributions that support [snap](https://snapcraft.io/docs/core/install) package manager, you can install with:
-->
1. Ubuntu を使っているか、 [snap](https://snapcraft.io/docs/core/install) パッケージ・マネジャーをサポートしている他の Linux ディストリビューションであれば、次のようにインストールできます：

        sudo snap install kubectl --classic

<!--2. Run `kubectl version` to verify that the version you've installed is sufficiently up-to-date.-->2. `kubectl version` を実行し、インストールされたバージョンが十分な最新版かどうかを確認します。

<!--
## Install with Homebrew on macOS
-->
## macOS に Homebrew でインストール {#install-with-homebrew-on-macos}

<!--
1. If you are on macOS and using [Homebrew](https://brew.sh/) package manager, you can install with:
-->
1. macOS を使っている場合は、[Homebrew](https://brew.sh/) パッケージ・マネジャーを使い、次のようにインストールできます：

        brew install kubernetes-cli

<!--2. Run `kubectl version` to verify that the version you've installed is sufficiently up-to-date.--> 2. `kubectl version` を実行し、インストールされたバージョンが十分な最新版かどうかを確認します。

<!--
## Install with Powershell from PSGallery
-->
## PSGallery から Powershell でインストール {#install-with-powershell-from-psgallery}

<!--
1. If you are on Windows and using [Powershell Gallery](https://www.powershellgallery.com/) package manager, you can install and update with:
-->
1. Windows で [Powershell Gallery](https://www.powershellgallery.com/) パッケージ・マネジャーを使う場合は、インストールと更新を次のように行えます：

        Install-Script -Name install-kubectl -Scope CurrentUser -Force
        install-kubectl.ps1 [-DownloadLocation <パス>]

<!--If no Downloadlocation is specified, kubectl will be installed in users temp Directory-->Downloadlocation（ダウンロードする場所）を指定しなければ、kubectl はユーザの temp ディレクトリにインストールされます。

<!--2. The installer creates $HOME/.kube and instructs it to create a config file-->2. インストーラは $HOME/.kube を作成し、設定ファイルの作成を指示します。

<!-- 3. Updating
re-run Install-Script to update the installer
re-run install-kubectl.ps1 to install latest binaries-->3. 最新のバイナリをインストールするには、 インストーラ install-kubectl.ps1 を再実行します。

<!--
## Install with Chocolatey on Windows
-->
## Windows の Chocolatey でインストール {#install-with-chocolatey-on-windows}

<!--
1. If you are on Windows and using [Chocolatey](https://chocolatey.org) package manager, you can install with:
-->
1. Windows で [Chocolatey](https://chocolatey.org) パッケージ・マネジャーを使う場合は、次のようにインストールできます：

        choco install kubernetes-cli

<!--2. Run `kubectl version` to verify that the version you've installed is sufficiently up-to-date.-->2. `kubectl version` を実行し、インストールされたバージョンが十分な最新版かどうかを確認します。

<!--3. Configure kubectl to use a remote Kubernetes cluster:-->3. kubectl がリモートの Kubernetes クラスタを使えるように設定します：

        cd C:\users\yourusername (Or wherever your %HOME% directory is)
        mkdir .kube
        cd .kube
        New-Item config -type file

<!--Edit the config file with a text editor of your choice, such as Notepad for example.-->
メモ帳（Notepad）などの任意のテキスト・エディタを使って設定ファイルを編集します。

<!--
## Download as part of the Google Cloud SDK
-->
## Google Cloud SDK の一部としてダウンロード {#download-as-part-of-the-google-cloud-sdk}

<!--
kubectl can be installed as part of the Google Cloud SDK.
-->
kubectl は Google Cloud SDK の一部としてインストールできます。

<!-- 1. Install the [Google Cloud SDK](https://cloud.google.com/sdk/). -->1. [Google Cloud SDK](https://cloud.google.com/sdk/) をインストールします。
<!--2. Run the following command to install `kubectl`:-->2. `kubectl` をインストールするため次のコマンドを実行します：

        gcloud components install kubectl

<!--3. Run `kubectl version` to verify that the version you've installed is sufficiently up-to-date.-->3. `kubectl version` を実行し、インストールされたバージョンが十分な最新版かどうかを確認します。

<!--
## Install kubectl binary via curl
-->
## kubectl バイナリを curl 経由でインストール {#install-kubectl-binary-via-curl}

{{< tabs name="kubectl_install_curl" >}}
{{% tab name="macOS" %}}
<!--1. Download the latest release with the command:-->
1. コマンドで最新のリリースをダウンロードします：

    ```		 
    curl -LO https://storage.googleapis.com/kubernetes-release/release/$(curl -s https://storage.googleapis.com/kubernetes-release/release/stable.txt)/bin/darwin/amd64/kubectl
    ```

    特定のバージョンをダウンロードするには、コマンドの `$(curl -s https://storage.googleapis.com/kubernetes-release/release/stable.txt)`  の箇所を対象バージョンに書き換えます。
    <!--To download a specific version, replace the `$(curl -s https://storage.googleapis.com/kubernetes-release/release/stable.txt)` portion of the command with the specific version.-->

    <!--For example, to download version {{< param "fullversion" >}} on MacOS, type:-->
    例えば MacOS のバージョン  {{< param "fullversion" >}} をダウンロードするには、次のように実行します：
		  
    ```
    curl -LO https://storage.googleapis.com/kubernetes-release/release/{{< param "fullversion" >}}/bin/darwin/amd64/kubectl
    ```

<!--2. Make the kubectl binary executable.-->2. kubectl バイナリを実行可能にします。
    ```
    chmod +x ./kubectl
    ```

<!--3. Move the binary in to your PATH.-->3. バイナリをパスに移動します。
    ```
    sudo mv ./kubectl /usr/local/bin/kubectl
    ```
{{% /tab %}}
{{% tab name="Linux" %}}

<!--1. Download the latest release with the command:-->
1. コマンドで最新のリリースをダウンロードします。

    ```
    curl -LO https://storage.googleapis.com/kubernetes-release/release/$(curl -s https://storage.googleapis.com/kubernetes-release/release/stable.txt)/bin/linux/amd64/kubectl
    ```

    <!--To download a specific version, replace the `$(curl -s https://storage.googleapis.com/kubernetes-release/release/stable.txt)` portion of the command with the specific version.-->
    特定のバージョンをダウンロードするには、コマンドの `$(curl -s https://storage.googleapis.com/kubernetes-release/release/stable.txt)`  の箇所を対象バージョンに書き換えます。

    <!--For example, to download version {{< param "fullversion" >}} on Linux, type:-->
    例えば Linux のバージョン  {{< param "fullversion" >}} をダウンロードするには、次のように実行します：
    ```
    curl -LO https://storage.googleapis.com/kubernetes-release/release/{{< param "fullversion" >}}/bin/linux/amd64/kubectl
    ```

<!--2. Make the kubectl binary executable.-->2. kubectl バイナリを実行可能にします。
    ```
    chmod +x ./kubectl
    ```

<!--3. Move the binary in to your PATH.-->3. バイナリをパスに移動します。
    ```
    sudo mv ./kubectl /usr/local/bin/kubectl
    ```
{{% /tab %}}
{{% tab  name="Windows" %}}
<!--
1. Download the latest release {{< param "fullversion" >}} from [this link](https://storage.googleapis.com/kubernetes-release/release/{{< param "fullversion" >}}/bin/windows/amd64/kubectl.exe).
-->
1. [こちらのリンク](https://storage.googleapis.com/kubernetes-release/release/{{< param "fullversion" >}}/bin/windows/amd64/kubectl.exe) から最新のリリースをダウンロードします。

    <!--Or if you have `curl` installed, use this command:-->
    あるいは `curl` をインストールしていれば、こちらのコマンドを使います：
    ```
    curl -LO https://storage.googleapis.com/kubernetes-release/release/{{< param "fullversion" >}}/bin/windows/amd64/kubectl.exe
    ```

    <!--To find out the latest stable version (for example, for scripting), take a look at [https://storage.googleapis.com/kubernetes-release/release/stable.txt](https://storage.googleapis.com/kubernetes-release/release/stable.txt).-->
    最新の安定バージョン（例えばスクリプト用など）を見つけるには、[https://storage.googleapis.com/kubernetes-release/release/stable.txt](https://storage.googleapis.com/kubernetes-release/release/stable.txt) を探します。

2. バイナリをパスに移動します。
{{% /tab %}}
{{< /tabs >}}


<!--
## Configure kubectl
-->
## kubectl の設定 {#configure-kubectl}

<!--
In order for kubectl to find and access a Kubernetes cluster, it needs a [kubeconfig file](/docs/tasks/access-application-cluster/configure-access-multiple-clusters/), which is created automatically when you create a cluster using kube-up.sh or successfully deploy a Minikube cluster. See the [getting started guides](/docs/setup/) for more about creating clusters. If you need access to a cluster you didn't create, see the [Sharing Cluster Access document](/docs/tasks/access-application-cluster/configure-access-multiple-clusters/).
By default, kubectl configuration is located at `~/.kube/config`.
-->
kubectl で Kubernetes クラスタを発見して接続するには、[kubeconfig ファイル](/jp/docs/tasks/access-application-cluster/configure-access-multiple-clusters/)が必要です。これは kube-up.sh を使ってクラスタを作成するか、Minikube クラスタの展開が成功したら作成されます。クラスタ作成に関する詳細は [導入ガイド]](/jp/docs/setup/) をご覧ください。自分で作成していないクラスタに接続する必要がある場合は、[クラスタ接続共有ドキュメント](/docs/tasks/access-application-cluster/configure-access-multiple-clusters/) をご覧ください。デフォルトでは、kubectl の設定ファイルは `~/.kube/config` にあります。

<!--
## Check the kubectl configuration
-->
## kubectl 設定ファイルの確認 {#check-the-kubectl-configuration}

<!--
Check that kubectl is properly configured by getting the cluster state:
-->
kubectl が適切に設定されているかどうかを、クラスタの状態を取得して調べます：


```shell
kubectl cluster-info
```

<!--
If you see a URL response, kubectl is correctly configured to access your cluster.
-->
URL の応答が表示されれば、kubectl はクラスタに接続できるように正しく設定されています。

<!--
If you see a message similar to the following, kubectl is not correctly configured or not able to connect to a Kubernetes cluster.
-->
もしも次のような表示がされれば、kubectl が正しく設定されていないか、Kubernetes クラスタに接続できません。

```shell
The connection to the server <server-name:port> was refused - did you specify the right host or port?
```

<!--
For example, if you are intending to run a Kubernetes cluster on your laptop (locally), you will need a tool like minikube to be installed first and then re-run the commands stated above.
-->
たとえば、Kuberntes クラスタを自分のノート PC 上で（ローカルに）実行したい場合は、minikube のようなツールを使ってインストールをしてから、先ほどのコマンドを再実行します。

<!--
If kubectl cluster-info returns the url response but you can't access your cluster, to check whether it is configured properly, use:
-->
もしも kubectl クラスタ情報の URL 応答があってもクラスタに接続できなければ、次のコマンドを使い、設定ファイルの場所が適切かどうかを確認します。

```shell
kubectl cluster-info dump
```

<!--
## Enabling shell autocompletion
-->
## シェル自動補完の有効化 {#enabling-shell-autocompletion}

<!--
kubectl includes autocompletion support, which can save a lot of typing!
-->
kubectl は自動補完に対応していますので、たくさんの入力をしなくても済みます！

<!--
The completion script itself is generated by kubectl, so you typically just need to invoke it from your profile.
-->
補完スクリプト自身は kubetl によって作成されるため、あとは一般的に自分の profile に移動して呼び出すだけです。

<!--
Common examples are provided here. For more details, consult `kubectl completion -h`.
-->
共通する例がここで表示されます。より詳しくは `kubectl completion -h` をご覧ください。

<!--
### On Linux, using bash
-->
### Linux 上で bash を使う {#on-linux-using-bash}

<!--
On CentOS Linux, you may need to install the bash-completion package which is not installed by default.
-->
CentOS Linux 上では bash-completion パッケージがデフォルトではインストールされていないため、まずインストールする必要があります。

```shell
yum install bash-completion -y
```

<!--
To add kubectl autocompletion to your current shell, run `source <(kubectl completion bash)`.
-->
kubectl 自動補完を追加するには、現在のシェル上で `source <(kubectl completion bash)` を実行します。

<!--
To add kubectl autocompletion to your profile, so it is automatically loaded in future shells run:
-->
profile に kubectl 自動補完を追加するには、次回以降のシェルで自動的に読み込まれるようにすため、次のように実行します：

```shell
echo "source <(kubectl completion bash)" >> ~/.bashrc
```

<!--
### On macOS, using bash
-->
### macOS 上で bash を使う {#on-macos-using-bash}

<!--
On macOS, you will need to install bash-completion support via [Homebrew](https://brew.sh/) first:
-->
macOS では、まず bash-completion サポートを [Homebrew](https://brew.sh/) 経由でインストールする必要があります。

<!--
```shell
## If running Bash 3.2 included with macOS
brew install bash-completion
## or, if running Bash 4.1+
brew install bash-completion@2
```
-->
```shell
## macOS で Bash 3.2 を実行している場合
brew install bash-completion
## Bash 4.1 以上を実行している場合
brew install bash-completion@2
```

<!--
Follow the "caveats" section of brew's output to add the appropriate bash completion path to your local .bashrc.
-->
brew の "caveats" セクションの出力を確認し、自分のローカルにある .bashrc に bash 補完のための適切なパスを追加します。

<!--
If you've installed kubectl using the [Homebrew instructions](#install-with-homebrew-on-macos) then kubectl completion should start working immediately.
-->
kubectl のインストールを [Homebrew 手順](#install-with-homebrew-on-macos) で行っている場合、kuectl 補完を直ちに使えます。

<!--
If you have installed kubectl manually, you need to add kubectl autocompletion to the bash-completion:
-->
kubectl を手作業でインストールした場合、kubectl 自動補完を bash-completion に追加する必要があります：

```shell
kubectl completion bash > $(brew --prefix)/etc/bash_completion.d/kubectl
```

<!--
The Homebrew project is independent from Kubernetes, so the bash-completion packages are not guaranteed to work.
-->
Homebrew プロジェクトは Kubernetes とは独立しているため、bash-completion パッケージの動作保証はありません。

<!--
### Using Zsh
-->
### Zsh を使う {#using-zsh}

<!--
If you are using zsh edit the ~/.zshrc file and add the following code to enable kubectl autocompletion:
-->
zsh を使っている場合は ~/.zshrc ファイルを編集し、以下のコードを追加して kubectl 自動補完を有効化します。

```shell
if [ $commands[kubectl] ]; then
  source <(kubectl completion zsh)
fi
```

<!--
Or when using [Oh-My-Zsh](http://ohmyz.sh/), edit the ~/.zshrc file and update the `plugins=` line to include the kubectl plugin.
-->
あるいは [Oh-My-Zsh](http://ohmyz.sh/) を使っている場合は ~/.zshrc ファイルを編集し、 `plugins=` 行で kubectl プラグインを読み込むように編集します。

```shell
source <(kubectl completion zsh)
```
{{% /capture %}}

{{% capture whatsnext %}}
<!--
[Learn how to launch and expose your application.](/docs/tasks/access-application-cluster/service-access-application-cluster/)
-->
[アプリケーションの起動と公開のしかたを学ぶ。](/docs/tasks/access-application-cluster/service-access-application-cluster/)
{{% /capture %}}

