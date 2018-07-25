---
title: CronJob（クロン・ジョブ）
id: cronjob
date: 2018-04-12
full_link: /jp/docs/concepts/workloads/controllers/cron-jobs/
short_description: >
  <!-- Manages a [Job](/docs/concepts/workloads/controllers/jobs-run-to-completion/) that runs on a periodic schedule. -->
  定期的にスケジュールして実行する [Job（ジョブ）](/jp/docs/concepts/workloads/controllers/jobs-run-to-completion/) を管理します。

aka: 
tags:
- core-object
- workload
---
 <!--Manages a [Job](/docs/concepts/workloads/controllers/jobs-run-to-completion/) that runs on a periodic schedule.-->
 定期的にスケジュールして実行する [Job（ジョブ）](/jp/docs/concepts/workloads/controllers/jobs-run-to-completion/) を管理します。

<!--more--> 

<!--
Similar to a line in a *crontab* file, a Cronjob object specifies a schedule using the [Cron](https://en.wikipedia.org/wiki/Cron) format.
-->
Crontab オブジェクトは *crontab* ファイルの行と似ており、 [Cron](https://en.wikipedia.org/wiki/Cron) 形式でスケジュールを指定します。
