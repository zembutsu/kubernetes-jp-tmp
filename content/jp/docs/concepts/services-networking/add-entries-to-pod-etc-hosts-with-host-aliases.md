---
reviewers:
- rickypai
- thockin
title: ポッドの /etc/hosts と HostAliases でエントリを追加
content_template: templates/concept
weight: 60
---

{{< toc >}}

{{% capture overview %}}
<!--
Adding entries to a Pod's /etc/hosts file provides Pod-level override of hostname resolution when DNS and other options are not applicable. In 1.7, users can add these custom entries with the HostAliases field in PodSpec.
-->
DNS や他のオプションを利用できないとき、ポッドの /etc/hosts ファイルにエントリを追加して、ホスト名の名前解決をポッド・レベルで上書きします。
1.7 では、ユーザは PodSpec の HostAliases フィールドでカスタム・エントリを追加できるようになりました。

<!--
Modification not using HostAliases is not suggested because the file is managed by Kubelet and can be overwritten on during Pod creation/restart.
-->
HostAliases を用いない変更はおすすめしません。ファイルは Kubelet によって管理されるもので、ポッドの作成・再起動によって上書きされる可能性があります。

{{% /capture %}}

{{% capture body %}}
<!--
## Default Hosts File Content
-->
## デフォルトのホスト・ファイル内容 {#default-host-file-content}

<!--
Lets start an Nginx Pod which is assigned a Pod IP:
-->
ポッド IP を割り当てて Nginx ポッドを起動しましょう。

```shell
$ kubectl run nginx --image nginx --generator=run-pod/v1
pod "nginx" created

$ kubectl get pods --output=wide
NAME     READY     STATUS    RESTARTS   AGE    IP           NODE
nginx    1/1       Running   0          13s    10.200.0.4   worker0
```

<!--
The hosts file content would look like this:
-->
hosts ファイルの内容は次のようになります。

```shell
$ kubectl exec nginx -- cat /etc/hosts
# Kubernetes-managed hosts file.
127.0.0.1	localhost
::1	localhost ip6-localhost ip6-loopback
fe00::0	ip6-localnet
fe00::0	ip6-mcastprefix
fe00::1	ip6-allnodes
fe00::2	ip6-allrouters
10.200.0.4	nginx
```

<!--
by default, the hosts file only includes ipv4 and ipv6 boilerplates like `localhost` and its own hostname.
-->
デフォルトでは、hosts ファイルには `localhost` のようなボイラー・プレート（定型文）と自分のホスト名と ipv4 および ipv6 のみがあります。

<!--
## Adding Additional Entries with HostAliases
-->
## HostAliases（ホスト別名）に追加エントリを加える {#addming-additional-entries-with-hostaliases}

<!--
In addition to the default boilerplate, we can add additional entries to the hosts file to resolve `foo.local`, `bar.local` to `127.0.0.1` and `foo.remote`, `bar.remote` to `10.1.2.3`, we can by adding HostAliases to the Pod under `.spec.hostAliases`:
-->
デフォルトのボイラー・プレートに加え、私達は `foo.bar` や `bar.local` を `127.0.0.1` に名前解決したり、 `foo.remote` や `bar.remote` を `10.1.2.3` に名前解決するためのエントリを追加できます。
このようなホスト別名（HostAliases）をポッドの `.spec.hostAliases` に追加します。

{{< code file="hostaliases-pod.yaml" >}}

<!--
This Pod can be started with the following commands:
-->
次のコマンドでポッドを起動できます。

```shell
$ kubectl apply -f hostaliases-pod.yaml
pod "hostaliases-pod" created

$ kubectl get pod -o=wide
NAME                           READY     STATUS      RESTARTS   AGE       IP              NODE
hostaliases-pod                0/1       Completed   0          6s        10.244.135.10   node3
```

<!--
The hosts file content would look like this:
-->
hosts ファイルの内容は以下のようなものです：

```shell
$ kubectl logs hostaliases-pod
# Kubernetes-managed hosts file.
127.0.0.1	localhost
::1	localhost ip6-localhost ip6-loopback
fe00::0	ip6-localnet
fe00::0	ip6-mcastprefix
fe00::1	ip6-allnodes
fe00::2	ip6-allrouters
10.244.135.10	hostaliases-pod
127.0.0.1	foo.local
127.0.0.1	bar.local
10.1.2.3	foo.remote
10.1.2.3	bar.remote
```

<!--
With the additional entries specified at the bottom.
-->
下の方に追加したエントリが表示されます。

<!--
## Limitations
-->
## 制限事項

<!--
HostAlias is only supported in 1.7+.
-->
HostAlias がサポートされているのは 1.7 以上です。

<!--
HostAlias support in 1.7 is limited to non-hostNetwork Pods because kubelet only manages the hosts file for non-hostNetwork Pods.
-->
1.7 でサポートされている HostAlias は、ホストネットワークではないポッド（non-hostNetwork）に対してのみ制限されています。
これは kubelet が管理できるのはホストネットワークではないポッドに対する hosts ファイルのみだからです。

<!--
In 1.8, HostAlias is supported for all Pods regardless of network configuration.
-->
1.8 では、HostAlias はネットワーク設定にかかわらず、すべてのポッドをサポートします。

<!--
## Why Does Kubelet Manage the Hosts File?
-->
## どうして Kubelet は hosts ファイルを管理するのでしょうか？ {#why-does-kubelet-manage-the-hosts-file}


<!--
Kubelet [manages](https://github.com/kubernetes/kubernetes/issues/14633) the hosts file for each container of the Pod to prevent Docker from [modifying](https://github.com/moby/moby/issues/17190) the file after the containers have already been started.
-->
Kubelet がポッド内の各コンテナ内にある hosts ファイルを [管理](https://github.com/kubernetes/kubernetes/issues/14633) するのは、既に開始しているコンテナに対して、Docker による [変更](https://github.com/moby/moby/issues/17190)  を阻止するためです。

<!--
Because of the managed-nature of the file, any user-written content will be overwritten whenever the hosts file is remounted by Kubelet in the event of a container restart or a Pod reschedule. Thus, it is not suggested to modify the contents of the file.
-->
ファイルは管理される性質のため、 hosts ファイルに書かれた内容は誰が書いたとしても、 Kubelet によってポッドの再スケジュールやコンテナの再起動といったイベントによって上書きされる場合があります。
そのため、ファイルの内容の変更は推奨されていません。

{{% /capture %}}

{{% capture whatsnext %}}

{{% /capture %}}
