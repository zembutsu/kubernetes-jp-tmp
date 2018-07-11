---
title: kubeadm のトラブルシューティング
weight: 80
---

<!--
As with any program, you might run into an error using or operating it. Below we have listed 
common failure scenarios and have provided steps that will help you to understand and hopefully
fix the problem.
-->
あらゆるプログラムと同様、使用および操作時は、エラーに遭遇する場合があります。以下の一覧は共通する障害シナリオと対応手順です。状況の理解と、願わくば問題解決に役立つでしょう。

<!--
If your problem is not listed below, please follow the following steps:

- If you think your problem is a bug with kubeadm:
  - Go to [github.com/kubernetes/kubeadm](https://github.com/kubernetes/kubeadm/issues) and search for existing issues.
  - If no issue exists, please [open one](https://github.com/kubernetes/kubeadm/issues/new) and follow the issue template.

- If you are unsure about how kubeadm or kubernetes works, and would like to receive
  support about your question, please ask on Slack in #kubeadm, or open a question on StackOverflow. Please include
  relevant tags like `#kubernetes` and `#kubeadm` so folks can help you.

If your cluster is in an error state, you may have trouble in the configuration if you see Pod statuses like `RunContainerError`,
`CrashLoopBackOff` or `Error`. If this is the case, please read below.
-->

以下に掲載されていない問題があれば、次の手順に従ってください。

- kubeadm のバグによる問題と考えられる場合:
  - [github.com/kubernetes/kubeadm](https://github.com/kubernetes/kubeadm/issues) に移動し、これまでの issue を検索します。
  - issue が無ければ、[issue をオープン](https://github.com/kubernetes/kubeadm/issues/new) し、issue テンプレートに従います。

- kubeadm か kubernetes の動作か確信が持てない場合、質問に対する支援を受けたい場合は、 Slack の #kubeadm でお訊ねいただくか、StackOverflow でオープンに質問してください。関連タグには  `#kubernetes` と `#kubeadm` があれば皆に役立つでしょう。

もしもクラスタがエラー状態の場合、ポッド状態が  `RunContainerError`、`CrashLoopBackOff`、`Error`であれば、設定上の問題の可能性があります。そのような場合は、以下をご覧ください。

<!--
#### `ebtables` or some similar executable not found during installation
-->
#### `ebtables` や類似実行コマンドがインストール中に見つからない

<!--
If you see the following warnings while running `kubeadm init`
-->
`kubeadm init` 実行時に以下のような警告が表示される場合

```sh
[preflight] WARNING: ebtables not found in system path
[preflight] WARNING: ethtool not found in system path
```

<!--
Then you may be missing `ebtables`, `ethtool` or a similar executable on your node. You can install them with the following commands:
-->

`ebtables` や `ethtool`等がノード上に無い可能性があります。次のコマンドでインストールできます。

<!--
- For Ubuntu/Debian users, run `apt install ebtables ethtool`.
- For CentOS/Fedora users, run `yum install ebtables ethtool`.
-->

- Ubuntu/Debian をお使いの場合、`apt install ebtables ethtool`を実行します。
- CentOS/Fedora をお使いの場合、`yum install ebtables ethtool`を実行します。

<!--
#### kubeadm blocks waiting for control plane during installation
-->
#### コントロール・プレーンのインストール時、kubeadm が固まる

<!--
If you notice that `kubeadm init` hangs after printing out the following line:
-->

以下の行を表示したあと `kubeadm init` が固まる場合、

```sh
[apiclient] Created API client, waiting for the control plane to become ready
```
<!--
This may be caused by a number of problems. The most common are:
-->
これには、いくつかの問題発生が考えられます。多くの場合で共通するのは、

<!--
- network connection problems. Check that your machine has full network connectivity before continuing.
- the default cgroup driver configuration for the kubelet differs from that used by Docker.
  Check the system log file (e.g. `/var/log/message`) or examine the output from `journalctl -u kubelet`. If you see something like the following:
-->
- ネットワーク接続の問題。作業をする前に、マシンのネットワーク疎通が完全か確認します。
- kubelet 用のデフォルト cgroup ドライバ設定が、Docker で使っているものと異なる場合。システムのログファイル（例： `/var/log/message`）を確認するか、 `journalctl -u kubelet` の出力を調査します。次のようなメッセージが出る場合は


  ```shell
  error: failed to run Kubelet: failed to create kubelet:
  misconfiguration: kubelet cgroup driver: "systemd" is different from docker cgroup driver: "cgroupfs"
  ```
<!--
  There are two common ways to fix the cgroup driver problem:
  -->
  cgroup ドライバ問題を解決するには、２つの共通手法があります。
  
  <!--
 1. Install docker again following instructions
  [here](/docs/setup/independent/install-kubeadm/#installing-docker).
 1. Change the kubelet config to match the Docker cgroup driver manually, you can refer to
    [Configure cgroup driver used by kubelet on Master Node](/docs/setup/independent/install-kubeadm/#configure-cgroup-driver-used-by-kubelet-on-master-node)
    for detailed instructions.
-->
1.   [こちら](/docs/setup/independent/install-kubeadm/#installing-docker) の手順に従い、Docker を再インストールします。
1. Docker cgroup ドライバと kubelet の設定が同じになるよう、手動で設定します。詳細な手順は  [マスタ・ノード上の kubelet が使う cgroup ドライバの設定](/jp/docs/setup/independent/install-kubeadm/#configure-cgroup-driver-used-by-kubelet-on-master-node)

<!--
- control plane Docker containers are crashlooping or hanging. You can check this by running `docker ps` and investigating each container by running `docker logs`.
-->
- コントロール・プレーン Docker コンテナがクラッシュするか固まる場合、 `docker ps` で実行しているかどうかの確認と、各コンテナを `docker logs` で調査します。

<!--
#### kubeadm blocks when removing managed containers
-->
#### 管理用コンテナの削除時、kubeadm が固まる

<!--
The following could happen if Docker halts and does not remove any Kubernetes-managed containers:
-->
Docker の処理が中断し、Kubernetes が管理するコンテナを削除できなくなる可能性があります。

```bash
sudo kubeadm reset
[preflight] Running pre-flight checks
[reset] Stopping the kubelet service
[reset] Unmounting mounted directories in "/var/lib/kubelet"
[reset] Removing kubernetes-managed containers
(block)
```

<!--
A possible solution is to restart the Docker service and then re-run `kubeadm reset`:
-->
解決策としては、Docker サービスを再起動し、再び `kubeadm reset` を実行します。

```bash
sudo systemctl restart docker.service
sudo kubeadm reset
```

<!--
Inspecting the logs for docker may also be useful:
-->
また、docker に対するログの調査も役立つでしょう。

```sh
journalctl -ul docker
```

<!--
#### Pods in `RunContainerError`, `CrashLoopBackOff` or `Error` state
-->
#### ポッドの状態が  `RunContainerError`、`CrashLoopBackOff`、`Error` 

<!--
Right after `kubeadm init` there should not be any pods in these states.
-->
`kubeadm init` 直後であれば、ポッドはこのような状態になりません。

<!--
- If there are pods in one of these states _right after_ `kubeadm init`, please open an
  issue in the kubeadm repo. `coredns` (or `kube-dns`) should be in the `Pending` state
  until you have deployed the network solution.
- If you see Pods in the `RunContainerError`, `CrashLoopBackOff` or `Error` state
  after deploying the network solution and nothing happens to `coredns` (or `kube-dns`),
  it's very likely that the Pod Network solution and nothing happens to the DNS server, it's very
  likely that the Pod Network solution that you installed is somehow broken. You
  might have to grant it more RBAC privileges or use a newer version. Please file
  an issue in the Pod Network providers' issue tracker and get the issue triaged there.
-->
- `kubeadm init` 実行直後、これらの状態のポッドがあるようでしたら、 kubeadm リポジトリで issue を開いてください。ネットワーク・ソリューションを展開する前に、 `coredns` （あるいは `kube-dns`）は `Pending` 状態であるべきです。
- ネットワーク・ソリューションを展開した後、ポッドが `RunContainerError`、`CrashLoopBackOff`、`Error` であり、 `coredns`（あるいは`kube-dns`）で何も起こっていなければ、ポッド・ネットワーク・ソリューションが DNS サーバに対して何もしていない可能性があります。これは、インストールしたポッド・ネットワーク・ソリューションで何か壊れている可能性があります。RBAC 権限を多く付与するか、あるいは、新しいバージョンを使います。ポッド・ネットワーク・プロバイダの issue トラッカーに問題を提出し、issue を取り上げてもらいます。

<!--
#### `coredns` (or `kube-dns`) is stuck in the `Pending` state
-->
`coredns`（か `kube-dns`）が `Pending` 状態のまま固まっています

<!--
This is **expected** and part of the design. kubeadm is network provider-agnostic, so the admin
should [install the pod network solution](/docs/concepts/cluster-administration/addons/)
of choice. You have to install a Pod Network
before CoreDNS may deployed fully. Hence the `Pending` state before the network is set up.
-->
これは **予想されうる** もので、設計の一部です。kubeadm はネットワーク・プロバイダが何であろうと動作します。そのため、管理者が[インストールするポッド・ネットワーク・ソリューション(/jp/docs/concepts/cluster-administration/addons/)を選ぶべきです。CoreDNS を展開する前に、ポッド・ネットワークのインストールを完全に終える必要があります。従って、ネットワークのセットアップ前は `Pending` （保留）の状態となります。

<!--
#### `HostPort` services do not work
-->
#### `HostPort`サービスが動作しません

<!--
The `HostPort` and `HostIP` functionality is available depending on your Pod Network
provider. Please contact the author of the Pod Network solution to find out whether
`HostPort` and `HostIP` functionality are available.
-->
`HostPort` と `HostIP`機能が利用できるかどうかは、ポッド・ネットワーク・プロバイダに依存します。ポッド・ネットワーク・ソリューションの開発者に連絡を取り、 `HostPort` と `HostIP`機能が利用可能かどうかご確認ください。

<!--
Calico, Canal, and Flannel CNI providers are verified to support HostPort.
-->
Calico、Canal、Flannel CNI プロバイダは HostProt のサポートが確認されています。

<!--
For more information, see the [CNI portmap documentation](https://github.com/containernetworking/plugins/blob/master/plugins/meta/portmap/README.md).
-->
詳しい情報は [CNI ポートマップ・ドキュメント](https://github.com/containernetworking/plugins/blob/master/plugins/meta/portmap/README.md) をご覧ください。

<!--
If your network provider does not support the portmap CNI plugin, you may need to use the [NodePort feature of
services](/docs/concepts/services-networking/service/#type-nodeport) or use `HostNetwork=true`.
-->
ネットワーク・プロバイダが CNI プラグインのポートマップ（portmap）に対応していなければ、[サービスのノードポート（ NodePort）機能](/jp/docs/concepts/services-networking/service/#type-nodeport) を使うか、あるいは、 `HostNetwork=true` を使う必要があります。

<!--
#### Pods are not accessible via their Service IP
-->
#### ポッドにサービス IP を通して接続できません

<!--
- Many network add-ons do not yet enable [hairpin mode](https://kubernetes.io/docs/tasks/debug-application-cluster/debug-service/#a-pod-cannot-reach-itself-via-service-ip)
  which allows pods to access themselves via their Service IP. This is an issue related to
  [CNI](https://github.com/containernetworking/cni/issues/476). Please contact the network
  add-on provider to get the latest status of their support for hairpin mode.
-->
- 多くのネットワーク・アドオンは、まだ [ヘアピン・モード(hairpin mode](/jp/docs/tasks/debug-application-cluster/debug-service/#a-pod-cannot-reach-itself-via-service-ip) が有効ではありません。このモードは、ポットに対してサービス IP を通してアクセス可能にするものです。これは [CNI](https://github.com/containernetworking/cni/issues/476) に間rnenする問題です。ネットワーク・アドオン・プロバイダに対して、ヘアピン・モードの最新のサポート状況についてお訊ねください。

<!--
- If you are using VirtualBox (directly or via Vagrant), you will need to
  ensure that `hostname -i` returns a routable IP address. By default the first
  interface is connected to a non-routable host-only network. A work around
  is to modify `/etc/hosts`, see this [Vagrantfile](https://github.com/errordeveloper/k8s-playground/blob/22dd39dfc06111235620e6c4404a96ae146f26fd/Vagrantfile#L11)
  for an example.
-->
- VirtualBox を使用している（直接、あるいは Vagrant を経由している）場合、 `hostname -i` が到達可能な IP アドレスを返す必要があります。１つめのインターフェースは、デフォルトでは到達できないホストのみのネットワークに接続します。動作するように `/etc/hosts`の対応が必要ですので、 こちらの [Vagrantfile](https://github.com/errordeveloper/k8s-playground/blob/22dd39dfc06111235620e6c4404a96ae146f26fd/Vagrantfile#L11) 例をご覧ください。

<!--
#### TLS certificate errors
-->
#### TLS 認証エラー


<!--
The following error indicates a possible certificate mismatch.
-->

以下のエラーは、証明書の不一致がある可能性を示します。

```none
# kubectl get pods
Unable to connect to the server: x509: certificate signed by unknown authority (possibly because of "crypto/rsa: verification error" while trying to verify candidate authority certificate "kubernetes")
```

<!--
- Verify that the `$HOME/.kube/config` file contains a valid certificate, and
  regenerate a certificate if necessary. The certificates in a kubeconfig file
  are base64 encoded. The `base64 -d` command can be used to decode the certificate
  and `openssl x509 -text -noout` can be used for viewing the certificate information.
- Another workaround is to overwrite the existing `kubeconfig` for the "admin" user:
-->
-  `$HOME/.kube/config` ファイルに有効な証明書があるかどうかを確認し、必要に応じて証明書を再作成します。kubeconfig ファイル内の証明書は base64 でエンコード（符号化）されています。 `base64 -d`コマンドを使えば証明書をデコード（復号化）できます。また、証明書の情報を表示するために `openssl x509 -text -noout` を使えます。
- この方法でうまくいかない時の対処方法は、既存の `admin` ユーザ向け  `kubeconfig` の上書きです。

  ```sh
  mv  $HOME/.kube $HOME/.kube.bak
  sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
  sudo chown $(id -u):$(id -g) $HOME/.kube/config
  ```

<!--
#### Default NIC When using flannel as the pod network in Vagrant
-->
#### Vagrant でポッド・ネットワークに flannel 使用時のデフォルト NIC

<!--
The following error might indicate that something was wrong in the pod network:
-->
以下のエラーは、ポッド・ネットワーク内で何らかの問題があるのを示します。

```sh
Error from server (NotFound): the server could not find the requested resource
```

<!--
- If you're using flannel as the pod network inside Vagrant, then you will have to specify the default interface name for flannel.

  Vagrant typically assigns two interfaces to all VMs. The first, for which all hosts are assigned the IP address `10.0.2.15`, is for external traffic that gets NATed.

  This may lead to problems with flannel, which defaults to the first interface on a host. This leads to all hosts thinking they have the same public IP address. To prevent this, pass the `--iface eth1` flag to flannel so that the second interface is chosen.
-->
- Vagrant 内でポッド・ネットワークを flannel として使用している場合、flannel 用のデフォルト・インターフェース名を指定する必要があります。

通常の Vagrant は、全ての仮想マシンに対して２つのインターフェースを割り当てます。１つは全てのホストが割り当てられる IP アドレス `10.0.2.15` であり、外部に対するトラフィックを NAT （ネットワークのアドレスを変換）します。

ホスト上の１つめのインターフェースがデフォルトなため、 flannel では問題を引き起こします。全てのホストが共通のパブリック IP アドレスを持ってしまうのが予想されます。回避するには flannel に対して `--iface eth1` フラグを与え、２つめのインターフェースを選択します。

<!--
#### Non-public IP used for containers
-->
#### コンテナに対してパブリックではない IP アドレスが割り当て

<!--
In some situations `kubectl logs` and `kubectl run` commands may return with the following errors in an otherwise functional cluster:
-->
`kubectl logs` と `kubectl run`コマンドを実行するような状況で、他のクラスタ機能に関するエラーが返ってくる場合があります。

```sh
Error from server: Get https://10.19.0.41:10250/containerLogs/default/mysql-ddc65b868-glc5m/mysql: dial tcp 10.19.0.41:10250: getsockopt: no route to host
```

<!--
- This may be due to Kubernetes using an IP that can not communicate with other IPs on the seemingly same subnet, possibly by policy of the machine provider.
- Digital Ocean assigns a public IP to `eth0` as well as a private one to be used internally as anchor for their floating IP feature, yet `kubelet` will pick the latter as the node's `InternalIP` instead of the public one.
-->
- このようになるのは、Kubernetes が使用する IP アドレスが同じサブネット上の他の IP と通信できないため、マシン提供事業者のポリシーによる影響の可能性があります。
- Digital Ocean はパブリック IP を `eth0` に割り当てるのと当時に、内部におけるフローティング IP 機能の到達先としても用いています。さらに、 `kubelet` はパブリック IP ではなく、後に続く `InternelIP`（内部IP）を取り上げてしまいます。

<!--
  Use `ip addr show` to check for this scenario instead of `ifconfig` because `ifconfig` will not display the offending alias IP address. Alternatively an API endpoint specific to Digital Ocean allows to query for the anchor IP from the droplet:
ｰｰ>
この状況を調べるには `ifconfig` ではなく `ip addr show`を使います。 `ifconfig` は IP アドレスのエイリアス（別名）を表示できないからです。あるいは Digital Ocean の API エンドポイントで、ドロップレット用のアンカー IP を問い合わせできます。

  ```sh
  curl http://169.254.169.254/metadata/v1/interfaces/public/0/anchor_ipv4/address
  ```

<!--
  The workaround is to tell `kubelet` which IP to use using `--node-ip`. When using Digital Ocean, it can be the public one (assigned to `eth0`) or the private one (assigned to `eth1`) should you want to use the optional private network. The [KubeletExtraArgs section of the MasterConfiguration file](https://github.com/kubernetes/kubernetes/blob/master/cmd/kubeadm/app/apis/kubeadm/v1alpha2/types.go#L147) can be used for this.
-->
他の回避策は、 `kubelet` に `--node-ip` を指定して、どの IP アドレスを使うかを伝える方法です。Digital Ocean では、公開 IP（`eth0` に割り当て）かプライベート IP （`eth1` に割り当て）のうち、どちらをプライベート・ネットワークとして使用するかのオプションで指定すべきでしょう。 [マスタの設定ファイルにある KubeletExtraArgs セクション](https://github.com/kubernetes/kubernetes/blob/master/cmd/kubeadm/app/apis/kubeadm/v1alpha2/types.go#L147) が対応しています。

<!--
  Then restart `kubelet`:
-->
それから `kubelet` を再開します。

  ```sh
  systemctl daemon-reload
  systemctl restart kubelet
  ```
<!--
#### Services with externalTrafficPolicy=Local are not reachable
-->
#### externalTrafficPolicy=Local のサービスに到達不可能

<!--
On nodes where the hostname for the kubelet is overridden using the `--hostname-override` option, kube-proxy will default to treating 127.0.0.1 as the node IP, which results in rejecting connections for Services configured for `externalTrafficPolicy=Local`. This situation can be verified by checking the output of `kubectl -n kube-system logs <kube-proxy pod name>`:
-->
kubelet を使うノード上のホスト名は `--hostname-override`（ホスト名上書き）オプションを使って上書きできます。kube-proxy はデフォルトで `127.0.0.1` をノードの IP として 扱うため、サービス設定を `externalTrafficPolicy=Local` としている場合、接続が拒否される結果になります。この状況では `kubectl -n kube-system logs <kube-proxy ポッド名>` の出力から確認可能です。

```sh
W0507 22:33:10.372369       1 server.go:586] Failed to retrieve node info: nodes "ip-10-0-23-78" not found
W0507 22:33:10.372474       1 proxier.go:463] invalid nodeIP, initializing kube-proxy with 127.0.0.1 as nodeIP
```

<!--
A workaround for this is to modify the kube-proxy DaemonSet in the following way:
-->
この次善策としては、kube-proxy の DaemonSet  を以下のように変更します。


```sh
kubectl -n kube-system patch --type json daemonset kube-proxy -p "$(cat <<'EOF'
[
    {
        "op": "add",
        "path": "/spec/template/spec/containers/0/env",
        "value": [
            {
                "name": "NODE_NAME",
                "valueFrom": {
                    "fieldRef": {
                        "apiVersion": "v1",
                        "fieldPath": "spec.nodeName"
                    }
                }
            }
        ]
    },
    {
        "op": "add",
        "path": "/spec/template/spec/containers/0/command/-",
        "value": "--hostname-override=${NODE_NAME}"
    }
]
EOF
)"

```
