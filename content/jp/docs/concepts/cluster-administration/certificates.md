---
title: 証明書（Certificates）
content_template: templates/concept
weight: 20
---


{{% capture overview %}}

<!--
When using client certificate authentication, you can generate certificates
manually through `easyrsa`, `openssl` or `cfssl`.
-->
クライアント証明書認証を使う場合は、証明書を `easyrsa` 、 `openssl` 、 `cfssl`  を通して手動で作成できます。

{{% /capture %}}

{{< toc >}}

{{% capture body %}}

### easyrsa

<!--
**easyrsa** can manually generate certificates for your cluster.
-->
**easyrsa**  はクラスタ用の証明書を自動的に生成できます。

<!--1.  Download, unpack, and initialize the patched version of easyrsa3.-->1. easyrsa3 のパッチ済みバージョンをダウンロード、展開、初期化します。

        curl -LO https://storage.googleapis.com/kubernetes-release/easy-rsa/easy-rsa.tar.gz
        tar xzf easy-rsa.tar.gz
        cd easy-rsa-master/easyrsa3
        ./easyrsa init-pki
<!--1.  Generate a CA. (`--batch` set automatic mode. `--req-cn` default CN to use.)-->2. CA（証明機関）を作成します。（ `--batch` を設定すると自動モード、 `--req-cn` はデフォルトの CN を使います ）

        ./easyrsa --batch "--req-cn=${MASTER_IP}@`date +%s`" build-ca nopass
<!--1.  Generate server certificate and key.
    The argument `--subject-alt-name` sets the possible IPs and DNS names the API server will
    be accessed with. The `MASTER_CLUSTER_IP` is usually the first IP from the service CIDR
    that is specified as the `--service-cluster-ip-range` argument for both the API server and
    the controller manager component. The argument `--days` is used to set the number of days
    after which the certificate expires.
    The sample below also assume that you are using `cluster.local` as the default
    DNS domain name.-->3. サーバ証明書と鍵を作成します。引数 `--subject-alt-name`  （別名の受け入れ）で、 API サーバになる可能性のある IP アドレスと DNS 名を設定します。 `MASTER_CLUSTER_IP` （マスタのクラスタ IP アドレス）には、通常はサービス CIDR にある最初の IP アドレスです。API サーバとコントローラ・マネージャ（構成）部品の両方のクラスタ範囲を、サービス CIDR の `--service-cluster-ip-range` 引数によって 指定します。引数 `--days` （日数）は、証明書の有効期限が切れてからの日数を指定します。また、以下の例では `cluster.local` をデフォルトの DNS ドメイン名とみなしています。
   
        ./easyrsa --subject-alt-name="IP:${MASTER_IP},"\
        "IP:${MASTER_CLUSTER_IP},"\
        "DNS:kubernetes,"\
        "DNS:kubernetes.default,"\
        "DNS:kubernetes.default.svc,"\
        "DNS:kubernetes.default.svc.cluster,"\
        "DNS:kubernetes.default.svc.cluster.local" \
        --days=10000 \
        build-server-full server nopass
<!--1.  Copy `pki/ca.crt`, `pki/issued/server.crt`, and `pki/private/server.key` to your directory.-->4. `pki/ca.crt`、`pki/issued/server.crt`、 `pki/private/server.key` を自分のディレクトリにコピーします。

<!--1.  Fill in and add the following parameters into the API server start parameters:　-->5. API サーバの起動パラメータ中に、以下のパラメータを埋めて追加します。

        --client-ca-file=/yourdirectory/ca.crt
        --tls-cert-file=/yourdirectory/server.crt
        --tls-private-key-file=/yourdirectory/server.key

### openssl

<!--
**openssl** can manually generate certificates for your cluster.
-->
**openssl** は自分のクラスタ用の証明書を手動で作成できます。

<!--1.  Generate a ca.key with 2048bit:-->1. 2048 ビットの ca.key （認証機関の鍵）を作成します。

        openssl genrsa -out ca.key 2048
<!--1.  According to the ca.key generate a ca.crt (use -days to set the certificate effective time):--> 2. ca.key に従って ca.cer（認証機関の証明書）を作成します（-days を使い、証明書の有効期間を設定します。）。

        openssl req -x509 -new -nodes -key ca.key -subj "/CN=${MASTER_IP}" -days 10000 -out ca.crt
<!--1.  Generate a server.key with 2048bit:-->3. 2048 ビットの server.key （サーバ鍵）を作成します。

        openssl genrsa -out server.key 2048
