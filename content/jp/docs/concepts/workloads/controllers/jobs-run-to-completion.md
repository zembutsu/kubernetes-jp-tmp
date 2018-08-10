---
reviewers:
- erictune
- soltysh
title: Jobs（ジョブ） - 完了する実行
content_template: templates/concept
weight: 70
---

{{% capture overview %}}

<!--
A _job_ creates one or more pods and ensures that a specified number of them successfully terminate.
As pods successfully complete, the _job_ tracks the successful completions.  When a specified number
of successful completions is reached, the job itself is complete.  Deleting a Job will cleanup the
pods it created.
-->
_job（ジョブ）_ は１つまたは複数のポッドを作成し、指定した数の終了（terminate）が成功するようにします。
ポッドの処理が成功すると、 _job_ トラックは処理完了になります。
処理完了が指定数に到達すると、ジョブ自身が完了となります。
ジョブの削除は、ジョブによって作成されたポッドをクリーンアップ（掃除）します。

<!--
A simple case is to create one Job object in order to reliably run one Pod to completion.
The Job object will start a new Pod if the first pod fails or is deleted (for example
due to a node hardware failure or a node reboot).
-->
簡単な例は、あるポッドに対して確実に処理を行いたい１つのジョブ・オブジェクトの作成です。
１つめのポッドが失敗または削除されると（たとえば、ノードのハードウェア障害やノードの再起動が原因で）、ジョブ・オブジェクトは新しいポッドを開始します。

<!--
A Job can also be used to run multiple pods in parallel.
-->
また、ジョブは複数のポッドで並列に実行できます。


{{% /capture %}}

{{< toc >}}

{{% capture body %}}

<!--
## Running an example Job
-->
## サンプル・ジョブの実行 {#running-an-example-ojb}

<!--
Here is an example Job config.  It computes π to 2000 places and prints it out.
It takes around 10s to complete.
-->
ここにあるのはサンプルのジョブ設定です。
 π を 2000 回計算し、出力します。

{{< codenew file="controllers/job.yaml" >}}

<!--
Run the example job by downloading the example file and then running this command:
-->
このサンプル・ジョブ緒w実行するには、サンプル・ファイルをダウンロードして実行するために、こちらのコマンドを実行します：

```shell
$ kubectl create -f https://k8s.io/examples/controllers/job.yaml
job "pi" created
```

<!--
Check on the status of the job using this command:
-->
ジョブの状態を確認するには、このコマンドを使います：

```shell
$ kubectl describe jobs/pi
Name:             pi
Namespace:        default
Selector:         controller-uid=b1db589a-2c8d-11e6-b324-0209dc45a495
Labels:           controller-uid=b1db589a-2c8d-11e6-b324-0209dc45a495
                  job-name=pi
Annotations:      <none>
Parallelism:      1
Completions:      1
Start Time:       Tue, 07 Jun 2016 10:56:16 +0200
Pods Statuses:    0 Running / 1 Succeeded / 0 Failed
Pod Template:
  Labels:       controller-uid=b1db589a-2c8d-11e6-b324-0209dc45a495
                job-name=pi
  Containers:
   pi:
    Image:      perl
    Port:
    Command:
      perl
      -Mbignum=bpi
      -wle
      print bpi(2000)
    Environment:        <none>
    Mounts:             <none>
  Volumes:              <none>
Events:
  FirstSeen    LastSeen    Count    From            SubobjectPath    Type        Reason            Message
  ---------    --------    -----    ----            -------------    --------    ------            -------
  1m           1m          1        {job-controller }                Normal      SuccessfulCreate  Created pod: pi-dtn4q
```

<!--
To view completed pods of a job, use `kubectl get pods`.
-->
ジョブを完了したポッドを見るには、 `kubectl get pods` を使います。

<!--
To list all the pods that belong to a job in a machine readable form, you can use a command like this:
-->
マシンでジョブを読み込んでいるポッドの一覧を表示するには、次のようなコマンドを使います：

```shell
$ pods=$(kubectl get pods --selector=job-name=pi --output=jsonpath={.items..metadata.name})
$ echo $pods
pi-aiw0a
```

<!--
Here, the selector is the same as the selector for the job.  The `--output=jsonpath` option specifies an expression
that just gets the name from each pod in the returned list.
-->
これで、セレクタはジョブに対するセレクタと同じになりました。
`--output=josnpath` オプションの指定が表しているのは、戻ってきた一覧から、各ポッドの名前のみを取得します。

<!--
View the standard output of one of the pods:
-->
ポッドの１つの標準出力を表示します：

