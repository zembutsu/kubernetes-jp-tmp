---
title: RBAC (Role-Based Access Control：役割に応じたアクセス制御)
id: rbac
date: 2018-04-12
full_link: /jp/docs/admin/authorization/rbac/
short_description: >
  <!--Manages authorization decisions, allowing admins to dynamically configure access policies through the Kubernetes API.-->
  認証の可否を管理するにあたり、管理者が Kubernetes API を通してアクセス・ポリシーを動的に設定変更できるようにします。

aka: 
tags:
- security
- fundamental
---
 <!--Manages authorization decisions, allowing admins to dynamically configure access policies through the {{< glossary_tooltip text="Kubernetes API" term_id="kubernetes-api" >}}.-->
 認証の可否を管理するにあたり、管理者が {{< glossary_tooltip text="Kubernetes API" term_id="kubernetes-api" >}} を通してアクセス・ポリシーを動的に設定変更できるようにします。

<!--more--> 

<!--
RBAC utilizes *roles*, which contain permission rules, and *role bindings*, which grant the permissions defined in a role to a set of users.
-->
RBAC は権限のルールを含む *role（ロール：役割）* と、ユーザのセットに対して役割の定義に応じて権限を与える　*role bindings（役割の割り当て）* を使います。