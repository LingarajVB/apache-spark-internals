= [[TaskSchedulerImpl]] TaskSchedulerImpl

*TaskSchedulerImpl* is the default xref:scheduler:TaskScheduler.adoc[TaskScheduler] that uses a <<backend, SchedulerBackend>> to schedule tasks (for execution on a cluster manager).

When a Spark application starts (and so an instance of xref:ROOT:SparkContext.adoc#creating-instance[SparkContext is created]) TaskSchedulerImpl with a xref:scheduler:SchedulerBackend.adoc[SchedulerBackend] and xref:scheduler:DAGScheduler.adoc[DAGScheduler] are created and soon started.

.TaskSchedulerImpl and Other Services
image::taskschedulerimpl-sparkcontext-schedulerbackend-dagscheduler.png[align="center"]

TaskSchedulerImpl <<resourceOffers, generates tasks for executor resource offers>>.

TaskSchedulerImpl can <<getRackForHost, track racks per host and port>> (that however is xref:spark-on-yarn:spark-yarn-yarnscheduler.adoc[only used with Hadoop YARN cluster manager]).

Using xref:ROOT:configuration-properties.adoc#spark.scheduler.mode[spark.scheduler.mode] configuration property you can select the xref:scheduler:spark-scheduler-SchedulingMode.adoc[scheduling policy].

TaskSchedulerImpl <<submitTasks, submits tasks>> using xref:scheduler:spark-scheduler-SchedulableBuilder.adoc[SchedulableBuilders].

[[CPUS_PER_TASK]]
TaskSchedulerImpl uses xref:ROOT:configuration-properties.adoc#spark.task.cpus[spark.task.cpus] configuration property for...FIXME

== [[creating-instance]] Creating Instance

TaskSchedulerImpl takes the following to be created:

* [[sc]] xref:ROOT:SparkContext.adoc[]
* <<maxTaskFailures, Acceptable number of task failures>>
* [[isLocal]] isLocal flag for local and cluster run modes (default: `false`)

TaskSchedulerImpl initializes the <<internal-properties, internal properties>>.

TaskSchedulerImpl sets xref:scheduler:TaskScheduler.adoc#schedulingMode[schedulingMode] to the value of xref:ROOT:configuration-properties.adoc#spark.scheduler.mode[spark.scheduler.mode] configuration property.

NOTE: `schedulingMode` is part of xref:scheduler:TaskScheduler.adoc#schedulingMode[TaskScheduler] contract.

Failure to set `schedulingMode` results in a `SparkException`:

```
Unrecognized spark.scheduler.mode: [schedulingModeConf]
```

Ultimately, TaskSchedulerImpl creates a xref:scheduler:TaskResultGetter.adoc[TaskResultGetter].

== [[backend]] SchedulerBackend

TaskSchedulerImpl is assigned a xref:scheduler:SchedulerBackend.adoc[SchedulerBackend] when requested to <<initialize, initialize>>.

The lifecycle of the SchedulerBackend is tightly coupled to the lifecycle of the owning TaskSchedulerImpl:

* When <<start, started up>> so is the xref:scheduler:SchedulerBackend.adoc#start[SchedulerBackend]

* When <<stop, stopped>>, so is the xref:scheduler:SchedulerBackend.adoc#stop[SchedulerBackend]

TaskSchedulerImpl <<waitBackendReady, waits until the SchedulerBackend is ready>> before requesting it for the following:

* xref:scheduler:SchedulerBackend.adoc#reviveOffers[Reviving resource offers] when requested to <<submitTasks, submitTasks>>, <<statusUpdate, statusUpdate>>, <<handleFailedTask, handleFailedTask>>, <<checkSpeculatableTasks, checkSpeculatableTasks>>, and <<executorLost, executorLost>>

* xref:scheduler:SchedulerBackend.adoc#killTask[Killing tasks] when requested to <<killTaskAttempt, killTaskAttempt>> and <<killAllTaskAttempts, killAllTaskAttempts>>

* xref:scheduler:SchedulerBackend.adoc#defaultParallelism[Default parallelism], <<applicationId, applicationId>> and <<applicationAttemptId, applicationAttemptId>> when requested for the <<defaultParallelism, defaultParallelism>>, xref:scheduler:SchedulerBackend.adoc#applicationId[applicationId] and xref:scheduler:SchedulerBackend.adoc#applicationAttemptId[applicationAttemptId], respectively

== [[applicationId]] Unique Identifier of Spark Application

[source, scala]
----
applicationId(): String
----

NOTE: `applicationId` is part of xref:scheduler:TaskScheduler.adoc#applicationId[TaskScheduler] contract.

`applicationId` simply request the <<backend, SchedulerBackend>> for the xref:scheduler:SchedulerBackend.adoc#applicationId[applicationId].

== [[nodeBlacklist]] `nodeBlacklist` Method

CAUTION: FIXME

== [[cleanupTaskState]] `cleanupTaskState` Method

CAUTION: FIXME

== [[newTaskId]] `newTaskId` Method

CAUTION: FIXME

== [[getExecutorsAliveOnHost]] `getExecutorsAliveOnHost` Method

CAUTION: FIXME

== [[isExecutorAlive]] `isExecutorAlive` Method

CAUTION: FIXME

== [[hasExecutorsAliveOnHost]] `hasExecutorsAliveOnHost` Method

CAUTION: FIXME

== [[hasHostAliveOnRack]] `hasHostAliveOnRack` Method

CAUTION: FIXME

== [[executorLost]] `executorLost` Method

CAUTION: FIXME

== [[mapOutputTracker]] `mapOutputTracker`

CAUTION: FIXME

== [[starvationTimer]] `starvationTimer`

CAUTION: FIXME

== [[executorHeartbeatReceived]] executorHeartbeatReceived Method

[source, scala]
----
executorHeartbeatReceived(
  execId: String,
  accumUpdates: Array[(Long, Seq[AccumulatorV2[_, _]])],
  blockManagerId: BlockManagerId): Boolean
----

executorHeartbeatReceived is...FIXME

executorHeartbeatReceived is part of the xref:scheduler:TaskScheduler.adoc#executorHeartbeatReceived[TaskScheduler] contract.

== [[cancelTasks]] Cancelling All Tasks of Stage -- `cancelTasks` Method

[source, scala]
----
cancelTasks(stageId: Int, interruptThread: Boolean): Unit
----

NOTE: `cancelTasks` is part of xref:scheduler:TaskScheduler.adoc#contract[TaskScheduler contract].

`cancelTasks` cancels all tasks submitted for execution in a stage `stageId`.

NOTE: `cancelTasks` is used exclusively when `DAGScheduler` xref:scheduler:DAGScheduler.adoc#failJobAndIndependentStages[cancels a stage].

== [[handleSuccessfulTask]] `handleSuccessfulTask` Method

[source, scala]
----
handleSuccessfulTask(
  taskSetManager: TaskSetManager,
  tid: Long,
  taskResult: DirectTaskResult[_]): Unit
----

`handleSuccessfulTask` simply xref:scheduler:TaskSetManager.adoc#handleSuccessfulTask[forwards the call to the input `taskSetManager`] (passing `tid` and `taskResult`).

NOTE: `handleSuccessfulTask` is called when xref:scheduler:TaskResultGetter.adoc#enqueueSuccessfulTask[`TaskSchedulerGetter` has managed to deserialize the task result of a task that finished successfully].

== [[handleTaskGettingResult]] `handleTaskGettingResult` Method

[source, scala]
----
handleTaskGettingResult(taskSetManager: TaskSetManager, tid: Long): Unit
----

`handleTaskGettingResult` simply xref:scheduler:TaskSetManager.adoc#handleTaskGettingResult[forwards the call to the `taskSetManager`].

NOTE: `handleTaskGettingResult` is used to inform that xref:scheduler:TaskResultGetter.adoc#enqueueSuccessfulTask[`TaskResultGetter` enqueues a successful task with `IndirectTaskResult` task result (and so is about to fetch a remote block from a `BlockManager`)].

== [[applicationAttemptId]] `applicationAttemptId` Method

[source, scala]
----
applicationAttemptId(): Option[String]
----

CAUTION: FIXME

== [[getRackForHost]] Tracking Racks per Hosts and Ports -- `getRackForHost` Method

[source, scala]
----
getRackForHost(value: String): Option[String]
----

`getRackForHost` is a method to know about the racks per hosts and ports. By default, it assumes that racks are unknown (i.e. the method returns `None`).

NOTE: It is overriden by the YARN-specific TaskScheduler xref:spark-on-yarn:spark-yarn-yarnscheduler.adoc[YarnScheduler].

`getRackForHost` is currently used in two places:

* <<resourceOffers, TaskSchedulerImpl.resourceOffers>> to track hosts per rack (using the <<internal-registries, internal `hostsByRack` registry>>) while processing resource offers.

* <<removeExecutor, TaskSchedulerImpl.removeExecutor>> to...FIXME

* xref:scheduler:TaskSetManager.adoc#addPendingTask[TaskSetManager.addPendingTask], xref:scheduler:TaskSetManager.adoc#[TaskSetManager.dequeueTask], and xref:scheduler:TaskSetManager.adoc#dequeueSpeculativeTask[TaskSetManager.dequeueSpeculativeTask]

== [[initialize]] Initializing -- `initialize` Method

[source, scala]
----
initialize(
  backend: SchedulerBackend): Unit
----

`initialize` initializes TaskSchedulerImpl.

.TaskSchedulerImpl initialization
image::TaskSchedulerImpl-initialize.png[align="center"]

`initialize` saves the input <<backend, SchedulerBackend>>.

`initialize` then sets <<rootPool, schedulable `Pool`>> as an empty-named xref:spark-scheduler-Pool.adoc[Pool] (passing in <<schedulingMode, SchedulingMode>>, `initMinShare` and `initWeight` as `0`).

NOTE: <<schedulingMode, SchedulingMode>> is defined when <<creating-instance, TaskSchedulerImpl is created>>.

NOTE: <<schedulingMode, schedulingMode>> and <<rootPool, rootPool>> are a part of xref:scheduler:TaskScheduler.adoc#contract[TaskScheduler Contract].

`initialize` sets <<schedulableBuilder, SchedulableBuilder>> (based on <<schedulingMode, SchedulingMode>>):

* xref:spark-scheduler-FIFOSchedulableBuilder.adoc[FIFOSchedulableBuilder] for `FIFO` scheduling mode
* xref:spark-scheduler-FairSchedulableBuilder.adoc[FairSchedulableBuilder] for `FAIR` scheduling mode

`initialize` xref:spark-scheduler-SchedulableBuilder.adoc#buildPools[requests `SchedulableBuilder` to build pools].

CAUTION: FIXME Why are `rootPool` and `schedulableBuilder` created only now? What do they need that it is not available when TaskSchedulerImpl is created?

NOTE: `initialize` is called while xref:ROOT:SparkContext.adoc#createTaskScheduler[SparkContext is created and creates SchedulerBackend and `TaskScheduler`].

== [[start]] Starting TaskSchedulerImpl

As part of xref:ROOT:spark-SparkContext-creating-instance-internals.adoc[initialization of a `SparkContext`], TaskSchedulerImpl is started (using `start` from the xref:scheduler:TaskScheduler.adoc#contract[TaskScheduler Contract]).

[source, scala]
----
start(): Unit
----

`start` starts the xref:scheduler:SchedulerBackend.adoc[scheduler backend].

.Starting TaskSchedulerImpl in Spark Standalone
image::taskschedulerimpl-start-standalone.png[align="center"]

`start` also starts <<task-scheduler-speculation, `task-scheduler-speculation` executor service>>.

== [[statusUpdate]] Handling Task Status Update -- `statusUpdate` Method

[source, scala]
----
statusUpdate(tid: Long, state: TaskState, serializedData: ByteBuffer): Unit
----

`statusUpdate` finds xref:scheduler:TaskSetManager.adoc[TaskSetManager] for the input `tid` task (in <<taskIdToTaskSetManager, taskIdToTaskSetManager>>).

When `state` is `LOST`, `statusUpdate`...FIXME

NOTE: `TaskState.LOST` is only used by the deprecated Mesos fine-grained scheduling mode.

When `state` is one of the xref:scheduler:Task.adoc#states[finished states], i.e. `FINISHED`, `FAILED`, `KILLED` or `LOST`, `statusUpdate` <<cleanupTaskState, cleanupTaskState>> for the input `tid`.

`statusUpdate` xref:scheduler:TaskSetManager.adoc#removeRunningTask[requests `TaskSetManager` to unregister `tid` from running tasks].

`statusUpdate` requests <<taskResultGetter, TaskResultGetter>> to xref:scheduler:TaskResultGetter.adoc#enqueueSuccessfulTask[schedule an asynchrounous task to deserialize the task result (and notify TaskSchedulerImpl back)] for `tid` in `FINISHED` state and xref:scheduler:TaskResultGetter.adoc#enqueueFailedTask[schedule an asynchrounous task to deserialize `TaskFailedReason` (and notify TaskSchedulerImpl back)] for `tid` in the other finished states (i.e. `FAILED`, `KILLED`, `LOST`).

If a task is in `LOST` state, `statusUpdate` xref:scheduler:DAGScheduler.adoc#executorLost[notifies `DAGScheduler` that the executor was lost] (with `SlaveLost` and the reason `Task [tid] was lost, so marking the executor as lost as well.`) and xref:scheduler:SchedulerBackend.adoc#reviveOffers[requests SchedulerBackend to revive offers].

In case the `TaskSetManager` for `tid` could not be found (in <<taskIdToTaskSetManager, taskIdToTaskSetManager>> registry), you should see the following ERROR message in the logs:

```
ERROR Ignoring update with state [state] for TID [tid] because its task set is gone (this is likely the result of receiving duplicate task finished status updates)
```

Any exception is caught and reported as ERROR message in the logs:

```
ERROR Exception in statusUpdate
```

CAUTION: FIXME image with scheduler backends calling `TaskSchedulerImpl.statusUpdate`.

[NOTE]
====
`statusUpdate` is used when:

* `DriverEndpoint` (of xref:scheduler:CoarseGrainedSchedulerBackend.adoc[CoarseGrainedSchedulerBackend]) is requested to xref:scheduler:CoarseGrainedSchedulerBackend-DriverEndpoint.adoc#StatusUpdate[handle a StatusUpdate message]

* `LocalEndpoint` is requested to xref:spark-local:spark-LocalEndpoint.adoc#StatusUpdate[handle a StatusUpdate message]

* `MesosFineGrainedSchedulerBackend` is requested to handle a task status update
====

== [[speculationScheduler]][[task-scheduler-speculation]] task-scheduler-speculation Scheduled Executor Service -- `speculationScheduler` Internal Attribute

`speculationScheduler` is a http://docs.oracle.com/javase/8/docs/api/java/util/concurrent/ScheduledExecutorService.html[java.util.concurrent.ScheduledExecutorService] with the name *task-scheduler-speculation* for xref:ROOT:speculative-execution-of-tasks.adoc[].

When <<start, TaskSchedulerImpl starts>> (in non-local run mode) with xref:ROOT:configuration-properties.adoc#spark.speculation[spark.speculation] enabled, `speculationScheduler` is used to schedule <<checkSpeculatableTasks, checkSpeculatableTasks>> to execute periodically every xref:ROOT:configuration-properties.adoc#spark.speculation.interval[spark.speculation.interval] after the initial `spark.speculation.interval` passes.

`speculationScheduler` is shut down when <<stop, TaskSchedulerImpl stops>>.

== [[checkSpeculatableTasks]] Checking for Speculatable Tasks

[source, scala]
----
checkSpeculatableTasks(): Unit
----

`checkSpeculatableTasks` requests `rootPool` to check for speculatable tasks (if they ran for more than `100` ms) and, if there any, requests xref:scheduler:SchedulerBackend.adoc#reviveOffers[SchedulerBackend to revive offers].

NOTE: `checkSpeculatableTasks` is executed periodically as part of xref:ROOT:speculative-execution-of-tasks.adoc[].

== [[maxTaskFailures]] Acceptable Number of Task Failures

TaskSchedulerImpl can be given the acceptable number of task failures when created or defaults to xref:ROOT:configuration-properties.adoc#spark.task.maxFailures[spark.task.maxFailures] configuration property.

The number of task failures is used when <<submitTasks, submitting tasks>> through xref:scheduler:TaskSetManager.adoc[TaskSetManager].

== [[removeExecutor]] Cleaning up After Removing Executor -- `removeExecutor` Internal Method

[source, scala]
----
removeExecutor(executorId: String, reason: ExecutorLossReason): Unit
----

`removeExecutor` removes the `executorId` executor from the following <<internal-registries, internal registries>>: <<executorIdToTaskCount, executorIdToTaskCount>>, `executorIdToHost`, `executorsByHost`, and `hostsByRack`. If the affected hosts and racks are the last entries in `executorsByHost` and `hostsByRack`, appropriately, they are removed from the registries.

Unless `reason` is `LossReasonPending`, the executor is removed from `executorIdToHost` registry and xref:spark-scheduler-Schedulable.adoc#executorLost[TaskSetManagers get notified].

NOTE: The internal `removeExecutor` is called as part of <<statusUpdate, statusUpdate>> and xref:scheduler:TaskScheduler.adoc#executorLost[executorLost].

== [[postStartHook]] Handling Nearly-Completed SparkContext Initialization -- `postStartHook` Callback

[source, scala]
----
postStartHook(): Unit
----

NOTE: `postStartHook` is part of the xref:scheduler:TaskScheduler.adoc#postStartHook[TaskScheduler Contract] to notify a xref:scheduler:TaskScheduler.adoc[task scheduler] that the `SparkContext` (and hence the Spark application itself) is about to finish initialization.

`postStartHook` simply <<waitBackendReady, waits until a scheduler backend is ready>>.

== [[stop]] Stopping TaskSchedulerImpl -- `stop` Method

[source, scala]
----
stop(): Unit
----

`stop()` stops all the internal services, i.e. <<task-scheduler-speculation, `task-scheduler-speculation` executor service>>, xref:scheduler:SchedulerBackend.adoc[SchedulerBackend], xref:scheduler:TaskResultGetter.adoc[TaskResultGetter], and <<starvationTimer, starvationTimer>> timer.

== [[defaultParallelism]] Finding Default Level of Parallelism -- `defaultParallelism` Method

[source, scala]
----
defaultParallelism(): Int
----

NOTE: `defaultParallelism` is part of xref:scheduler:TaskScheduler.adoc#defaultParallelism[TaskScheduler contract] as a hint for sizing jobs.

`defaultParallelism` simply requests <<backend, SchedulerBackend>> for the xref:scheduler:SchedulerBackend.adoc#defaultParallelism[default level of parallelism].

NOTE: *Default level of parallelism* is a hint for sizing jobs that `SparkContext` xref:ROOT:SparkContext.adoc#defaultParallelism[uses to create RDDs with the right number of partitions when not specified explicitly].

== [[submitTasks]] Submitting Tasks (of TaskSet) for Execution -- `submitTasks` Method

[source, scala]
----
submitTasks(taskSet: TaskSet): Unit
----

NOTE: `submitTasks` is part of the xref:scheduler:TaskScheduler.adoc#submitTasks[TaskScheduler Contract] to submit the tasks (of the given xref:scheduler:TaskSet.adoc[TaskSet]) for execution.

In essence, `submitTasks` registers a new xref:scheduler:TaskSetManager.adoc[TaskSetManager] (for the given xref:scheduler:TaskSet.adoc[TaskSet]) and requests the <<backend, SchedulerBackend>> to xref:scheduler:SchedulerBackend.adoc#reviveOffers[handle resource allocation offers (from the scheduling system)].

.TaskSchedulerImpl.submitTasks
image::taskschedulerImpl-submitTasks.png[align="center"]

Internally, `submitTasks` first prints out the following INFO message to the logs:

```
Adding task set [id] with [length] tasks
```

`submitTasks` then <<createTaskSetManager, creates a TaskSetManager>> (for the given xref:scheduler:TaskSet.adoc[TaskSet] and the <<maxTaskFailures, acceptable number of task failures>>).

`submitTasks` registers (_adds_) the `TaskSetManager` per xref:scheduler:TaskSet.adoc#stageId[stage] and xref:scheduler:TaskSet.adoc#stageAttemptId[stage attempt] IDs (of the xref:scheduler:TaskSet.adoc[TaskSet]) in the <<taskSetsByStageIdAndAttempt, taskSetsByStageIdAndAttempt>> internal registry.

NOTE: <<taskSetsByStageIdAndAttempt, taskSetsByStageIdAndAttempt>> internal registry tracks the xref:scheduler:TaskSetManager.adoc[TaskSetManagers] (that represent xref:scheduler:TaskSet.adoc[TaskSets]) per stage and stage attempts. In other words, there could be many `TaskSetManagers` for a single stage, each representing a unique stage attempt.

NOTE: Not only could a task be retried (cf. <<maxTaskFailures, acceptable number of task failures>>), but also a single stage.

`submitTasks` makes sure that there is exactly one active `TaskSetManager` (with different `TaskSet`) across all the managers (for the stage). Otherwise, `submitTasks` throws an `IllegalStateException`:

```
more than one active taskSet for stage [stage]: [TaskSet ids]
```

NOTE: `TaskSetManager` is considered *active* when it is not a *zombie*.

`submitTasks` requests the <<schedulableBuilder, SchedulableBuilder>> to xref:spark-scheduler-SchedulableBuilder.adoc#addTaskSetManager[add the TaskSetManager to the schedulable pool].

NOTE: The xref:scheduler:TaskScheduler.adoc#rootPool[schedulable pool] can be a single flat linked queue (in xref:spark-scheduler-FIFOSchedulableBuilder.adoc[FIFO scheduling mode]) or a hierarchy of pools of `Schedulables` (in xref:spark-scheduler-FairSchedulableBuilder.adoc[FAIR scheduling mode]).

`submitTasks` <<submitTasks-starvationTimer, schedules a starvation task>> to make sure that the requested resources (i.e. CPU and memory) are assigned to the Spark application for a <<isLocal, non-local environment>> (the very first time the Spark application is started per <<hasReceivedTask, hasReceivedTask>> flag).

NOTE: The very first time (<<hasReceivedTask, hasReceivedTask>> flag is `false`) in cluster mode only (i.e. `isLocal` of the TaskSchedulerImpl is `false`), `starvationTimer` is scheduled to execute after xref:ROOT:configuration-properties.adoc#spark.starvation.timeout[spark.starvation.timeout]  to ensure that the requested resources, i.e. CPUs and memory, were assigned by a cluster manager.

NOTE: After the first xref:ROOT:configuration-properties.adoc#spark.starvation.timeout[spark.starvation.timeout] passes, the <<hasReceivedTask, hasReceivedTask>> internal flag is `true`.

In the end, `submitTasks` requests the <<backend, SchedulerBackend>> to xref:scheduler:SchedulerBackend.adoc#reviveOffers[reviveOffers].

TIP: Use `dag-scheduler-event-loop` thread to step through the code in a debugger.

=== [[submitTasks-starvationTimer]] Scheduling Starvation Task

Every time the starvation timer thread is executed and `hasLaunchedTask` flag is `false`, the following WARN message is printed out to the logs:

```
WARN Initial job has not accepted any resources; check your cluster UI to ensure that workers are registered and have sufficient resources
```

Otherwise, when the `hasLaunchedTask` flag is `true` the timer thread cancels itself.

== [[createTaskSetManager]] Creating TaskSetManager -- `createTaskSetManager` Method

[source, scala]
----
createTaskSetManager(taskSet: TaskSet, maxTaskFailures: Int): TaskSetManager
----

`createTaskSetManager` xref:scheduler:TaskSetManager.adoc#creating-instance[creates a `TaskSetManager`] (passing on the reference to TaskSchedulerImpl, the input `taskSet` and `maxTaskFailures`, and optional `BlacklistTracker`).

NOTE: `createTaskSetManager` uses the optional <<blacklistTrackerOpt, BlacklistTracker>> that is specified when <<creating-instance, TaskSchedulerImpl is created>>.

NOTE: `createTaskSetManager` is used exclusively when <<submitTasks, TaskSchedulerImpl submits tasks (for a given `TaskSet`)>>.

== [[handleFailedTask]] Notifying TaskSetManager that Task Failed -- `handleFailedTask` Method

[source, scala]
----
handleFailedTask(
  taskSetManager: TaskSetManager,
  tid: Long,
  taskState: TaskState,
  reason: TaskFailedReason): Unit
----

`handleFailedTask` xref:scheduler:TaskSetManager.adoc#handleFailedTask[notifies `taskSetManager` that `tid` task has failed] and, only when xref:scheduler:TaskSetManager.adoc#zombie-state[`taskSetManager` is not in zombie state] and `tid` is not in `KILLED` state, xref:scheduler:SchedulerBackend.adoc#reviveOffers[requests SchedulerBackend to revive offers].

NOTE: `handleFailedTask` is called when xref:scheduler:TaskResultGetter.adoc#enqueueSuccessfulTask[`TaskResultGetter` deserializes a `TaskFailedReason`] for a failed task.

== [[taskSetFinished]] `taskSetFinished` Method

[source, scala]
----
taskSetFinished(manager: TaskSetManager): Unit
----

`taskSetFinished` looks all xref:scheduler:TaskSet.adoc[TaskSet]s up by the stage id (in <<taskSetsByStageIdAndAttempt, taskSetsByStageIdAndAttempt>> registry) and removes the stage attempt from them, possibly with removing the entire stage record from `taskSetsByStageIdAndAttempt` registry completely (if there are no other attempts registered).

.TaskSchedulerImpl.taskSetFinished is called when all tasks are finished
image::taskschedulerimpl-tasksetmanager-tasksetfinished.png[align="center"]

NOTE: A `TaskSetManager` manages a `TaskSet` for a stage.

`taskSetFinished` then xref:spark-scheduler-Pool.adoc#removeSchedulable[removes `manager` from the parent's schedulable pool].

You should see the following INFO message in the logs:

```
Removed TaskSet [id], whose tasks have all completed, from pool [name]
```

NOTE: `taskSetFinished` method is called when xref:scheduler:TaskSetManager.adoc#maybeFinishTaskSet[`TaskSetManager` has received the results of all the tasks in a `TaskSet`].

== [[executorAdded]] Notifying DAGScheduler About New Executor -- `executorAdded` Method

[source, scala]
----
executorAdded(execId: String, host: String)
----

`executorAdded` just xref:scheduler:DAGScheduler.adoc#executorAdded[notifies `DAGScheduler` that an executor was added].

CAUTION: FIXME Image with a call from TaskSchedulerImpl to DAGScheduler, please.

NOTE: `executorAdded` uses <<dagScheduler, DAGScheduler>> that was given when <<setDAGScheduler, setDAGScheduler>>.

== [[waitBackendReady]] Waiting Until SchedulerBackend is Ready -- `waitBackendReady` Internal Method

[source, scala]
----
waitBackendReady(): Unit
----

`waitBackendReady` waits until the <<backend, SchedulerBackend>> is xref:scheduler:SchedulerBackend.adoc#isReady[ready]. If it is, `waitBackendReady` returns immediately. Otherwise, `waitBackendReady` keeps checking every `100` milliseconds (hardcoded) or the <<sc, SparkContext>> is xref:ROOT:SparkContext.adoc#stopped[stopped].

NOTE: A SchedulerBackend is xref:scheduler:SchedulerBackend.adoc#isReady[ready] by default.

If the `SparkContext` happens to be stopped while waiting, `waitBackendReady` throws an `IllegalStateException`:

```
Spark context stopped while waiting for backend
```

NOTE: `waitBackendReady` is used exclusively when TaskSchedulerImpl is requested to <<postStartHook, handle a notification that SparkContext is about to be fully initialized>>.

== [[resourceOffers]] Creating TaskDescriptions For Available Executor Resource Offers

[source, scala]
----
resourceOffers(
  offers: Seq[WorkerOffer]): Seq[Seq[TaskDescription]]
----

`resourceOffers` takes the resources `offers` (as <<WorkerOffer, WorkerOffers>>) and generates a collection of tasks (as xref:spark-scheduler-TaskDescription.adoc[TaskDescription]) to launch (given the resources available).

NOTE: <<WorkerOffer, WorkerOffer>> represents a resource offer with CPU cores free to use on an executor.

.Processing Executor Resource Offers
image::taskscheduler-resourceOffers.png[align="center"]

Internally, `resourceOffers` first updates <<hostToExecutors, hostToExecutors>> and <<executorIdToHost, executorIdToHost>> lookup tables to record new hosts and executors (given the input `offers`).

For new executors (not in <<executorIdToRunningTaskIds, executorIdToRunningTaskIds>>) `resourceOffers` <<executorAdded, notifies `DAGScheduler` that an executor was added>>.

NOTE: TaskSchedulerImpl uses `resourceOffers` to track active executors.

CAUTION: FIXME a picture with `executorAdded` call from TaskSchedulerImpl to DAGScheduler.

`resourceOffers` requests `BlacklistTracker` to `applyBlacklistTimeout` and filters out offers on blacklisted nodes and executors.

NOTE: `resourceOffers` uses the optional <<blacklistTrackerOpt, BlacklistTracker>> that was given when <<creating-instance, TaskSchedulerImpl was created>>.

CAUTION: FIXME Expand on blacklisting

`resourceOffers` then randomly shuffles offers (to evenly distribute tasks across executors and avoid over-utilizing some executors) and initializes the local data structures `tasks` and `availableCpus` (as shown in the figure below).

.Internal Structures of resourceOffers with 5 WorkerOffers (with 4, 2, 0, 3, 2 free cores)
image::TaskSchedulerImpl-resourceOffers-internal-structures.png[align="center"]

`resourceOffers` xref:spark-scheduler-Pool.adoc#getSortedTaskSetQueue[takes `TaskSets` in scheduling order] from xref:scheduler:TaskScheduler.adoc#rootPool[top-level Schedulable Pool].

.TaskSchedulerImpl Requesting TaskSets (as TaskSetManagers) from Root Pool
image::TaskSchedulerImpl-resourceOffers-rootPool-getSortedTaskSetQueue.png[align="center"]

[NOTE]
====
`rootPool` is configured when <<initialize, TaskSchedulerImpl is initialized>>.

`rootPool` is part of the xref:scheduler:TaskScheduler.adoc#rootPool[TaskScheduler Contract] and exclusively managed by xref:scheduler:spark-scheduler-SchedulableBuilder.adoc[SchedulableBuilders], i.e. xref:scheduler:spark-scheduler-FIFOSchedulableBuilder.adoc[FIFOSchedulableBuilder] and xref:scheduler:spark-scheduler-FairSchedulableBuilder.adoc[FairSchedulableBuilder] (that  xref:scheduler:spark-scheduler-SchedulableBuilder.adoc#addTaskSetManager[manage registering TaskSetManagers with the root pool]).

xref:scheduler:TaskSetManager.adoc[TaskSetManager] manages execution of the tasks in a single xref:scheduler:TaskSet.adoc[TaskSet] that represents a single xref:scheduler:Stage.adoc[Stage].
====

For every `TaskSetManager` (in scheduling order), you should see the following DEBUG message in the logs:

```
parentName: [name], name: [name], runningTasks: [count]
```

Only if a new executor was added, `resourceOffers` xref:scheduler:TaskSetManager.adoc#executorAdded[notifies every `TaskSetManager` about the change] (to recompute locality preferences).

`resourceOffers` then takes every `TaskSetManager` (in scheduling order) and offers them each node in increasing order of locality levels (per xref:scheduler:TaskSetManager.adoc#computeValidLocalityLevels[TaskSetManager's valid locality levels]).

NOTE: A `TaskSetManager` xref:scheduler:TaskSetManager.adoc##computeValidLocalityLevels[computes locality levels of the tasks] it manages.

For every `TaskSetManager` and the ``TaskSetManager``'s valid locality level, `resourceOffers` tries to <<resourceOfferSingleTaskSet, find tasks to schedule (on executors)>> as long as the `TaskSetManager` manages to launch a task (given the locality level).

If `resourceOffers` did not manage to offer resources to a `TaskSetManager` so it could launch any task, `resourceOffers` xref:scheduler:TaskSetManager.adoc#abortIfCompletelyBlacklisted[requests the `TaskSetManager` to abort the `TaskSet` if completely blacklisted].

When `resourceOffers` managed to launch a task, the internal <<hasLaunchedTask, hasLaunchedTask>> flag gets enabled (that effectively means what the name says _"there were executors and I managed to launch a task"_).

[NOTE]
====
`resourceOffers` is used when:

* xref:scheduler:CoarseGrainedSchedulerBackend-DriverEndpoint.adoc#makeOffers[`CoarseGrainedSchedulerBackend` (via RPC endpoint) makes executor resource offers]

* xref:spark-local:spark-LocalEndpoint.adoc#reviveOffers[`LocalEndpoint` revives resource offers]

* Spark on Mesos' `MesosFineGrainedSchedulerBackend` does `resourceOffers`
====

== [[resourceOfferSingleTaskSet]] Finding Tasks from TaskSetManager to Schedule on Executors -- `resourceOfferSingleTaskSet` Internal Method

[source, scala]
----
resourceOfferSingleTaskSet(
  taskSet: TaskSetManager,
  maxLocality: TaskLocality,
  shuffledOffers: Seq[WorkerOffer],
  availableCpus: Array[Int],
  tasks: Seq[ArrayBuffer[TaskDescription]]): Boolean
----

`resourceOfferSingleTaskSet` takes every `WorkerOffer` (from the input `shuffledOffers`) and (only if the number of available CPU cores (using the input `availableCpus`) is at least xref:ROOT:configuration-properties.adoc#spark.task.cpus[spark.task.cpus]) xref:scheduler:TaskSetManager.adoc#resourceOffer[requests `TaskSetManager` (as the input `taskSet`) to find a `Task` to execute (given the resource offer)] (as an executor, a host, and the input `maxLocality`).

`resourceOfferSingleTaskSet` adds the task to the input `tasks` collection.

`resourceOfferSingleTaskSet` records the task id and `TaskSetManager` in the following registries:

* <<taskIdToTaskSetManager, taskIdToTaskSetManager>>
* <<taskIdToExecutorId, taskIdToExecutorId>>
* <<executorIdToRunningTaskIds, executorIdToRunningTaskIds>>

`resourceOfferSingleTaskSet` decreases xref:ROOT:configuration-properties.adoc#spark.task.cpus[spark.task.cpus] from the input `availableCpus` (for the `WorkerOffer`).

NOTE: `resourceOfferSingleTaskSet` makes sure that the number of available CPU cores (in the input `availableCpus` per `WorkerOffer`) is at least `0`.

If there is a `TaskNotSerializableException`, you should see the following ERROR in the logs:

```
ERROR Resource offer failed, task set [name] was not serializable
```

`resourceOfferSingleTaskSet` returns whether a task was launched or not.

NOTE: `resourceOfferSingleTaskSet` is used when TaskSchedulerImpl <<resourceOffers, creates `TaskDescriptions` for available executor resource offers (with CPU cores)>>.

== [[TaskLocality]] TaskLocality -- Task Locality Preference

`TaskLocality` represents a task locality preference and can be one of the following (from most localized to the widest):

. `PROCESS_LOCAL`
. `NODE_LOCAL`
. `NO_PREF`
. `RACK_LOCAL`
. `ANY`

== [[WorkerOffer]] WorkerOffer -- Free CPU Cores on Executor

[source, scala]
----
WorkerOffer(executorId: String, host: String, cores: Int)
----

`WorkerOffer` represents a resource offer with free CPU `cores` available on an `executorId` executor on a `host`.

== [[workerRemoved]] workerRemoved Method

[source, scala]
----
workerRemoved(
  workerId: String,
  host: String,
  message: String): Unit
----

workerRemoved prints out the following INFO message to the logs:

```
Handle removed worker [workerId]: [message]
```

workerRemoved then requests the <<dagScheduler, DAGScheduler>> to xref:scheduler:DAGScheduler.adoc#workerRemoved[handle it].

workerRemoved is part of the xref:scheduler:TaskScheduler.adoc#workerRemoved[TaskScheduler] abstraction.

== [[maybeInitBarrierCoordinator]] maybeInitBarrierCoordinator Method

[source,scala]
----
maybeInitBarrierCoordinator(): Unit
----

maybeInitBarrierCoordinator...FIXME

maybeInitBarrierCoordinator is used when TaskSchedulerImpl is requested to <<resourceOffers, resourceOffers>>.

== [[logging]] Logging

Enable `ALL` logging level for `org.apache.spark.scheduler.TaskSchedulerImpl` logger to see what happens inside.

Add the following line to `conf/log4j.properties`:

[source]
----
log4j.logger.org.apache.spark.scheduler.TaskSchedulerImpl=ALL
----

Refer to xref:ROOT:spark-logging.adoc[Logging].

== [[internal-properties]] Internal Properties

[cols="30m,70",options="header",width="100%"]
|===
| Name
| Description

| dagScheduler
a| [[dagScheduler]] xref:scheduler:DAGScheduler.adoc[DAGScheduler]

Used when...FIXME

| executorIdToHost
a| [[executorIdToHost]] Lookup table of hosts per executor.

Used when...FIXME

| executorIdToRunningTaskIds
a| [[executorIdToRunningTaskIds]] Lookup table of running tasks per executor.

Used when...FIXME

| executorIdToTaskCount
a| [[executorIdToTaskCount]] Lookup table of the number of running tasks by xref:executor:Executor.adoc[].

| executorsByHost
a| [[executorsByHost]] Collection of xref:executor:Executor.adoc[executors] per host

| hasLaunchedTask
a| [[hasLaunchedTask]] Flag...FIXME

Used when...FIXME

| hostToExecutors
a| [[hostToExecutors]] Lookup table of executors per hosts in a cluster.

Used when...FIXME

| hostsByRack
a| [[hostsByRack]] Lookup table of hosts per rack.

Used when...FIXME

| nextTaskId
a| [[nextTaskId]] The next xref:scheduler:Task.adoc[task] id counting from `0`.

Used when TaskSchedulerImpl...

| rootPool
a| [[rootPool]] xref:spark-scheduler-Pool.adoc[Schedulable pool]

Used when TaskSchedulerImpl...

| schedulableBuilder
a| [[schedulableBuilder]] <<spark-scheduler-SchedulableBuilder.adoc#, SchedulableBuilder>>

Created when TaskSchedulerImpl is requested to <<initialize, initialize>> and can be one of two available builders:

* xref:spark-scheduler-FIFOSchedulableBuilder.adoc[FIFOSchedulableBuilder] when scheduling policy is FIFO (which is the default scheduling policy).

* xref:spark-scheduler-FairSchedulableBuilder.adoc[FairSchedulableBuilder] for FAIR scheduling policy.

NOTE: Use xref:ROOT:configuration-properties.adoc#spark.scheduler.mode[spark.scheduler.mode] configuration property to select the scheduling policy.

| schedulingMode
a| [[schedulingMode]] xref:spark-scheduler-SchedulingMode.adoc[SchedulingMode]

Used when TaskSchedulerImpl...

| taskSetsByStageIdAndAttempt
a| [[taskSetsByStageIdAndAttempt]] Lookup table of xref:scheduler:TaskSet.adoc[TaskSet] by stage and attempt ids.

| taskIdToExecutorId
a| [[taskIdToExecutorId]] Lookup table of xref:executor:Executor.adoc[] by task id.

| taskIdToTaskSetManager
a| [[taskIdToTaskSetManager]] Registry of active xref:scheduler:TaskSetManager.adoc[TaskSetManagers] per task id.

|===