```shell
$ kubectl logs $pods
3.1415926535897932384626433832795028841971693993751058209749445923078164062862089986280348253421170679821480865132823066470938446095505822317253594081284811174502841027019385211055596446229489549303819644288109756659334461284756482337867831652712019091456485669234603486104543266482133936072602491412737245870066063155881748815209209628292540917153643678925903600113305305488204665213841469519415116094330572703657595919530921861173819326117931051185480744623799627495673518857527248912279381830119491298336733624406566430860213949463952247371907021798609437027705392171762931767523846748184676694051320005681271452635608277857713427577896091736371787214684409012249534301465495853710507922796892589235420199561121290219608640344181598136297747713099605187072113499999983729780499510597317328160963185950244594553469083026425223082533446850352619311881710100031378387528865875332083814206171776691473035982534904287554687311595628638823537875937519577818577805321712268066130019278766111959092164201989380952572010654858632788659361533818279682303019520353018529689957736225994138912497217752834791315155748572424541506959508295331168617278558890750983817546374649393192550604009277016711390098488240128583616035637076601047101819429555961989467678374494482553797747268471040475346462080466842590694912933136770289891521047521620569660240580381501935112533824300355876402474964732639141992726042699227967823547816360093417216412199245863150302861829745557067498385054945885869269956909272107975093029553211653449872027559602364806654991198818347977535663698074265425278625518184175746728909777727938000816470600161452491921732172147723501414419735685481613611573525521334757418494684385233239073941433345477624168625189835694855620992192221842725502542568876717904946016534668049886272327917860857843838279679766814541009538837863609506800642251252051173929848960841284886269456042419652850222106611863067442786220391949450471237137869609563643719172874677646575739624138908658326459958133904780275901
```

<!--
## Writing a Job Spec
-->
## ジョブの Spec を書く {#writing-a-job-spec}

<!--
As with all other Kubernetes config, a Job needs `apiVersion`, `kind`, and `metadata` fields.
-->
他の全ての Kubernetes 設定ファイルと同様に、ジョブには `apiAversion`、 `kind` 、 `metadata` フィールドが必要です。

