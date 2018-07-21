---
title: Minikube のインストール
content_template: templates/task
weight: 20
---

{{% capture overview %}}

<!--
This page shows how to install Minikube.
-->
このページは Minikube のインストール方法をご案内します。

{{% /capture %}}

{{% capture prerequisites %}}

<!--
VT-x or AMD-v virtualization must be enabled in your computer's BIOS.
-->
コンピュータの BIOS で VT-x または AMD-v 仮想化を有効にする必要があります。

{{% /capture %}}

{{% capture steps %}}

<!--
## Install a Hypervisor
-->
## ハイパーバイザのインストール {#install-a-hypervisor}

<!--
If you do not already have a hypervisor installed, install one now.
-->
ハイパーバイザをインストールしていなければ、どれか１つを今インストールします。

<!--
* For OS X, install
[VirtualBox](https://www.virtualbox.org/wiki/Downloads) or
[VMware Fusion](https://www.vmware.com/products/fusion), or
[HyperKit](https://github.com/moby/hyperkit).
* For Linux, install
[VirtualBox](https://www.virtualbox.org/wiki/Downloads) or
[KVM](http://www.linux-kvm.org/).
-->
* OS X 用は、[VirtualBox](https://www.virtualbox.org/wiki/Downloads) か [VMware Fusion](https://www.vmware.com/products/fusion), か [HyperKit](https://github.com/moby/hyperkit) をインストールします。

* Linux 用は [VirtualBox](https://www.virtualbox.org/wiki/Downloads) か [KVM](http://www.linux-kvm.org/) をインストールします。

  {{< note >}}
<!--
   **Note:** Minikube also supports a `-\-vm-driver=none` option that runs the Kubernetes components on the host and not in a VM.  Docker is required to use this driver but a hypervisor is not required.
-->
  **メモ：** Minikube は `-\-vm-driver=none` オプションもサポートしています。これは Kubernetes 構成要素を仮想マシンではないホスト所宇にインストールする時に指定します。Docker ではこのドライバの指定が必要ですが、ハイパーバイザは不要です。
  {{< /note >}}

<!--
* For Windows, install
[VirtualBox](https://www.virtualbox.org/wiki/Downloads) or
[Hyper-V](https://msdn.microsoft.com/en-us/virtualization/hyperv_on_windows/quick_start/walkthrough_install).
-->
* Windows 用は、[VirtualBox](https://www.virtualbox.org/wiki/Downloads) または [Hyper-V](https://msdn.microsoft.com/en-us/virtualization/hyperv_on_windows/quick_start/walkthrough_install) をインストールします。

<!--
## Install kubectl
-->
## kubectl のインストール {#install-kubectl}

* [kubectl のインストール](/jp/docs/tasks/tools/install-kubectl/).

<!--
## Install Minikube
-->
## Minikube のインストール {#install-minikube}

<!--
* Install Minikube according to the instructions for the
[latest release](https://github.com/kubernetes/minikube/releases).
-->
* Minikube を [最新リリース](https://github.com/kubernetes/minikube/releases) の手順に従ってインストールします。

{{% /capture %}}

{{% capture whatsnext %}}
<!--
* [Running Kubernetes Locally via Minikube](/docs/getting-started-guides/minikube/)
-->
* [Minikube を経由して Kubernetes をローカルで](/jp/docs/getting-started-guides/minikube/)


{{% /capture %}}


