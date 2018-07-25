---
title: こんにちは Minikube
content_template: templates/tutorial
weight: 5
---

{{% capture overview %}}

<!--
The goal of this tutorial is for you to turn a simple Hello World Node.js app
into an application running on Kubernetes. The tutorial shows you how to
take code that you have developed on your machine, turn it into a Docker
container image and then run that image on [Minikube](/docs/getting-started-guides/minikube).
Minikube provides a simple way of running Kubernetes on your local machine for free.
-->
このチュートリアルの目的は、単純な Hello World Node.js アプリを Kubernetes 上のアプリケーションとして動かします。チュートリアルでは自分のマシン上で開発したコードの取り込み方を案内するため、Docker コンテナ・イメージにコードを入れて、それから [Minikube](/jp/docs/getting-started-guides/minikube) 上でイメージを実行します。Minikube は自分のローカル PC 上で自由に Kubernetes を実行するための、最も簡単な方法です。


{{% /capture %}}

{{% capture objectives %}}

<!--
* Run a hello world Node.js application.
* Deploy the application to Minikube.
* View application logs.
* Update the application image.
-->
* hello world（ハロー・ワールド） Node.js アプリケーションを実行
* アプリケーションを Minikube に展開（デプロイ）
* アプリケーションのログを表示
* アプリケーションのイメージを更新

{{% /capture %}}

{{% capture prerequisites %}}

