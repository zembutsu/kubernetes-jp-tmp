---
reviewers:
- sig-cluster-lifecycle
title: kubeadm でコントロール・プレーンをカスタマイズ
content_template: templates/concept
weight: 50
---

{{% capture overview %}}

コントロール・プレーンを構成する APIServer（APIサーバ）、ControllerManager（コントローラ・マネージャ）、Scheduler（スケジューラ）などの要素に対して、kubeadm で以下の項目に対する項目を使えば、初期フラグを上書きできます：

- `APIServerExtraArgs`
- `ControllerManagerExtraArgs`
- `SchedulerExtraArgs`

各項目は `キー:値` の組み合わせです。コントロール・プレーンの構成要素に対するフラグを上書きするには：

1.  設定対象に、適切な項目を追加
2.  上書きする項目に対して、フラグを追加

設定項目に関する詳細は、[API 参照ページ](https://godoc.org/k8s.io/kubernetes/cmd/kubeadm/app/apis/kubeadm#MasterConfiguration) をご覧ください。

{{% /capture %}}

{{% capture body %}}

## APIServer フラグ

詳細は [kube-apiserver 参照ドキュメント](https://kubernetes.io/docs/reference/command-line-tools-reference/kube-apiserver/) をご覧ください。

記述例：
```yaml
apiVersion: kubeadm.k8s.io/v1alpha2
kind: MasterConfiguration
kubernetesVersion: v1.11.0
metadata:
  name: 1.11-sample
apiServerExtraArgs:
  advertise-address: 192.168.0.103
  anonymous-auth: false
  enable-admission-plugins: AlwaysPullImages,DefaultStorageClass
  audit-log-path: /home/johndoe/audit.log
```

## ControllerManager フラグ

詳細は [kube-controller-manager 参照ドキュメント](https://kubernetes.io/docs/reference/command-line-tools-reference/kube-controller-manager/) をご覧ください。

記述例：
```yaml
apiVersion: kubeadm.k8s.io/v1alpha2
kind: MasterConfiguration
kubernetesVersion: v1.11.0
metadata:
  name: 1.11-sample
controllerManagerExtraArgs:
  cluster-signing-key-file: /home/johndoe/keys/ca.key
  bind-address: 0.0.0.0
  deployment-controller-sync-period: 50
```

## Scheduler フラグ

詳細は [kube-scheduler 参照ドキュメント](https://kubernetes.io/docs/reference/command-line-tools-reference/kube-scheduler/).

記述例：
```yaml
apiVersion: kubeadm.k8s.io/v1alpha2
kind: MasterConfiguration
kubernetesVersion: v1.11.0
metadata:
  name: 1.11-sample
schedulerExtraArgs:
  address: 0.0.0.0
  config: /home/johndoe/schedconfig.yaml
  kubeconfig: /home/johndoe/kubeconfig.yaml
```

{{% /capture %}}
