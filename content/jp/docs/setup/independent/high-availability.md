---
reviewers:
- sig-cluster-lifecycle
title: kubeadm で高可用性クラスタを作成
content_template: templates/task
weight: 60
---

{{% capture overview %}}

この手順書では、高可用性 Kubernetes クラスタを kubeadm でセットアップするための２つの方法を説明します。

- マスタ群を使う。この方法は必要なインフラが少ないです。etcd メンバとコントロール・プレーンが同じ場所です。
- 外部にある etd クラスタを使う。この方法は多くのインフラが必要です。コントロール・プレーンのノードと etcd メンバは別々です。

クラスタでは Kubernetes バージョン 1.11 以上を動かす必要があります。また、kubeadm で HA クラスタをセットアップするのは、まだ実験的ですのでご注意ください。そのため、クラスタのアップグレード時に問題が出るかもしれません。どちらか一方の手法を試して、フィードバックをください。

{{< caution >}}
**注意**: このページではクラウド事業者上でクラスタを実行する方法を扱いません。おける負荷分散や永続的領域（PersistentVolume）といった、クラウド環境におけるサービス種別を利用するドキュメントはありません。
{{< /caution >}}

{{% /capture %}}

{{% capture prerequisites %}}

いずれの手法でも、次のインフラが必要です：

