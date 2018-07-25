---
title: Secret（機密情報：シークレット）
id: secret
date: 2018-04-12
full_link: /jp/docs/concepts/configuration/secret/
short_description: >
  <!--Stores sensitive information, such as passwords, OAuth tokens, and ssh keys. -->パスワード、OAuth トークン、ssh 鍵のような機密事項を扱う情報を保管します。

aka: 
tags:
- core-object
- security
---
 <!--Stores sensitive information, such as passwords, OAuth tokens, and ssh keys.-->
 パスワード、OAuth トークン、ssh 鍵のような機密事項を扱う情報を保管します。

<!--more--> 

<!--
Allows for more control over how sensitive information is used and reduces the risk of accidental exposure, including [encryption](https://kubernetes.io/docs/tasks/administer-cluster/encrypt-data/#ensure-all-secrets-are-encrypted) at rest.  A {{< glossary_tooltip text="Pod" term_id="pod" >}} references the secret as a file in a volume mount or by the kubelet pulling images for a pod. Secrets are great for confidential data and [ConfigMaps](https://kubernetes.io/docs/tasks/configure-pod-container/configure-pod-configmap/) for non-confidential data.
-->
機密情報をどのように制御して使うかどうかや、偶発的な漏洩リスクを減らすための情報は [暗号化](/jp/docs/tasks/administer-cluster/encrypt-data/#ensure-all-secrets-are-encrypted) にあります。{{< glossary_tooltip text="ポッド" term_id="pod" >}} は機密情報をボリュームでマウントしたファイルとして参照するか、kubelet で pod.Secrets にあるイメージを取得して、秘密のデータと秘密ではないデータ用の [ConfigMaps](https://kubernetes.io/docs/tasks/configure-pod-container/configure-pod-configmap/) に使うと役立つでしょう。