<!--1.  Create a config file for generating a Certificate Signing Request (CSR).
    Be sure to substitute the values marked with angle brackets (e.g. `<MASTER_IP>`)
    with real values before saving this to a file (e.g. `csr.conf`).
    Note that the value for `MASTER_CLUSTER_IP` is the service cluster IP for the
    API server as described in previous subsection.
    The sample below also assume that you are using `cluster.local` as the default
    DNS domain name.-->4. 証明書署名リクエスト（CSR）のための設定ファイルを作成します。カギ括弧（例： `<MASTER_IP>`）記号の場所は、ファイル（例： `csr.conf` ）に保存する前に、確実に値を入力します。前のサブセクションで説明したように、 `MASTER_CLUSTER_IP` （マスタのクラスタ IP）とは API サーバ用のクラスタ IP サービスなのでご注意ください。また、以下の例ではデフォルトの DNS ドメイン名として、 `cluster.local` を使っている前提です。

        [ req ]
        default_bits = 2048
        prompt = no
        default_md = sha256
        req_extensions = req_ext
        distinguished_name = dn
        
        [ dn ]
        C = <country>
        ST = <state>
        L = <city>
        O = <organization>
        OU = <organization unit>
        CN = <MASTER_IP>
        
        [ req_ext ]
        subjectAltName = @alt_names
        
        [ alt_names ]
        DNS.1 = kubernetes
        DNS.2 = kubernetes.default
        DNS.3 = kubernetes.default.svc
        DNS.4 = kubernetes.default.svc.cluster
        DNS.5 = kubernetes.default.svc.cluster.local
        IP.1 = <MASTER_IP>
        IP.2 = <MASTER_CLUSTER_IP>
        
        [ v3_ext ]
        authorityKeyIdentifier=keyid,issuer:always
        basicConstraints=CA:FALSE
        keyUsage=keyEncipherment,dataEncipherment
        extendedKeyUsage=serverAuth,clientAuth
        subjectAltName=@alt_names
<!--1.  Generate the certificate signing request based on the config file:-->5. 設定ファイルをもとに、証明書署名要求（CSR）を作成します。

        openssl req -new -key server.key -out server.csr -config csr.conf
<!--1.  Generate the server certificate using the ca.key, ca.crt and server.csr: -->6. ca.key、ca.crt、server.csr を使ってサーバ証明書を生成します。

        openssl x509 -req -in server.csr -CA ca.crt -CAkey ca.key \
        -CAcreateserial -out server.crt -days 10000 \
        -extensions v3_ext -extfile csr.conf
<!--1.  View the certificate: -->7. 証明書を表示します。

        openssl x509  -noout -text -in ./server.crt

<!--
Finally, add the same parameters into the API server start parameters.
-->
最後に、API サーバ起動パラメータに対し、同じパラメータを追加します。

### cfssl

<!--
**cfssl** is another tool for certificate generation.
-->
**cfssl**  は証明書を生成するための他のツールです。

<!--
1.  Download, unpack and prepare the command line tools as shown below.
    Note that you may need to adapt the sample commands based on the hardware
    architecture and cfssl version you are using.
-->1. 以下にあるようにコマンドライン・ツールをダウンロードし、展開し、準備します。以下の例で使用しているハードウェアのアーキテクチャと利用する cfssl バージョンは、皆さんの環境と違う場合がありますのでご注意ください。

        curl -L https://pkg.cfssl.org/R1.2/cfssl_linux-amd64 -o cfssl
        chmod +x cfssl
        curl -L https://pkg.cfssl.org/R1.2/cfssljson_linux-amd64 -o cfssljson
        chmod +x cfssljson
        curl -L https://pkg.cfssl.org/R1.2/cfssl-certinfo_linux-amd64 -o cfssl-certinfo
        chmod +x cfssl-certinfo
<!--1.  Create a directory to hold the artifacts and initialize cfssl:-->2. 成果物（アーティファクト）を置き、cfssl を初期化するディレクトリを作成します。

        mkdir cert
        cd cert
        ../cfssl print-defaults config > config.json
        ../cfssl print-defaults csr > csr.json
<!--1.  Create a JSON config file for generating the CA file, for example, `ca-config.json`:-->3. CA（認証機関）ファイルを生成用の JSON 設定ファイルを作成します。例えば `ca-config.json` です。

        {
          "signing": {
            "default": {
              "expiry": "8760h"
            },
            "profiles": {
              "kubernetes": {
                "usages": [
                  "signing",
                  "key encipherment",
                  "server auth",
                  "client auth"
                ],
                "expiry": "8760h"
              }
            }
          }
        }