<!--
* For OS X, you need [Homebrew](https://brew.sh) to install the `xhyve` driver.
-->
* OS X では `xhyve` ドライバをインストールするため、 [Homebrew](https://brew.sh) のインストールが必要です。

  {{< note >}}
<!--
  **Note:** If you see the following Homebrew error when you run `brew update` after you update your computer to MacOS 10.13:
  -->
  **メモ：**  もしも MacOS 10.13 にコンピュータを更新後、 `brew update` コマンドの実行維持に次のようなエラーが出る場合は：
  
  ```
  Error: /usr/local is not writable. You should change the ownership
  and permissions of /usr/local back to your user account:
  sudo chown -R $(whoami) /usr/local
  ```
  <!--You can resolve the issue by reinstalling Homebrew: -->
  この問題を解決するためには Homebrew を再インストールします。
  ```
  /usr/bin/ruby -e "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/master/install)"
  ```
  {{< /note >}}

<!--
* [NodeJS](https://nodejs.org/en/) is required to run the sample application.

* Install Docker. On OS X, we recommend
[Docker for Mac](https://docs.docker.com/engine/installation/mac/).
-->
* サンプル・アプリケーションの実行には[NodeJS](https://nodejs.org/ｊｐ/) が必要です。
* OS X に Docker をインストールするには、[Docker for Mac](https://docs.docker.com/engine/installation/mac/) を推奨します。

{{% /capture %}}

{{% capture lessoncontent %}}

<!--
## Create a Minikube cluster
-->
## Minikube クラスタの作成 {#create-a-minikube-cluster}

<!--
This tutorial uses [Minikube](https://github.com/kubernetes/minikube) to
create a local cluster. This tutorial also assumes you are using
[Docker for Mac](https://docs.docker.com/engine/installation/mac/)
on OS X. If you are on a different platform like Linux, or using VirtualBox
instead of Docker for Mac, the instructions to install Minikube may be
slightly different. For general Minikube installation instructions, see
the [Minikube installation guide](/docs/getting-started-guides/minikube/).
-->
このチュートリアルは [Minikube](https://github.com/kubernetes/minikube) を使ってクラスタを構築します。また、このチュートリアルでは [Docker for Mac](https://docs.docker.com/engine/installation/mac/) の利用を想定しています。Linux のような他のプラットフォーム上であれば、Docker for Mac の代わりに VirtualBox をお使い下さい。Minikube のインストール手順は若干変わる可能性があります。通常の Minikube インストール手順については [Minikube インストール・ガイド](/docs/getting-started-guides/minikube/) をご覧ください。

<!--
Use `curl` to download and install the latest Minikube release:
-->
最新の Minikube リリースのダウンロードとインストールに `curl` を使います。

```shell
curl -Lo minikube https://storage.googleapis.com/minikube/releases/latest/minikube-darwin-amd64 && \
  chmod +x minikube && \
  sudo mv minikube /usr/local/bin/
```

<!--
Use Homebrew to install the xhyve driver and set its permissions:
-->
Homebrew で xhyve ドライバをインストールし、実行権限を設定します：

```shell
brew install docker-machine-driver-xhyve
sudo chown root:wheel $(brew --prefix)/opt/docker-machine-driver-xhyve/bin/docker-machine-driver-xhyve
sudo chmod u+s $(brew --prefix)/opt/docker-machine-driver-xhyve/bin/docker-machine-driver-xhyve
```

<!--
Use Homebrew to download the `kubectl` command-line tool, which you can
use to interact with Kubernetes clusters:
-->
Homebrew で `kubectl` コマンドライン・ツールをダウンロードします。これは Kubernetes クラスタとやりとりするために使います。

```shell
brew install kubernetes-cli
```

<!--
Determine whether you can access sites like [https://cloud.google.com/container-registry/](https://cloud.google.com/container-registry/) directly without a proxy, by opening a new terminal and using
-->
 [https://cloud.google.com/container-registry/](https://cloud.google.com/container-registry/) のような、のサイトにアクセスするかを決めます。プロキシを使わず直接つなぐには、新しいターミナルをを開いて次のように実行します。

```shell
curl --proxy "" https://cloud.google.com/container-registry/
```

<!--
Make sure that the Docker daemon is started. You can determine if docker is running by using a command such as:
-->
Docker デーモンが起動しているかどうかを確認します。次のようなコマンドを使って、Docker が起動中かどうか判断できます。

```shell
docker images
```

<!--
If NO proxy is required, start the Minikube cluster:
-->
プロキシが不要であれば、Minikube クラスタを起動します：

```shell
minikube start --vm-driver=xhyve
```

<!--
If a proxy server is required, use the following method to start Minikube cluster with proxy setting:
-->
プロキシ・サーバが必要であれば、プロキシ設定と一緒に Minikube クラスタを起動する手順を使います：

```shell
minikube start --vm-driver=xhyve --docker-env HTTP_PROXY=http://your-http-proxy-host:your-http-proxy-port  --docker-env HTTPS_PROXY=http(s)://your-https-proxy-host:your-https-proxy-port
```

<!--
The `--vm-driver=xhyve` flag specifies that you are using Docker for Mac. The
default VM driver is VirtualBox.
-->
`--vm-driver=xhyve` フラグは Docker for Mac の使用時に指定します。デフォルトの VM ドライバは VirtualBox です。

<!--
Note if `minikube start --vm-driver=xhyve` is unsuccessful due to the error:
-->
ただし、 `minikube start --vm-driver=xhyve` が成功せず次のようなエラーがでる場合：
```
Error creating machine: Error in driver during machine creation: Could not convert the UUID to MAC address: exit status 1
```

<!--
Then the following may resolve the `minikube start --vm-driver=xhyve` issue:
-->
`minikube start --vm-driver=xhyve` に関する問題を解決するため、次の手順に従います：
```
rm -rf ~/.minikube
sudo chown root:wheel $(brew --prefix)/opt/docker-machine-driver-xhyve/bin/docker-machine-driver-xhyve
sudo chmod u+s $(brew --prefix)/opt/docker-machine-driver-xhyve/bin/docker-machine-driver-xhyve
```

<!--
Now set the Minikube context. The context is what determines which cluster
`kubectl` is interacting with. You can see all your available contexts in the
`~/.kube/config` file.
-->
次は Minikube コンテクスト（context）を設定します。コンテクストとは `kubectl` とやりとりするクラスタが何処かを示します。利用可能なコンテクストのすべてが `~/.kube/config` ファイル内にあります。

```shell
kubectl config use-context minikube
```

<!--
Verify that `kubectl` is configured to communicate with your cluster:
-->
`kubectl` がクラスタと通信可能に設定されているかどうかを確認します：

```shell
kubectl cluster-info
```
<!--
Open the Kubernetes dashboard in a browser:
-->
ブラウザで Kubernetes ダッシュボードを開きます：

```shell
minikube dashboard
```

<!--
## Create your Node.js application
-->
## Node.js アプリケーションの作成 {#create-your-nodejs-application}

<!--
The next step is to write the application. Save this code in a folder named `hellonode`
with the filename `server.js`:
-->
次のステップはアプリケーションを書きます。以下のコードを `hellonode` フォルダに `server.jp` という名前で保存します。

{{< code language="js" file="server.js" >}}

<!--
Run your application:
-->
アプリケーションを実行します：

```shell
node server.js
```

<!--
You should be able to see your "Hello World!" message at http://localhost:8080/.

Stop the running Node.js server by pressing **Ctrl-C**.

The next step is to package your application in a Docker container.
-->
"Hello World!"メッセージが http://localhost:8080/ で表示できるでしょう。

実行中の Node.js を停止するには **Ctrl-C** を使います。

次のステップはアプリケーションを Docker コンテナに梱包（パッケージ）します。

<!--
## Create a Docker container image
-->
## Docker コンテナ・イメージの作成 {#create-a-docker-container-image}

<!--
Create a file, also in the `hellonode` folder, named `Dockerfile`. A Dockerfile describes
the image that you want to build. You can build a Docker container image by extending an
existing image. The image in this tutorial extends an existing Node.js image.
-->
`hellonode` フォルダでは、 `Dockerfile` という名前のファイルも作成します。Dockerfile にはどのようにしてイメージを構築するか記述します。既存のイメージを拡張して Docker コンテナ・イメージを構築できます。このチュートリアルでは既存の Node.js イメージを拡張します。

{{< code language="conf" file="Dockerfile" >}}

<!--
This recipe for the Docker image starts from the official Node.js LTS image
found in the Docker registry, exposes port 8080, copies your `server.js` file
to the image and starts the Node.js server.
-->
この Docker イメージ用のレシピは Docker レジストリにある公式 Node.js LTS イメージを探す所から始めます。それからポート 8080 を公開し、作成した `server.js` ファイルをイメージ内にコピーし、Node.js サーバを起動します。

<!--
Because this tutorial uses Minikube, instead of pushing your Docker image to a
registry, you can simply build the image using the same Docker host as
the Minikube VM, so that the images are automatically present. To do so, make
sure you are using the Minikube Docker daemon:
-->
このチュートリアルでは Minikube を使うため、レジストリに Docker イメージを送信する代わりに、シンプルに Minikue VM が動く同じ Docker ホスト上でイメージを構築します。そのためには、Minikube Docker デーモンを使ってイメージを自動作成します。


```shell
eval $(minikube docker-env)
```

{{< note >}}
<!--
**Note:** Later, when you no longer wish to use the Minikube host, you can undo
this change by running `eval $(minikube docker-env -u)`.
-->
**メモ：** あとで Minikube ホストを使う必要がなくなれば、`eval $(minikube docker-env -u)` を実行し、変更を元に戻せます。
{{< /note >}}

<!--
Build your Docker image, using the Minikube Docker daemon (mind the trailing dot):
-->
Docker イメージの構築に Minikube Docker デーモンを使います（最後にドットが必要なのでご注意ください）：

```shell
docker build -t hello-node:v1 .
```

<!--
Now the Minikube VM can run the image you built.
-->
これで Minikube VM は構築したイメージを起動できます。

<!--
## Create a Deployment
-->
## デプロイメントの作成

<!--
A Kubernetes [*Pod*](/docs/concepts/workloads/pods/pod/) is a group of one or more Containers,
tied together for the purposes of administration and networking. The Pod in this
tutorial has only one Container. A Kubernetes
[*Deployment*](/docs/concepts/workloads/controllers/deployment/) checks on the health of your
Pod and restarts the Pod's Container if it terminates. Deployments are the
recommended way to manage the creation and scaling of Pods.
-->
Kubernetes [*ポッド（Pod）*](/jp/docs/concepts/workloads/pods/pod/) は１つまたは複数のコンテナのグループであり、管理とネットワーク機能の用途でつなぎ合わさっています。このチュートリアルのポッドは、コンテナが１つです。Kubernetes [*デプロイメント（Deployment）*](jp//docs/concepts/workloads/controllers/deployment/) はポッドの正常性を監視し、もしもコンテナが停止したら Pod はコンテナを再起動します。デプロイメントはポッドの作成とスケールを管理するために推奨されている方法です。

<!--
Use the `kubectl run` command to create a Deployment that manages a Pod. The
Pod runs a Container based on your `hello-node:v1` Docker image. Set the 
`--image-pull-policy` flag to `Never` to always use the local image, rather than
pulling it from your Docker registry (since you haven't pushed it there):
-->
`kubectl run` コマンドを使い、ポッドを管理するデプロイメントを作成します。ポッドは `hello-node:v1` Docker イメージを基盤としたコンテナを起動します。Docker レジストからではなく（まだイメージを送信していないので）、常にローカルにあるイメージを使うため、 `--image-pull-policy` （イメージ取得ポリシー）フラグを `Never` に指定します。


```shell
kubectl run hello-node --image=hello-node:v1 --port=8080 --image-pull-policy=Never
```

<!--
View the Deployment:
-->
デプロイメントを表示します：


```shell
kubectl get deployments
```

Output:


```shell
NAME         DESIRED   CURRENT   UP-TO-DATE   AVAILABLE   AGE
hello-node   1         1         1            1           3m
```

<!--
View the Pod:
-->
ポッドを表示します：

```shell
kubectl get pods
```

<!--
Output:
-->
出力：

```shell
NAME                         READY     STATUS    RESTARTS   AGE
hello-node-714049816-ztzrb   1/1       Running   0          6m
```

<!--
View cluster events:
-->
クラスタ・イベントの表示：

```shell
kubectl get events
```

<!--
View the `kubectl` configuration:
-->
`kubectl` 設定の表示

```shell
kubectl config view
```

<!--
For more information about `kubectl`commands, see the
[kubectl overview](/docs/user-guide/kubectl-overview/).
-->
`kubectl` コマンドに関する詳しい情報は [kubectl 概要](/jp/docs/reference/kubectl/overview/)をご覧ください。

<!--
## Create a Service
-->
## サービスの作成 {#create-a-service}
<!--
By default, the Pod is only accessible by its internal IP address within the
Kubernetes cluster. To make the `hello-node` Container accessible from outside the
Kubernetes virtual network, you have to expose the Pod as a
Kubernetes [*Service*](/docs/concepts/services-networking/service/).
-->
デフォルトでは、Pod は Kubernetes クラスタ内での内部 IP アドレスを通してのみ接続できます。 `hello-node` コンテナを Kuvernetes 仮想ネットワーク外から接続できるようにするには、ポッドを Kubernetes  [*サービス（Service）*](/jp/docs/concepts/services-networking/service/) として露出する必要があります。

<!--
From your development machine, you can expose the Pod to the public internet
using the `kubectl expose` command:
-->
展開したマシンで、ポッドをインターネットに露出するには `kubectl expose`コマンドを使います：


```shell
kubectl expose deployment hello-node --type=LoadBalancer
```

<!--
View the Service you just created:
-->
作成したサービスを表示します：

```shell
kubectl get services
```

Output:

```shell
NAME         CLUSTER-IP   EXTERNAL-IP   PORT(S)    AGE
hello-node   10.0.0.71    <pending>     8080/TCP   6m
kubernetes   10.0.0.1     <none>        443/TCP    14d
```

<!--
The `--type=LoadBalancer` flag indicates that you want to expose your Service
outside of the cluster. On cloud providers that support load balancers,
an external IP address would be provisioned to access the Service. On Minikube,
the `LoadBalancer` type makes the Service accessible through the `minikube service`
command.
-->
`--type=LoadBalancer` フラグはクラスタ外にサービスを露出したいと指示します。クラウド・プロバイダは負荷分散（load balancer）をサポートしており、外部 IP アドレスを通してサービスへ接続できるようになります。Minikube では `LoadBalancer` 型であれば、 `minikube service` コマンドを通してサービスに接続できるようになります。

```shell
minikube service hello-node
```

<!--
This automatically opens up a browser window using a local IP address that
serves your app and shows the "Hello World" message.
-->
これは自動的にローカル IP アドレスのブラウザ画面を開き、アプリケーションが "Hello World" メッセージを表示します。

<!--
Assuming you've sent requests to your new web service using the browser or curl,
you should now be able to see some logs:
-->
ブラウザや curl を通して新しいウェブサービスに送ったリクエストは、どれも同じログ上で参照できます：

```shell
kubectl logs <ポッド名>
```

<!--
## Update your app
-->
## アプリケーションの更新 {#update-you-app}

<!--
Edit your `server.js` file to return a new message:
-->
新しいメッセージを返すように `server.js` ファイルを編集します：


```javascript
response.end('Hello World Again!');

```

<!--
Build a new version of your image (mind the trailing dot):
-->
新しいバージョンのイメージを構築します（最後にドットがあるのを忘れないでください）：

```shell
docker build -t hello-node:v2 .
```

<!--
Update the image of your Deployment:
-->
デプロイメントのイメージを更新します：

```shell
kubectl set image deployment/hello-node hello-node=hello-node:v2
```

<!--
Run your app again to view the new message:
-->
アプリケーションを再実行し、新しいメッセージを表示します：

```shell
minikube service hello-node
```

<!--
## Enable addons
-->
## アドオンの有効化 {#enable-addons}

<!--
Minikube has a set of built-in addons that can be enabled, disabled and opened in the local Kubernetes environment.
-->
Minikube は一通りの内蔵アドオンがあります。これはローカルの Kubernetes 環境で有効化や無効化したり、開いたりできます。

<!--
First list the currently supported addons:
-->
まず、現在サポートされているアドオンを一覧表示します：

```shell
minikube addons list
```

<!--
Output:
-->
出力結果です：

```shell
- storage-provisioner: enabled
- kube-dns: enabled
- registry: disabled
- registry-creds: disabled
- addon-manager: enabled
- dashboard: disabled
- default-storageclass: enabled
- coredns: disabled
- heapster: disabled
- efk: disabled
- ingress: disabled
```

<!--
Minikube must be running for these commands to take effect. To enable `heapster` addon, for example:
-->
Minikube で有効化するには、コマンドを実行する必要があります。たとえば、 `heapster` アドオンを有効化するには：

```shell
minikube addons enable heapster
```

<!--
Output:
-->
出力結果：

```shell
heapster was successfully enabled
```

<!--
View the Pod and Service you just created:
-->
今作成したポッドとサービスを表示します：

```shell
kubectl get po,svc -n kube-system
```

<!--
Output:
-->
出力結果：

```shell
NAME                             READY     STATUS    RESTARTS   AGE
po/heapster-zbwzv                1/1       Running   0          2m
po/influxdb-grafana-gtht9        2/2       Running   0          2m

NAME                       TYPE        CLUSTER-IP   EXTERNAL-IP   PORT(S)             AGE
svc/heapster               NodePort    10.0.0.52    <none>        80:31655/TCP        2m
svc/monitoring-grafana     NodePort    10.0.0.33    <none>        80:30002/TCP        2m
svc/monitoring-influxdb    ClusterIP   10.0.0.43    <none>        8083/TCP,8086/TCP   2m
```

<!--
Open the endpoint to interacting with heapster in a browser:
-->
エンドポイントを開き、ブラウザで heapster を操作します：

```shell
minikube addons open heapster
```

<!--
Output:
-->
出力結果：

```shell
Opening kubernetes service kube-system/monitoring-grafana in default browser...
```

<!--
## Clean up
-->
## 後片付け {#cleanup}

<!--
Now you can clean up the resources you created in your cluster:
-->
あとは、クラスタで作成したリソースを片付けられます。

```shell
kubectl delete service hello-node
kubectl delete deployment hello-node
```

<!--
Optionally, force removal of the Docker images created:
-->
オプションで、作成した Docker イメージも強制削除します：

```shell
docker rmi hello-node:v1 hello-node:v2 -f
```

<!--
Optionally, stop the Minikube VM:
-->
オプションで、Minikube 仮想マシンを停止します：

```shell
minikube stop
eval $(minikube docker-env -u)
```

<!--
Optionally, delete the Minikube VM:
-->
オプションで、Minikube 仮想マシンを削除します：

```shell
minikube delete
```

{{% /capture %}}


{{% capture whatsnext %}}

<!--
* Learn more about [Deployment objects](/docs/concepts/workloads/controllers/deployment/).
* Learn more about [Deploying applications](/docs/user-guide/deploying-applications/).
* Learn more about [Service objects](/docs/concepts/services-networking/service/).
-->
* [デプロイメント・オブジェクト](/jp/docs/concepts/workloads/controllers/deployment/) を詳しく学ぶ
* [アプリケーションのデプロイ](/jp/docs/tasks/run-application/run-stateless-application-deployment/) を詳しく学ぶ
* [サービス・オブジェクト](/jp/docs/concepts/services-networking/service/) を詳しく学ぶ

{{% /capture %}}