- [kubeadm 動作条件](/jp/docs/tasks/tools/install-kubeadm/#before-you-begin) を持つマシンで３台のマスタを構成
- [kubeadm 動作条件](/jp/docs/tasks/tools/install-kubeadm/#before-you-begin) を持つマシンで３台のワーカを構成
- クラスタ内の全てのマシンに対する、完全なネットワーク接続性（パブリックまたはプライベート・ネットワークどちらでも構わない）
- システム上の全てのノードに SSH 接続できる手段
- 全てのマシン上で sudo 特権

外部 etcd クラスタを使う場合のみ、以下も必要です：

- etcd メンバ用に３台の追加マシン

{{< note >}}
**メモ**: 以下の例は Pod ネットワーク・プロバイダとして Calico を使用します。もしも他のネットワーク・プロバイダを動作する場合は、必要に応じて初期値を置き換えてください。
{{< /note >}}

{{% /capture %}}

{{% capture steps %}}

## 初めにする共通の手順

{{< note >}}
**メモ**: このガイドにおけるコントロール・プレーンや etcd ノード上でのコマンドは、root として実行すべきです。
{{< /note >}}

- ポッド CIDR を確認します。詳細については [CNI ネットワーク資料](/jp/docs/setup/independent/create-cluster-kubeadm/#pod-network) をご覧ください。Calico を使う場合、ポッド CIDR は `192.168.0.0/16` です。

### SSH の設定

1.  メイン・デバイス上（訳者注：PCやサーバなど、作業用の環境）で ssh-agent を有効にします。これでシステム内の他のノードすべてにアクセスします。

     ```
     eval $(ssh-agent)
     ```

1.  セッションに自分の SSH 秘密鍵を追加します。

     ```
     ssh-add ~/.ssh/path_to_private_key
     ```

1.  ノード間の SSH 接続が正常に行えるかを確認します。

    - ノードに SSH するときは、 `-A` フラグを確認します。

      ```
      ssh -A 10.0.0.7
      ```

    - ノードで sudo を実行するときは、SSH 転送（forwarding）で持続するようにします。

      ```
      sudo -E -s
      ```

### kube-apiserver 用のロードバランサを作成

{{< note >}}
**メモ**: ロードバランサには多くの設定方法があります。以下の例はそのうちの１つです。クラスタの要件によっては、別の設定が必要になるでしょう。
{{< /note >}}

1.  DNS で名前解決する kube-apiserver ロードバランサを作成します。

    - クラウド環境では、TCP 転送ロードバランサの派小河にコントロール・プレーン・ノードを配置すべきでしょう。ロードバランサは、コントロール・プレーン・ノードの一覧から、正常なノードにトラフィックを分散します。apiserver に対するヘルスチェックとは、kube-apiserver が開く（listen）ポート（初期値： `6443`）に対する TCP 確認です。

    - クラウド環境では、 IP アドレスを直接使う方法は推奨しません。

    - ロードバランサは全てのコントロール・プレーン・ノード上の apiserver ポートと通信できる必要があります。また、開いている（listening）ポートに対する受信トラフィック（incoming traffic）の許可も必要です。

1. １つめのコントロール・プレーン・ノードをロードバランサに追加し、接続を確認します。

    ```sh
    nc -v LOAD_BALANCER_IP PORT
    ```

    - 接続拒否エラーが出た場合、予想されるのは apiserver がまだ実行していない場合です。タイムアウトとは、ロードバランサがコントロール・プレーン・ノードと通信できないのを意味します。もしもタイムアウトが発生したら、ロードバランサがコントロール・プレーン・ノードと通信できるように再設定します。

1.  残りのコントロール・プレーン・ノードもロードバランサの対象グループに追加します。

## 積み重なる（スタックした）コントロール・プレーン・ノード

### １台めのスタック用コントロール・プレーン・ノードを準備

1.  `kubeadm-config.yaml` テンプレート・ファイルを作成します:

        apiVersion: kubeadm.k8s.io/v1alpha2
        kind: MasterConfiguration
        kubernetesVersion: v1.11.0
        apiServerCertSANs:
        - "LOAD_BALANCER_DNS"
        api:
            controlPlaneEndpoint: "LOAD_BALANCER_DNS:LOAD_BALANCER_PORT"
        etcd:
          local:
            extraArgs:
              listen-client-urls: "https://127.0.0.1:2379,https://CP0_IP:2379"
              advertise-client-urls: "https://CP0_IP:2379"
              listen-peer-urls: "https://CP0_IP:2380"
              initial-advertise-peer-urls: "https://CP0_IP:2380"
              initial-cluster: "CP0_HOSTNAME=https://CP0_IP:2380"
            serverCertSANs:
              - CP0_HOSTNAME
              - CP0_IP
            peerCertSANs:
              - CP0_HOSTNAME
              - CP0_IP
        networking:
            # This CIDR is a Calico default. Substitute or remove for your CNI provider.
            podSubnet: "192.168.0.0/16"


1.  テンプレートにある以下の変数を、自分のクラスタにあわせた適切な値に置き換えます。

    * `LOAD_BALANCER_DNS`
    * `LOAD_BALANCER_PORT`
    * `CP0_HOSTNAME`
    * `CP0_IP`

1.  `kubeadm init --config kubeadm-config.yaml`を実行します。

### 必要なファイルを他のコントロール・プレーン・ノードにコピー

`kubeadm init` を実行したら、以下の証明書や他に必要なファイルが作成されます。各ファイルを他のコントロール・プレーン・ノードにコピーします。

- `/etc/kubernetes/pki/ca.crt`
- `/etc/kubernetes/pki/ca.key`
- `/etc/kubernetes/pki/sa.key`
- `/etc/kubernetes/pki/sa.pub`
- `/etc/kubernetes/pki/front-proxy-ca.crt`
- `/etc/kubernetes/pki/front-proxy-ca.key`
- `/etc/kubernetes/pki/etcd/ca.crt`
- `/etc/kubernetes/pki/etcd/ca.key`

admin kubeconfig（管理用設定ファイル）を他のコントロール・プレーン・ノードにコピーします。

- `/etc/kubernetes/admin.conf`

以下の例にある  `CONTROL_PLANE_IPS` にある IP アドレスは、各コントロール・プレーン・ノードのものに書き換えます。

```sh
USER=ubuntu # customizable
CONTROL_PLANE_IPS="10.0.0.7 10.0.0.8"
for host in ${CONTROL_PLANE_IPS}; do
    scp /etc/kubernetes/pki/ca.crt "${USER}"@$host:
    scp /etc/kubernetes/pki/ca.key "${USER}"@$host:
    scp /etc/kubernetes/pki/sa.key "${USER}"@$host:
    scp /etc/kubernetes/pki/sa.pub "${USER}"@$host:
    scp /etc/kubernetes/pki/front-proxy-ca.crt "${USER}"@$host:
    scp /etc/kubernetes/pki/front-proxy-ca.key "${USER}"@$host:
    scp /etc/kubernetes/pki/etcd/ca.crt "${USER}"@$host:etcd-ca.crt
    scp /etc/kubernetes/pki/etcd/ca.key "${USER}"@$host:etcd-ca.key
    scp /etc/kubernetes/admin.conf "${USER}"@$host:
done
```

{{< note >}}
**メモ**: この例と皆さんの設定ファイルとは異なる場合がありますので、ご注意ください。
{{< /note >}}

### ２台めのスタック用コントロール・プレーン・ノードを追加

1.  2台めの作成にあたり、異なる `kubeadm-config.yaml` テンプレート・ファイルを作ります。

        apiVersion: kubeadm.k8s.io/v1alpha2
        kind: MasterConfiguration
        kubernetesVersion: v1.11.0
        apiServerCertSANs:
        - "LOAD_BALANCER_DNS"
        api:
            controlPlaneEndpoint: "LOAD_BALANCER_DNS:LOAD_BALANCER_PORT"
        etcd:
          local:
            extraArgs:
              listen-client-urls: "https://127.0.0.1:2379,https://CP1_IP:2379"
              advertise-client-urls: "https://CP1_IP:2379"
              listen-peer-urls: "https://CP1_IP:2380"
              initial-advertise-peer-urls: "https://CP1_IP:2380"
              initial-cluster: "CP0_HOSTNAME=https://CP0_IP:2380,CP1_HOSTNAME=https://CP1_IP:2380"
              initial-cluster-state: existing
            serverCertSANs:
              - CP1_HOSTNAME
              - CP1_IP
            peerCertSANs:
              - CP1_HOSTNAME
              - CP1_IP
        networking:
            # This CIDR is a calico default. Substitute or remove for your CNI provider.
            podSubnet: "192.168.0.0/16"

1.  テンプレートにある以下の変数を、自分のクラスタにあわせた適切な値に置き換えます。

    - `LOAD_BALANCER_DNS`
    - `LOAD_BALANCER_PORT`
    - `CP0_HOSTNAME`
    - `CP0_IP`
    - `CP1_HOSTNAME`
    - `CP1_IP`

1.  コピーしたファイルを適切な場所に移動します。

      ```sh
      USER=ubuntu # customizable
      mkdir -p /etc/kubernetes/pki/etcd
      mv /home/${USER}/ca.crt /etc/kubernetes/pki/
      mv /home/${USER}/ca.key /etc/kubernetes/pki/
      mv /home/${USER}/sa.pub /etc/kubernetes/pki/
      mv /home/${USER}/sa.key /etc/kubernetes/pki/
      mv /home/${USER}/front-proxy-ca.crt /etc/kubernetes/pki/
      mv /home/${USER}/front-proxy-ca.key /etc/kubernetes/pki/
      mv /home/${USER}/etcd-ca.crt /etc/kubernetes/pki/etcd/ca.crt
      mv /home/${USER}/etcd-ca.key /etc/kubernetes/pki/etcd/ca.key
      mv /home/${USER}/admin.conf /etc/kubernetes/admin.conf
      ```

1.  kubeadm phase コマンドを実行し、 kubelet を立ち上げます。

      ```sh
      kubeadm alpha phase certs all --config kubeadm-config.yaml
      kubeadm alpha phase kubelet config write-to-disk --config kubeadm-config.yaml
      kubeadm alpha phase kubelet write-env-file --config kubeadm-config.yaml
      kubeadm alpha phase kubeconfig kubelet --config kubeadm-config.yaml
      systemctl start kubelet
      ```

1.  ノードを etcd クラスタに追加するため、以下のコマンドを実行します。

      ```sh
      CP0_IP=10.0.0.7
      CP0_HOSTNAME=cp0
      CP1_IP=10.0.0.8
      CP1_HOSTNAME=cp1

      KUBECONFIG=/etc/kubernetes/admin.conf kubectl exec -n kube-system etcd-${CP0_HOSTNAME} -- etcdctl --ca-file /etc/kubernetes/pki/etcd/ca.crt --cert-file /etc/kubernetes/pki/etcd/peer.crt --key-file /etc/kubernetes/pki/etcd/peer.key --endpoints=https://${CP0_IP}:2379 member add ${CP1_HOSTNAME} https://${CP1_IP}:2380
      kubeadm alpha phase etcd local --config kubeadm-config.yaml
      ```

      - こちらのコマンドの実行で、etcd クラスタは短時間利用できなくなります。これは、ノードが稼働中のクラスタに追加後、新しいノードが etcd クラスタに参加するまでです。

1.  コントロール・プレーン構成要素を展開し、ノードをマスタとして記録（マーク）します。

      ```sh
      kubeadm alpha phase kubeconfig all --config kubeadm-config.yaml
      kubeadm alpha phase controlplane all --config kubeadm-config.yaml
      kubeadm alpha phase mark-master --config kubeadm-config.yaml
      ```

### ３台めのスタック用コントロール・プレーン・ノードを追加

1.  ３台めの作成にあたり、異なる `kubeadm-config.yaml` テンプレート・ファイルを作ります。

        apiVersion: kubeadm.k8s.io/v1alpha2
        kind: MasterConfiguration
        kubernetesVersion: v1.11.0
        apiServerCertSANs:
        - "LOAD_BALANCER_DNS"
        api:
            controlPlaneEndpoint: "LOAD_BALANCER_DNS:LOAD_BALANCER_PORT"
        etcd:
          local:
            extraArgs:
              listen-client-urls: "https://127.0.0.1:2379,https://CP2_IP:2379"
              advertise-client-urls: "https://CP2_IP:2379"
              listen-peer-urls: "https://CP2_IP:2380"
              initial-advertise-peer-urls: "https://CP2_IP:2380"
              initial-cluster: "CP0_HOSTNAME=https://CP0_IP:2380,CP1_HOSTNAME=https://CP1_IP:2380,CP2_HOSTNAME=https://CP2_IP:2380"
              initial-cluster-state: existing
            serverCertSANs:
              - CP2_HOSTNAME
              - CP2_IP
            peerCertSANs:
              - CP2_HOSTNAME
              - CP2_IP
        networking:
            # This CIDR is a calico default. Substitute or remove for your CNI provider.
            podSubnet: "192.168.0.0/16"

1.  テンプレートにある以下の変数を、自分のクラスタにあわせた適切な値に置き換えます。

    - `LOAD_BALANCER_DNS`
    - `LOAD_BALANCER_PORT`
    - `CP0_HOSTNAME`
    - `CP0_IP`
    - `CP1_HOSTNAME`
    - `CP1_IP`
    - `CP2_HOSTNAME`
    - `CP2_IP`

1.  コピーしたファイルを適切な場所に移動します。

      ```sh
      USER=ubuntu # customizable
      mkdir -p /etc/kubernetes/pki/etcd
      mv /home/${USER}/ca.crt /etc/kubernetes/pki/
      mv /home/${USER}/ca.key /etc/kubernetes/pki/
      mv /home/${USER}/sa.pub /etc/kubernetes/pki/
      mv /home/${USER}/sa.key /etc/kubernetes/pki/
      mv /home/${USER}/front-proxy-ca.crt /etc/kubernetes/pki/
      mv /home/${USER}/front-proxy-ca.key /etc/kubernetes/pki/
      mv /home/${USER}/etcd-ca.crt /etc/kubernetes/pki/etcd/ca.crt
      mv /home/${USER}/etcd-ca.key /etc/kubernetes/pki/etcd/ca.key
      mv /home/${USER}/admin.conf /etc/kubernetes/admin.conf
      ```

1.  kubeadm phase コマンドを実行し、 kubelet を立ち上げます。

      ```sh
      kubeadm alpha phase certs all --config kubeadm-config.yaml
      kubeadm alpha phase kubelet config write-to-disk --config kubeadm-config.yaml
      kubeadm alpha phase kubelet write-env-file --config kubeadm-config.yaml
      kubeadm alpha phase kubeconfig kubelet --config kubeadm-config.yaml
      systemctl start kubelet
      ```

1.  ノードを etcd クラスタに追加するため、以下のコマンドを実行します。

      ```sh
      CP0_IP=10.0.0.7
      CP0_HOSTNAME=cp0
      CP2_IP=10.0.0.9
      CP2_HOSTNAME=cp2

      KUBECONFIG=/etc/kubernetes/admin.conf kubectl exec -n kube-system etcd-${CP0_HOSTNAME} -- etcdctl --ca-file /etc/kubernetes/pki/etcd/ca.crt --cert-file /etc/kubernetes/pki/etcd/peer.crt --key-file /etc/kubernetes/pki/etcd/peer.key --endpoints=https://${CP0_IP}:2379 member add ${CP2_HOSTNAME} https://${CP2_IP}:2380
      kubeadm alpha phase etcd local --config kubeadm-config.yaml
      ```

1.  コントロール・プレーン構成要素を展開し、ノードをマスタとして記録（マーク）します。

      ```sh
      kubeadm alpha phase kubeconfig all --config kubeadm-config.yaml
      kubeadm alpha phase controlplane all --config kubeadm-config.yaml
      kubeadm alpha phase mark-master --config kubeadm-config.yaml
      ```

## 外部 etcd

### クラスタのセットアップ

- [これらの手順](/jp/docs/tasks/administer-cluster/setup-ha-etcd-with-kubeadm/)に従い、 etcd クラスタをセットアップします。

### 必要なファイルを他のコントロール・プレーン・ノードにコピー

クラスタ作成時、以下の証明書が作成されます。それぞれ他のコントロール・プレーン・ノードにコピーします。

- `/etc/kubernetes/pki/etcd/ca.crt`
- `/etc/kubernetes/pki/apiserver-etcd-client.crt`
- `/etc/kubernetes/pki/apiserver-etcd-client.key`

以下の例にある `USER` と `CONTROL_PLANE_HOSTS` の値は環境にあわせて置き換える必要があります。

```sh
USER=ubuntu
CONTROL_PLANE_HOSTS="10.0.0.7 10.0.0.8 10.0.0.9"
for host in $CONTROL_PLANE_HOSTS; do
    scp /etc/kubernetes/pki/etcd/ca.crt "${USER}"@$host:
    scp /etc/kubernetes/pki/apiserver-etcd-client.crt "${USER}"@$host:
    scp /etc/kubernetes/pki/apiserver-etcd-client.key "${USER}"@$host:
done
```

### １台めのコントロール・プレーン・ノードをセットアップ

1.  `kubeadm-config.yaml` テンプレート・ファイルを作成します。

        apiVersion: kubeadm.k8s.io/v1alpha2
        kind: MasterConfiguration
        kubernetesVersion: v1.11.0
        apiServerCertSANs:
        - "LOAD_BALANCER_DNS"
        api:
            controlPlaneEndpoint: "LOAD_BALANCER_DNS:LOAD_BALANCER_PORT"
        etcd:
            external:
                endpoints:
                - https://ETCD_0_IP:2379
                - https://ETCD_1_IP:2379
                - https://ETCD_2_IP:2379
                caFile: /etc/kubernetes/pki/etcd/ca.crt
                certFile: /etc/kubernetes/pki/apiserver-etcd-client.crt
                keyFile: /etc/kubernetes/pki/apiserver-etcd-client.key
        networking:
            # This CIDR is a calico default. Substitute or remove for your CNI provider.
            podSubnet: "192.168.0.0/16"

1.  テンプレートにある以下の変数を、自分のクラスタにあわせた適切な値に置き換えます。

    - `LOAD_BALANCER_DNS`
    - `LOAD_BALANCER_PORT`
    - `ETCD_0_IP`
    - `ETCD_1_IP`
    - `ETCD_2_IP`

1.  `kubeadm init --config kubeadm-config.yaml` を実行します。

### 必要なファイルを適切な場所にコピーします。


`kubeadm init` を実行したら、以下の証明書や他に必要なファイルが作成されます。各ファイルを他のコントロール・プレーン・ノードにコピーします。

- `/etc/kubernetes/pki/ca.crt`
- `/etc/kubernetes/pki/ca.key`
- `/etc/kubernetes/pki/sa.key`
- `/etc/kubernetes/pki/sa.pub`
- `/etc/kubernetes/pki/front-proxy-ca.crt`
- `/etc/kubernetes/pki/front-proxy-ca.key`

以下の例にある  `CONTROL_PLANE_IPS` にある IP アドレスは、各コントロール・プレーン・ノードのものに書き換えます。

```sh
USER=ubuntu # customizable
CONTROL_PLANE_IPS="10.0.0.7 10.0.0.8"
for host in ${CONTROL_PLANE_IPS}; do
    scp /etc/kubernetes/pki/ca.crt "${USER}"@CONTROL_PLANE_IP:
    scp /etc/kubernetes/pki/ca.key "${USER}"@CONTROL_PLANE_IP:
    scp /etc/kubernetes/pki/sa.key "${USER}"@CONTROL_PLANE_IP:
    scp /etc/kubernetes/pki/sa.pub "${USER}"@CONTROL_PLANE_IP:
    scp /etc/kubernetes/pki/front-proxy-ca.crt "${USER}"@CONTROL_PLANE_IP:
    scp /etc/kubernetes/pki/front-proxy-ca.key "${USER}"@CONTROL_PLANE_IP:
done
```

{{< note >}}
**メモ**: この例と皆さんの設定ファイルとは異なる場合がありますので、ご注意ください。
{{< /note >}}

### 他のコントロール・プレーン・ノードをセットアップ

1.  コピーしたファイルの場所を確認します。`/etc/kubernetes` ディレクトリは、このようになっています。

    - `/etc/kubernetes/pki/apiserver-etcd-client.crt`
    - `/etc/kubernetes/pki/apiserver-etcd-client.key`
    - `/etc/kubernetes/pki/ca.crt`
    - `/etc/kubernetes/pki/ca.key`
    - `/etc/kubernetes/pki/front-proxy-ca.crt`
    - `/etc/kubernetes/pki/front-proxy-ca.key`
    - `/etc/kubernetes/pki/sa.key`
    - `/etc/kubernetes/pki/sa.pub`
    - `/etc/kubernetes/pki/etcd/ca.crt`

1.  各コントロール・プレーン・ノードの既に作成した `kubeadm-config.yaml` がある場所で `kubeadm init --config kubeadm-config.yaml` を実行します。

## コントロール・プレーンを立ち上げた後の共通作業

### ポッド・ネットワークのインストール

[こちらの手順に従い](/jp/docs/setup/independent/create-cluster-kubeadm/#pod-network)、ポッド・ネットワークをインストールします。マスタの設定ファイルで指定したポッド CIDR と一致するようにします。

### ワーカのインストール

各ワーカ・ノードをクラスタに追加するには、`kubeadm init` コマンドの表示結果をコマンドとして実行します。

{{% /capture %}}