<!--1.  Create a JSON config file for CA certificate signing request (CSR), for example,
    `ca-csr.json`. Be sure the replace the values marked with angle brackets with
    real values you want to use.-->4. CA（認証機関）証明書署名要求（CSR）用の JSON 設定ファイル、例えば `ca-csr` を作成します。カギ括弧で印がある値は、使用したい実際の値に置き換える必要がありますのでご注意ください。

        {
          "CN": "kubernetes",
          "key": {
            "algo": "rsa",
            "size": 2048
          },
          "names":[{
            "C": "<country>",
            "ST": "<state>",
            "L": "<city>",
            "O": "<organization>",
            "OU": "<organization unit>"
          }]
        }
<!--1.  Generate CA key (`ca-key.pem`) and certificate (`ca.pem`):-->5. CA 鍵（ `ca-key.pem` ）と証明書（ `ca.pem` ）を作成します。

        ../cfssl gencert -initca ca-csr.json | ../cfssljson -bare ca
<!--1.  Create a JSON config file for generating keys and certificates for the API
    server as shown below. Be sure to replace the values in angle brackets with
    real values you want to use. The `MASTER_CLUSTER_IP` is the service cluster
    IP for the API server as described in previous subsection.
    The sample below also assume that you are using `cluster.local` as the default
    DNS domain name.-->6. 以下に出てくる API サーバ用の鍵と証明書を生成するための、 JSON 設定ファイルを作成します。カギ括弧（例： `<MASTER_IP>`）記号の場所は、ファイル（例： `csr.conf` ）に保存する前に、確実に値を入力します。前のサブセクションで説明したように、 `MASTER_CLUSTER_IP` （マスタのクラスタ IP）とは API サーバ用のクラスタ IP サービスなのでご注意ください。また、以下のサンプルではデフォルトの DNS ドメイン名として `cluster.local` を使っているものと想定しています。

        {
          "CN": "kubernetes",
          "hosts": [
            "127.0.0.1",
            "<MASTER_IP>",
            "<MASTER_CLUSTER_IP>",
            "kubernetes",
            "kubernetes.default",
            "kubernetes.default.svc",
            "kubernetes.default.svc.cluster",
            "kubernetes.default.svc.cluster.local"
          ],
          "key": {
            "algo": "rsa",
            "size": 2048
          },
          "names": [{
            "C": "<country>",
            "ST": "<state>",
            "L": "<city>",
            "O": "<organization>",
            "OU": "<organization unit>"
          }]
        } 
<!--1.  Generate the key and certificate for the API server, which are by default
    saved into file `server-key.pem` and `server.pem` respectively:--> 7. API サーバ用の鍵と証明書を生成します。デフォルトではファイル `server-key.pem` と `server.pem` がそれぞれ相当します。

        ../cfssl gencert -ca=ca.pem -ca-key=ca-key.pem \
        --config=ca-config.json -profile=kubernetes \
        server-csr.json | ../cfssljson -bare server

<!--
## Distributing Self-Signed CA Certificate
-->
## 自己署名 CA （証明機関）証明書の配布 {#distributing-self-signed-ca-certificate}

<!--
A client node may refuse to recognize a self-signed CA certificate as valid.
For a non-production deployment, or for a deployment that runs behind a company
firewall, you can distribute a self-signed CA certificate to all clients and
refresh the local list for valid certificates.
-->
クライアント・ノードによっては、自己署名 CA 証明書は無効と認識し、拒否する場合があります。
本番環境以外への展開（デプロイ）や、会社のファイアウォールの後ろで動くように展開（デプロイ）するには、自己署名 CA 証明書を全てのクライアントに配布し、ローカルで有効な証明書の一覧を更新できます。

<!--
On each client, perform the following operations:
-->
クライアントごとに以下の操作を処理します：

```bash
$ sudo cp ca.crt /usr/local/share/ca-certificates/kubernetes.crt
$ sudo update-ca-certificates
Updating certificates in /etc/ssl/certs...
1 added, 0 removed; done.
Running hooks in /etc/ca-certificates/update.d....
done.
```
<!--
## Certificates API
-->
## 証明書 API（Certificates API） {#certificates-api}

<!--
You can use the `certificates.k8s.io` API to provision
x509 certificates to use for authentication as documented
[here](/docs/tasks/tls/managing-tls-in-a-cluster).
-->
x509 証明書をプロビジョン（自動配置）するための認証で `certificates.k8s.io` API を使う場合は、[こちら](/jp/docs/tasks/tls/managing-tls-in-a-cluster) にドキュメントがあります。

{{% /capture %}}