<!--
A Job also needs a [`.spec` section](https://git.k8s.io/community/contributors/devel/api-conventions.md#spec-and-status).
-->
また、ジョブには [`.spec` セレクション](https://git.k8s.io/community/contributors/devel/api-conventions.md#spec-and-status) も必要です。

<!--
### Pod Template
-->
### ポッド・テンプレート

<!--
The `.spec.template` is the only required field of the `.spec`.
-->
`.spec.template` は `.spec` で唯一必須なフィールドです。

<!--
The `.spec.template` is a [pod template](/docs/concepts/workloads/pods/pod-overview/#pod-templates). It has exactly the same schema as a [pod](/docs/user-guide/pods), except it is nested and does not have an `apiVersion` or `kind`.
-->


<!--
In addition to required fields for a Pod, a pod template in a job must specify appropriate
labels (see [pod selector](#pod-selector)) and an appropriate restart policy.
-->

<!--
Only a [`RestartPolicy`](/docs/concepts/workloads/pods/pod-lifecycle/#restart-policy) equal to `Never` or `OnFailure` is allowed.
-->

<!--
### Pod Selector
-->
### ポッド・セレクタ {#pod-selector}

<!--
The `.spec.selector` field is optional.  In almost all cases you should not specify it.
See section [specifying your own pod selector](#specifying-your-own-pod-selector).
-->
`.spec.selector` フィールドはオプションです。
ほとんどのケースでこれを指定すべきではありません。
[自分でポッド・セレクタを指定](#specifying-your-own-pod-selector) をご覧ください。

<!--
### Parallel Jobs
-->
### 並列ジョブ（Parallel Jobs） {#parallel-jobs}

<!--
There are three main types of jobs:
-->
ジョブには主に３つのタイプがあります：

<!--
1. Non-parallel Jobs
  - normally only one pod is started, unless the pod fails.
  - job is complete as soon as Pod terminates successfully.
1. Parallel Jobs with a *fixed completion count*:
  - specify a non-zero positive value for `.spec.completions`.
  - the job is complete when there is one successful pod for each value in the range 1 to `.spec.completions`.
  - **not implemented yet:** Each pod passed a different index in the range 1 to `.spec.completions`.
1. Parallel Jobs with a *work queue*:
  - do not specify `.spec.completions`, default to `.spec.parallelism`.
  - the pods must coordinate with themselves or an external service to determine what each should work on.
  - each pod is independently capable of determining whether or not all its peers are done, thus the entire Job is done.
  - when _any_ pod terminates with success, no new pods are created.
  - once at least one pod has terminated with success and all pods are terminated, then the job is completed with success.
  - once any pod has exited with success, no other pod should still be doing any work or writing any output.  They should all be
    in the process of exiting.
-->
1. 並列ではないジョブ
  - 通常はポッドが開始した時のみで、ポッド障害時は除く
  - ポッドの停止（terminates）に成功すると、ただちにジョブも完了
1. *fixed completion count* （固定した完了数）を持つ並列ジョブ：
  - 0 ではない整数を `.spec.completions` に指定
  - 1 から `.spec.completions` までの値のポッドが成功すると、ジョブが完了
  - **未実装：** １から `.spec.completions` まで、各ポッドは異なるインデックスを持つ
1. *work queue* （動作キュー）を持つ並列ジョブ：
  - `.spec.completions` を指定せず、デフォルトは `.spec.parallelism`
  - ポッドは自分自身または外部のサービスを通し、何を処理すべきか決定する必要がある
  - ポッドは各々のピア（末端）として独立して何を行ったかどうかを決められる能力が必要であり、ここにはジョブの処理全体も含める
  - _あらゆる_ ポッドの完了が成功したら、新しいポッドを作成しない
  - 少なくとも１つのポッドの成功が完了（terminated）すると、全てのポッドが削除され、ジョブも成功として処理が完了する。
  - あらゆるポッドの成功が終了（exited)すると、他のポッドは何も処理しないか、何も出力しない。そして、残った全てのプロセスを終了（exit）する。

<!--
For a Non-parallel job, you can leave both `.spec.completions` and `.spec.parallelism` unset.  When both are
unset, both are defaulted to 1.
-->
並列ではないジョブの場合は  `.spec.completions` と  `.spec.parallelism` のいずれも設定不要です。
どちらも設定しなければ、いずれもデフォルトの 1 になります。

<!--
For a Fixed Completion Count job, you should set `.spec.completions` to the number of completions needed.
You can set `.spec.parallelism`, or leave it unset and it will default to 1.
-->
固定完了カウント・ジョブ（Fixed Completion Count job）の場合、 `.spec.completions` が終了数の指定のために必要になります。
 `.spec.parallelism` を指定できますし、未指定のままであればデフォルトの 1 になります。

<!--
For a Work Queue Job, you must leave `.spec.completions` unset, and set `.spec.parallelism` to
a non-negative integer.
-->
ワーク・キュー・ジョブ（Work Queue Job）の場合、 `.spec.completions` は未指定のままにする必要があり、また、  `.spec.completions`  は負ではない整数の指定が必要です。

<!--
For more information about how to make use of the different types of job, see the [job patterns](#job-patterns) section.
-->
異なるジョブの使い方に関する情報は  [ジョブ・パターン](#job-patterns)  をご覧ください。

<!--
#### Controlling Parallelism
-->
#### 並列化の制御 {#controlling-parallelism}

<!--
The requested parallelism (`.spec.parallelism`) can be set to any non-negative value.
If it is unspecified, it defaults to 1.
If it is specified as 0, then the Job is effectively paused until it is increased.
-->
並列化の要求（ `.spec.parallelism` ）には、あらゆる負ではない整数を設定できます。
指定しなければ、デフォルトの 1 になります。
0 を指定すると、ジョブのカウント数が増えないため、事実上の停止となります。

<!--
Actual parallelism (number of pods running at any instant) may be more or less than requested
parallelism, for a variety of reasons:
-->
実際の並列化（あらゆるインスタンス上で実行するポッドの数）にあたっては、様々な理由によって、並列化を要求した数よりも少なくなります：

<!--
- For Fixed Completion Count jobs, the actual number of pods running in parallel will not exceed the number of
  remaining completions.   Higher values of `.spec.parallelism` are effectively ignored.
- For work queue jobs, no new pods are started after any pod has succeeded -- remaining pods are allowed to complete, however.
- If the controller has not had time to react.
- If the controller failed to create pods for any reason (lack of ResourceQuota, lack of permission, etc.),
  then there may be fewer pods than requested.
- The controller may throttle new pod creation due to excessive previous pod failures in the same Job.
- When a pod is gracefully shutdown, it takes time to stop.
-->
- Fixed Completion Count ジョブでは、実際に並列で実行されるジョブ数は、完了に達する数を超えない。 `.spec.parallelism` で指定したよりも高い値は無視される
- ワークキュー・ジョブでは、何らかのポッドが成功（succeeded）すると、新しいポッドは開始しない。ただし、既に動いているポッドは完了（complete）できる
- コントローラが反応する時間が無い
- コントローラは何らかの理由によってポッド作成に失敗し（ResourceQuote 不足、権限不足、など）、いくつかの追加ポッドがリクエストされる
- 以前に同じ名前のジョブが失敗していると、コントローラは新しいポッド作成を絞る
- ポッドが丁寧な（gracefully）シャットダウンをすると、停止まで時間がかかる

<!--
## Handling Pod and Container Failures
-->
## ポッドとコンテナ障害の扱い {#handling-pod-and-container-failures}

<!--
A Container in a Pod may fail for a number of reasons, such as because the process in it exited with
a non-zero exit code, or the Container was killed for exceeding a memory limit, etc.  If this
happens, and the `.spec.template.spec.restartPolicy = "OnFailure"`, then the Pod stays
on the node, but the Container is re-run.  Therefore, your program needs to handle the case when it is
restarted locally, or else specify `.spec.template.spec.restartPolicy = "Never"`.
See [pods-states](/docs/concepts/workloads/pods/pod-lifecycle/#example-states) for more information on `restartPolicy`.
-->
ポッド内のコンテナは複数の理由によって障害（fail）となる可能性があります。
たとえば、プロセスが0以外の終了コードで終了したり、メモリ制限に達したためコンテナが強制停止（kill）されたりです。
障害が起こると、 `.spec.template.spec.restartPolicy = "OnFailure"` となり、ノード上のポッドは維持されたままですが、コンテナは再実行されます。
それゆれ、プログラムが障害発生時にローカルでリスタートできるように扱えるようにするか、あるいは `.spec.template.spec.restartPolicy = "Never"` を指定する必要があります。
`restartPolicy` （再起動方針）の詳しい情報については [pods-states](/docs/concepts/workloads/pods/pod-lifecycle/#example-states) をご覧ください。

<!--
An entire Pod can also fail, for a number of reasons, such as when the pod is kicked off the node
(node is upgraded, rebooted, deleted, etc.), or if a container of the Pod fails and the
`.spec.template.spec.restartPolicy = "Never"`.  When a Pod fails, then the Job controller
starts a new Pod.  Therefore, your program needs to handle the case when it is restarted in a new
pod.  In particular, it needs to handle temporary files, locks, incomplete output and the like
caused by previous runs.
-->
また、いくつかの理由によるポッド全体で障害も起こります。
たとえば、ノードによってポッドがキック・オフ（追い出され）たり（ノードの更新、再起動、削除、など）、あるいはポッドのコンテナで障害がおこり、かつ `.spec.template.spec.restartPolicy = "Never"` の場合です。
ポッドが障害になると、ジョブ・コントローラは新しいポッドを開始します。
そのため、プログラムは新しいポッドが再起動した場合の処理を扱う必要があります。
とりわけ必要なのは、以前の実行によって行われた一次ファイル、ロック、完了していない出力の扱いです。


<!--
Note that even if you specify `.spec.parallelism = 1` and `.spec.completions = 1` and
`.spec.template.spec.restartPolicy = "Never"`, the same program may
sometimes be started twice.
-->
たとえ `.spec.parallelism = 1` と `.spec.completions = 1` と `.spec.template.spec.restartPolicy = "Never"` を指定したとしても、同じプログラムが時々２つ同時に起動する場合がありますのでご注意ください。

<!--
If you do specify `.spec.parallelism` and `.spec.completions` both greater than 1, then there may be
multiple pods running at once.  Therefore, your pods must also be tolerant of concurrency.
-->
`.spec.parallelism` と `.spec.completions` を、どちらも１以上指定した場合、複数のポッドが同時に実行される場合があります。
そのため、ポッドが同時の実行処理に耐えうるようにする必要があります。

<!--
### Pod Backoff failure policy
-->
### ポッド・バックオフ障害方針（#pod-backoff-failure-policy）

<!--
There are situations where you want to fail a Job after some amount of retries
due to a logical error in configuration etc.
To do so, set `.spec.backoffLimit` to specify the number of retries before
considering a Job as failed. The back-off limit is set by default to 6. Failed
Pods associated with the Job are recreated by the Job controller with an
exponential back-off delay (10s, 20s, 40s ...) capped at six minutes. The
back-off count is reset if no new failed Pods appear before the Job's next
status check.
-->
設定ファイル上の論理エラー等によって、ジョブが失敗した後も、リトライを繰り返したい場合があります。
このような時は、ジョブが失敗したと考えられる前にリトライする数を `.spec.backoffLimit` で設定します。
バックオフ・リミット（back-off limit）はデフォルトでは6に設定されています。
ジョブに関連するポッドで障害が起こると、ジョブ・コントローラは指数関数的にバック・オフ遅延（10秒、20秒、40秒…）を6分間を上限とする再作成を繰り返します。
ジョブの次の状態チェックの前に新しいポッド障害が派生しなければ、バック・オフカウントはリセットされます。

{{< note >}}
<!--
**Note:** Due to a known issue [#54870](https://github.com/kubernetes/kubernetes/issues/54870),
when the `.spec.template.spec.restartPolicy` field is set to "`OnFailure`", the
back-off limit may be ineffective. As a short-term workaround, set the restart
policy for the embedded template to "`Never`".
-->
**メモ：** 衆知の問題 [#54870](https://github.com/kubernetes/kubernetes/issues/54870) によると、  `.spec.template.spec.restartPolicy` フィールドを `OnFailure` と設定すると、バック・オフ制限が効かなくなります。
短期間のワークアラウンドであれば、embedded テンプレートに対する再起動ポリシーは "`Never`" に設定します。
{{< /note >}}

<!--
## Job Termination and Cleanup
-->
## ジョブの終了とクリーンアップ {#job-termination-and-cleanup}

<!--
When a Job completes, no more Pods are created, but the Pods are not deleted either.  Keeping them around
allows you to still view the logs of completed pods to check for errors, warnings, or other diagnostic output.
The job object also remains after it is completed so that you can view its status.  It is up to the user to delete
old jobs after noting their status.  Delete the job with `kubectl` (e.g. `kubectl delete jobs/pi` or `kubectl delete -f ./job.yaml`). When you delete the job using `kubectl`, all the pods it created are deleted too.
-->
ジョブが完了すると、それ以上のポッドは作成されませんが、ポッドもまた削除されません。
ポッドが維持されているのは、完了したポッドのログを確認し、エラー、警告、その他の判断用の出力を確認できるようになっています。
ジョブ・オブジェクトもまた完了後に残り続けているため、状態を確認できます。
ユーザ次第で、状態の確認後に古いジョブを削除できます。
ジョブの削除は `kubectl` で行います（例：  `kubectl delete jobs/pi` or `kubectl delete -f ./job.yaml`）。
`kubectl` を使ってジョブを削除すると、作成されたポッドも削除されます。

<!--
By default, a Job will run uninterrupted unless a Pod fails, at which point the Job defers to the
`.spec.backoffLimit` described above. Another way to terminate a Job is by setting an active deadline.
Do this by setting the `.spec.activeDeadlineSeconds` field of the Job to a number of seconds.
-->
デフォルトでは、ポッドはポッドの障害による中断がなければ、ジョブは `.spec.backoffLimit`  を保留します。
あるいは、別のジョブ削除方法として、アクティブなデッド・ラインを設定できます。
この設定をするには、ジョブの `.spec.activeDeadlineSeconds` フィールドで秒数を指定します。

<!--
The `activeDeadlineSeconds` applies to the duration of the job, no matter how many Pods are created.
Once a Job reaches `activeDeadlineSeconds`, the Job and all of its Pods are terminated.
The result is that the job has a status with `reason: DeadlineExceeded`. 
-->
`activeDeadlineSeconds` にはジョブの存続期間を適用するもので、ポッドをいくつ作成しようと関係ありません。
ひとたびジョブが `activeDeadlineSeconds` に到達すると、ジョブとジョブを持っているポッドの一覧を停止（terminated）します。
その結果、ジョブの状態は `reason:DeadlineExceeded` （デッドラインに到達）となります。

<!--
Note that a Job's `.spec.activeDeadlineSeconds` takes precedence over its `.spec.backoffLimit`. Therefore, a Job that is retrying one or more failed Pods will not deploy additional Pods once it reaches the time limit specified by `activeDeadlineSeconds`, even if the `backoffLimit` is not yet reached.
-->
ジョブの `.spec.activeDeadlineSeconds` は自身の `.spec.backoffLimit` よりも優先されます。
そのため、ジョブはポッドの１回または複数の障害によって再試行されますが、指定した `activeDeadlineSeconds` の時間制限に到達すると、追加のポッドを展開（デプロイ）しません。この時、たとえ `backoffLimit` には到達していなくてもです。

<!--
Example:
-->
例：

```yaml
apiVersion: batch/v1
kind: Job
metadata:
  name: pi-with-timeout
spec:
  backoffLimit: 5
  activeDeadlineSeconds: 100
  template:
    spec:
      containers:
      - name: pi
        image: perl
        command: ["perl",  "-Mbignum=bpi", "-wle", "print bpi(2000)"]
      restartPolicy: Never
```

Note that both the Job Spec and the [Pod Template Spec](https://kubernetes.io/docs/concepts/workloads/pods/init-containers/#detailed-behavior) within the Job have an `activeDeadlineSeconds` field. Ensure that you set this field at the proper level.
ジョブ Spec とジョブ内の [ポッド・テンプレート Spec](https://kubernetes.io/docs/concepts/workloads/pods/init-containers/#detailed-behavior) は `activeDeadlineSeconds` フィールダウがあります。
このフィールドは適切なレベルに確実に指定してください。

<!--
## Job Patterns
-->
## ジョブ・パターン {#job-patterns}

<!--
The Job object can be used to support reliable parallel execution of Pods.  The Job object is not
designed to support closely-communicating parallel processes, as commonly found in scientific
computing.  It does support parallel processing of a set of independent but related *work items*.
These might be emails to be sent, frames to be rendered, files to be transcoded, ranges of keys in a
NoSQL database to scan, and so on.
-->
ジョブ・オブジェクトはポッドの信頼できうる並列実行をサポートします。
ジョブ・オブジェクトは並列化プロセスが密接に通信できるようにするための、通常の計算科学で見られるような設計をサポートしていません。
個々には並列化プロセスをサポートしていませんが、 関連した *work items* があります。
これらによって、メールを送ったり、フレームをレンダーしたり、ファイルを送ったり、NoSQL でキーの幅や、データベースのスキャンなどができます。

<!--
In a complex system, there may be multiple different sets of work items.  Here we are just
considering one set of work items that the user wants to manage together &mdash; a *batch job*.
-->
複雑なシステムでは、work items で様々な設定ができます。
ここにあるのは work items のセットの例であり、ユーザによっては *batch job（バッチジョブ）* と一緒に管理したい場合もあるでしょう。

<!--
There are several different patterns for parallel computation, each with strengths and weaknesses.
The tradeoffs are:
-->
並列化計算には複数のパターンがありますが、どれも利点と欠点があります。
トレードオフとは：

<!--
- One Job object for each work item, vs. a single Job object for all work items.  The latter is
  better for large numbers of work items.  The former creates some overhead for the user and for the
  system to manage large numbers of Job objects.
- Number of pods created equals number of work items, vs. each pod can process multiple work items.
  The former typically requires less modification to existing code and containers.  The latter
  is better for large numbers of work items, for similar reasons to the previous bullet.
- Several approaches use a work queue.  This requires running a queue service,
  and modifications to the existing program or container to make it use the work queue.
  Other approaches are easier to adapt to an existing containerised application.
-->
ｰ work item ごとに１つのジョブ・オブジェクトを作成 vs 全ての work item に対して１つのジョブ・オブジェクトを作成。後者は多くの数の work item で有利。前者はユーザとシステムにとっては大量のジョブ・オブジェクトを管理する必要があるためオーバヘッドになる。
- ポッドの作成数は work item 数と同じ vs 書くポッドは複数の work item を処理。前者は典型的に必要なのは、既存のコードやコンテナに対する変更がより少ない。後者は多くの work items 数だが、前項と似たような問題。
- 複数のアプローチをワーク・キューで使う。複数のキュー・サービスの実行が必要であり、ワーク・キューを利用できるようにするには、既存のプログラムやコンテナに対する変更が必要になる。他のアプローチは簡単に既存のコンテナ化アプリケーションを導入できる。

<!--
The tradeoffs are summarized here, with columns 2 to 4 corresponding to the above tradeoffs.
The pattern names are also links to examples and more detailed description.
-->
トレードオフを要約したのがこちらです。列２～４が前述のトレードオフに相当します。
また、パターン名のリンク先にサンプルと詳細な説明があります。

<!--
|                            Pattern                                   | Single Job object | Fewer pods than work items? | Use app unmodified? |  Works in Kube 1.1? |
| -------------------------------------------------------------------- |:-----------------:|:---------------------------:|:-------------------:|:-------------------:|
| [Job Template Expansion](/docs/tasks/job/parallel-processing-expansion/)            |                   |                             |          ✓          |          ✓          |
| [Queue with Pod Per Work Item](/docs/tasks/job/coarse-parallel-processing-work-queue/)   |         ✓         |                             |      sometimes      |          ✓          |
| [Queue with Variable Pod Count](/docs/tasks/job/fine-parallel-processing-work-queue/)  |         ✓         |             ✓               |                     |          ✓          |
| Single Job with Static Work Assignment                               |         ✓         |                             |          ✓          |                     |
-->
|                            パターン                                   | 単一ジョブ・オブジェクト | ポッドが works item より少ないか？ | 使用するアプリは変更しないか？ |  Kube 1.1で動くか？ |
| -------------------------------------------------------------------- |:-----------------:|:---------------------------:|:-------------------:|:-------------------:|
| [ジョブ・テンプレート拡張](/jp/docs/tasks/job/parallel-processing-expansion/)            |                   |                             |          ✓          |          ✓          |
| [Work Item ごとにポッドでキュー](/jp/docs/tasks/job/coarse-parallel-processing-work-queue/)   |         ✓         |                             |      sometimes      |          ✓          |
| [可変ポッド・カウントでキュー](/jp/docs/tasks/job/fine-parallel-processing-work-queue/)  |         ✓         |             ✓               |                     |          ✓          |
| 単一ジョブに静的な Work を割り当て                               |         ✓         |                             |          ✓          |                     |

<!--
When you specify completions with `.spec.completions`, each Pod created by the Job controller
has an identical [`spec`](https://git.k8s.io/community/contributors/devel/api-conventions.md#spec-and-status).  This means that
all pods will have the same command line and the same
image, the same volumes, and (almost) the same environment variables.  These patterns
are different ways to arrange for pods to work on different things.
-->
各ポッドを作成するとき、ジョブ・コントローラによって `.spec.completions` を指定すると、同一の  [`spec`](https://git.k8s.io/community/contributors/devel/api-conventions.md#spec-and-status) を持ちます。
つまり、全てのポッドが同じコマンドライン、同じイメージ、同じボリューム、（ほとんど）同じ環境変数を持ちます。
これらのパターンはポッドの組み合わせにより、動作に様々な影響を与えます。

<!--
This table shows the required settings for `.spec.parallelism` and `.spec.completions` for each of the patterns.
Here, `W` is the number of work items.
-->
こちらの表は `.spec.parallelism` と `.spec.completions` を各パターンごとの設定で必要なものを示します。
この `W` とは work item の数です。

<!--
|                             Pattern                                  | `.spec.completions` |  `.spec.parallelism` |
| -------------------------------------------------------------------- |:-------------------:|:--------------------:|
| [Job Template Expansion](/docs/tasks/job/parallel-processing-expansion/)           |          1          |     should be 1      |
| [Queue with Pod Per Work Item](/docs/tasks/job/coarse-parallel-processing-work-queue/)   |          W          |        any           |
| [Queue with Variable Pod Count](/docs/tasks/job/fine-parallel-processing-work-queue/)  |          1          |        any           |
| Single Job with Static Work Assignment                               |          W          |        any           |
-->
|                             パターン                                  | `.spec.completions` |  `.spec.parallelism` |
| -------------------------------------------------------------------- |:-------------------:|:--------------------:|
| [ジョブ・テンプレート拡張](/jp/docs/tasks/job/parallel-processing-expansion/)           |          1          |     1 であるべき     |
| [Work Item ごとにポッドでキュー](/jp/docs/tasks/job/coarse-parallel-processing-work-queue/)   |          W          |        何でも           |
| [可変ポッド・カウントでキュー](/jp/docs/tasks/job/fine-parallel-processing-work-queue/)  |          1          |        何でも           |
| 単一ジョブに静的な Work を割り当て                               |          W          |        何でも           |


<!--
## Advanced Usage
-->
## 高度な使い方 {#advanced-usage}

<!--
### Specifying your own pod selector
-->
### 自身のポッド・セレクタを指定 {#specifying-your-own-pod-selector}

<!--
Normally, when you create a job object, you do not specify `.spec.selector`.
The system defaulting logic adds this field when the job is created.
It picks a selector value that will not overlap with any other jobs.
-->
通常、ジョブ・オブジェクトの作成時に `.spec.selector` を指定しません。
システムはジョブの作成時、デフォルトでは論理的にこのフィールドを追加します。
ここで選択される値は、他のジョブとは重複しません。

<!--
However, in some cases, you might need to override this automatically set selector.
To do this, you can specify the `.spec.selector` of the job.
-->
しかし、いくつかの場合において、自動的に設定されるセレクタを上書きしたい必要があるでしょう。
そのような場合は、ジョブに対して `.spec.selector` を指定できます。

<!--
Be very careful when doing this.  If you specify a label selector which is not
unique to the pods of that job, and which matches unrelated pods, then pods of the unrelated
job may be deleted, or this job may count other pods as completing it, or one or both
of the jobs may refuse to create pods or run to completion.  If a non-unique selector is
chosen, then other controllers (e.g. ReplicationController) and their pods may behave
in unpredictable ways too.  Kubernetes will not stop you from making a mistake when
specifying `.spec.selector`.
--
この作業を行うには非常に注意が必要です。
ジョブのポッドに対してユニークではないラベル・セレクタを指定しまうと、関係のないポッドに一致してしまう可能性があり、関係の無いポッドのジョブが削除されるか、あるいはそのジョブが他のポッドと競合するか、あるいは両方のジョブが競合によってポッドの作成が拒否される場合があります。
ユニークではないセレクタを選択してしまうと、他のコントローラ（例：ReplicationController） や他のポッドも予測できない挙動となる可能性があります。
`.spec.selector` の指定を間違えたとしても、Kubernetes はそれを止められません。

<!--
Here is an example of a case when you might want to use this feature.
-->
ここにある例は、将来的に皆さんが使いたくなるものかもしれません。

<!--
Say job `old` is already running.  You want existing pods
to keep running, but you want the rest of the pods it creates
to use a different pod template and for the job to have a new name.
You cannot update the job because these fields are not updatable.
Therefore, you delete job `old` but leave its pods
running, using `kubectl delete jobs/old --cascade=false`.
Before deleting it, you make a note of what selector it uses:
-->
ジョブ `old` が既に実行中とします。
既存のポッドは実行中のままにし、これから作成するポッドに対しては、異なったポッド・テンプレートを使い、ジョブに対しては新しい名前を与えたい場合があるでしょう。
各フィールドは更新できないため、ジョブは更新できません。
そのため、 ジョブ `old` を削除し、実行中のポッドから話すには、 `kubectl delete job/old --cascade=false` を使います。
削除する前に、どのセレクタを使うかメモをとっておきます：

```
kind: Job
metadata:
  name: old
  ...
spec:
  selector:
    matchLabels:
      job-uid: a8f3d00d-c6d2-11e5-9f87-42010af00002
  ...
```

<!--
Then you create a new job with name `new` and you explicitly specify the same selector.
Since the existing pods have label `job-uid=a8f3d00d-c6d2-11e5-9f87-42010af00002`,
they are controlled by job `new` as well.
-->
それから、ジョブ名 `new` という新しいジョブを作成し、同じセレクタを明示的に指定します。
既存のポッドのラベルは `job-uid=a8f3d00d-c6d2-11e5-9f87-42010af00002` なので、新しいジョブ `new` からも同様に制御できます。

<!--
You need to specify `manualSelector: true` in the new job since you are not using
the selector that the system normally generates for you automatically.
-->
新しいジョブでは `manualSelector: true`  を指定する必要があります。これは、セレクタが使っていないもの、これは、システムによって通常自動的に作成されないものを選択します。

```
kind: Job
metadata:
  name: new
  ...
spec:
  manualSelector: true
  selector:
    matchLabels:
      job-uid: a8f3d00d-c6d2-11e5-9f87-42010af00002
  ...
```

<!--
The new Job itself will have a different uid from `a8f3d00d-c6d2-11e5-9f87-42010af00002`.  Setting
`manualSelector: true` tells the system to that you know what you are doing and to allow this
mismatch.
-->
新しいジョブ自身は `a8f3d00d-c6d2-11e5-9f87-42010af00002` とは異なる UID を持っています。
システムに対して `manualSelector: true` を設定するとは、自分自身でこの不一致に対してどのような対処をするのか知っている場合に行います。

<!--
## Alternatives
-->
## 別の方法 {#alternatives}

<!--
### Bare Pods
-->
### 単なるポッド（Bare Pods） {#bare-pods}

<!--
When the node that a pod is running on reboots or fails, the pod is terminated
and will not be restarted.  However, a Job will create new pods to replace terminated ones.
For this reason, we recommend that you use a job rather than a bare pod, even if your application
requires only a single pod.
-->
再起動や障害時にポッドが実行中だとしても、停止されたポッドは再起動されないのでご注意ください。
ただし、ジョブが停止されたポッドに置き換えるため、新しいポッドを作成します。
そのため、私たちが推奨するのはジョブを使うのではなく、単なるポッドを使う方法です。たとアプリケーションが必要なのが１つのポッドだとしてもです。

<!--
### Replication Controller
-->
### レプリケーション・コントローラ {#replication-controller}

<!--
Jobs are complementary to [Replication Controllers](/docs/user-guide/replication-controller).
A Replication Controller manages pods which are not expected to terminate (e.g. web servers), and a Job
manages pods that are expected to terminate (e.g. batch jobs).
-->
ジョブは [レプリケーション・コントローラ](/jp/docs/user-guide/replication-controller) を補完するものです。
レプリケーション・コントローラによるポッドの管理には、停止（例：ウェブ・サーバ）を想定していません。また、ジョブの管理するポッドは停止を想定しています（例：バッチ・ジョブ）。

<!--
As discussed in [Pod Lifecycle](/docs/concepts/workloads/pods/pod-lifecycle/), `Job` is *only* appropriate for pods with
`RestartPolicy` equal to `OnFailure` or `Never`.  (Note: If `RestartPolicy` is not set, the default
value is `Always`.)
-->
[ポッド・ライフサイクル](/jp/docs/concepts/workloads/pods/pod-lifecycle/) に関する議論においては、 `ジョブ` とはポッドに対して *唯一* 適切なのは 'RestartPolicy' が `OnFailure` または `Never` です。（メモ： `RestartPolicy` を指定しなければ、デフォルト値は `Always` です）

<!--
### Single Job starts Controller Pod
-->
### シングル・ジョブでコントローラ・ポッドを開始 {#single-job-starts-controller-pod}

<!--
Another pattern is for a single Job to create a pod which then creates other pods, acting as a sort
of custom controller for those pods.  This allows the most flexibility, but may be somewhat
complicated to get started with and offers less integration with Kubernetes.
-->
シングル・ジョブに対する他のパターンは、ポッドを作成するために他のポッドを作成することです。各ポッドに対して様々な種類のコントローラを動かします。
これによって、最も柔軟性を得られます。
しかし、始めるにあたっては何らかの複雑さがあり、Kubernetes との統合もしづらくなります。

<!--
One example of this pattern would be a Job which starts a Pod which runs a script that in turn
starts a Spark master controller (see [spark example](https://github.com/kubernetes/examples/tree/{{< param "githubbranch" >}}/staging/spark/README.md)), runs a spark
driver, and then cleans up.
-->
このパターンの例としては、ジョブをポッドで開始するにあたってスクリプトで Spark マスタ・コントローラを開始し([spark サンプル](https://github.com/kubernetes/examples/tree/{{< param "githubbranch" >}}/staging/spark/README.md)をご覧ください）、spark ドライバを実行し、その後クリーンアップする場合でしょう。

<!--
An advantage of this approach is that the overall process gets the completion guarantee of a Job
object, but complete control over what pods are created and how work is assigned to them.
-->
この手法の利点は、全体的なプロセスが完全なジョブ・オブジェクトの実行を保証します。
しかしなが、完全な制御を及ぼせるのはポッドの作成時と、処理がポッドに割り当てられた時のみです。

<!--
## Cron Jobs
-->
## cron ジョブ {#cron-jobs}

<!--
Support for creating Jobs at specified times/dates (i.e. cron) is available in Kubernetes [1.4](https://github.com/kubernetes/kubernetes/pull/11980). More information is available in the [cron job documents](/docs/concepts/workloads/controllers/cron-jobs/)
-->
Kubernetes [1.4](https://github.com/kubernetes/kubernetes/pull/11980) では時間と日付（cron のように）を指定するジョブの作成をサポートしました。
詳しい情報については  [cron ジョブ・ドキュメント](/jp/docs/concepts/workloads/controllers/cron-jobs/) をご覧ください。

{{% /capture %}}
