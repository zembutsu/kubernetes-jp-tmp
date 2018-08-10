---
reviewers:
- erictune
- soltysh
- janetkuo
title: CronJob（Cronジョブ）
content_template: templates/concept
weight: 80
---

{{% capture overview %}}

<!--
A _Cron Job_ manages time based [Jobs](/docs/concepts/workloads/controllers/jobs-run-to-completion/), namely:
-->
_Cron Job（Cron ジョブ）_ は時間を基準とした [ジョブ](/docs/concepts/workloads/controllers/jobs-run-to-completion/) であり、言い換えると：

<!--
* Once at a specified point in time
* Repeatedly at a specified point in time
-->
* 指定した時間に１回だけです。
* 指定した時間で繰り返します。

<!---
One CronJob object is like one line of a _crontab_ (cron table) file. It runs a job periodically
on a given schedule, written in [Cron](https://en.wikipedia.org/wiki/Cron) format.
-->
CronJob オブジェクトは１行の _crontab_ （cron テーブル）ファイルと同様です。
指定したスケジュールによって、ジョブを定期的に実行します。
これは [Cron](https://jp.wikipedia.org/wiki/Cron) 書式で書かれます。

<!--
For instructions on creating and working with cron jobs, and for an example of a spec file for a cron job, see [Running automated tasks with cron jobs](/docs/tasks/job/automated-tasks-with-cron-jobs).
-->
cron ジョブの作成と動作の命令や、cron ジョブの spec ファイル例については、 [cron ジョブの自動化タスクを実行](/docs/tasks/job/automated-tasks-with-cron-jobs) をご覧ください。

{{% /capture %}}

{{< toc >}}

{{% capture body %}}

<!--
## Cron Job Limitations
-->
## cron ジョブの制限 {#cron-job-limitations}

<!--
A cron job creates a job object _about_ once per execution time of its schedule. We say "about" because there
are certain circumstances where two jobs might be created, or no job might be created. We attempt to make these rare,
but do not completely prevent them. Therefore, jobs should be _idempotent_.
-->
cron ジョブは、ジョブのスケジュールを実行する時間ごとに、ジョブ・オブジェクトを _おおよそ_ １つ作成します。
「おおよそ」と書いたのは、特定の状況によっては、２つのジョブが作成される場合もありますし、ジョブが全く作成されない場合もあります。
私たちはこの状態を滅多に起こらないように試みています。
しかし、完全に防止できません。
そのため、ジョブは _idempotent（冪等）_ であるべきです（訳者注：何度実行しても結果が同じであるべき状態）。

<!--
If `startingDeadlineSeconds` is set to a large value or left unset (the default)
and if `concurrencyPolicy` is set to `Allow`, the jobs will always run
at least once.
-->
もし `startingDeadlineSeconds` を大きな値にするか、未指定のままであれば（デフォルト）、また、 `concurrencyPolicy` を `Allow` に設定する場合、ジョブは常に少なくとも１回実行します。

<!--
Jobs may fail to run if the CronJob controller is not running or broken for a
span of time from before the start time of the CronJob to start time plus
`startingDeadlineSeconds`, or if the span covers multiple start times and
`concurrencyPolicy` does not allow concurrency.
For example, suppose a cron job is set to start at exactly `08:30:00` and its
`startingDeadlineSeconds` is set to 10, if the CronJob controller happens to
be down from `08:29:00` to `08:42:00`, the job will not start.
Set a longer `startingDeadlineSeconds` if starting later is better than not
starting at all.
-->
ジョブの実行に失敗するのは、CronJob コントローラが実行中でないか、Crontab の開始時間の前の待機期間中に故障するか、 `startingDeadlineSeconds` で指定した時間を経過するか、あるいは `concurrencyPolicy` が並列性を許可しないのに処理中に複数回実行されれる場合です。
たとえば、cron ジョブの開始を `08:30:00` 丁度に設定し、 `startingDeadlineSeconds` を 10 に設定したとします。
もし CronJob コントローラのダウンが `08:29:00` から `08:42:00` にかけて発生したとすると、ジョブは開始されません。
長い `startingDeadlineSeconds` を指定すると、開始が遅れますが実行できないわけではありません。

<!--
The Cronjob is only responsible for creating Jobs that match its schedule, and
the Job in turn is responsible for the management of the Pods it represents.
-->
Cron ジョブが唯一責任を持つのはジョブの作成であり、それがスケジュールに一致したらジョブを順番に実行しくのは管理しているポッドです。


{{% /capture %}}
