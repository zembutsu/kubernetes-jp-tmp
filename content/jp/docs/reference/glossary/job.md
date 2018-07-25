---
title: Job（ジョブ）
id: job
date: 2018-04-12
full_link: /jp/docs/concepts/workloads/controllers/jobs-run-to-completion
short_description: >
  <!--A finite or batch task that runs to completion.-->
  有限もしくはバッチのタスク完了するまで実行します。

aka: 
tags:
- fundamental
- core-object
- workload
---
 <!--A finite or batch task that runs to completion.-->
 有限もしくはバッチのタスク完了するまで実行します。

<!--more--> 

<!--
Creates one or more {{< glossary_tooltip term_id="pod" >}} objects and ensures that a specified number of them successfully terminate. As Pods successfully complete, the Job tracks the successful completions.
-->
１つまたは複数の {{< glossary_tooltip text="ポッド" term_id="pod" >}} オブジェクトを作成し、指定した数のオブジェクトを完全に終了できるようにします。ポッドが無事に完了すると、ジョブの処理進行も完了します。
