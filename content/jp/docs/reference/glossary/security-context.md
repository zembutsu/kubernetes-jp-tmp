---
title: Security Context（セキュリティ・コンテキスト）
id: security-context
date: 2018-04-12
full_link: /jp/docs/tasks/configure-pod-container/security-context/
short_description: >
  <!--The securityContext field defines privilege and access control settings for a Pod or Container, including the runtime UID and GID.-->securityContext フィールドはポッドやコンテナに対する権限やアクセス制御設定を定義します。これには実行時の UID と GID も含みます。

aka: 
tags:
- security
---
 <!--The securityContext field defines privilege and access control settings for a Pod or Container, including the runtime UID and GID.-->
 securityContext フィールドはポッドやコンテナに対する権限やアクセス制御設定を定義します。これには実行時の UID と GID も含みます。

<!--more--> 

<!--
The securityContext field in a {{< glossary_tooltip term_id="pod" >}} (applying to all containers) or container is used to set the user (runAsUser) and group (fsGroup), capabilities, privilege settings, and security policies (SELinux/AppArmor/Seccomp) that container processes use.
-->
{{< glossary_tooltip text="ポッド" term_id="pod" >}} (すべてのコンテナに適用) やコンテナの securityContext は、のコンテナ・プロセスが使うユーザ（runAsUser）、グループ（fsGroup）、ケーパビリティ、特権設定、セキュリティ・ポリシー（SELinux/AppArmors/Seccomp）の指定に使います。
