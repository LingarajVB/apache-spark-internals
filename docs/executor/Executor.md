= Executor

Executor is a process that is used for executing xref:scheduler:Task.adoc[tasks].

Executor _typically_ runs for the entire lifetime of a Spark application which is called *static allocation of executors* (but you could also opt in for xref:ROOT:spark-dynamic-allocation.adoc[dynamic allocation]).

Executors are managed by xref:executor:ExecutorBackend.adoc[executor backends].

Executors <<startDriverHeartbeater, reports heartbeat and partial metrics for active tasks>> to the <<heartbeatReceiverRef, HeartbeatReceiver RPC Endpoint>> on the driver.

.HeartbeatReceiver's Heartbeat Message Handler
image::HeartbeatReceiver-Heartbeat.png[align="center"]

Executors provide in-memory storage for RDDs that are cached in Spark applications (via xref:storage:BlockManager.adoc[]).

When started, an executor first registers itself with the driver that establishes a communication channel directly to the driver to accept tasks for execution.

.Launching tasks on executor using TaskRunners
image::executor-taskrunner-executorbackend.png[align="center"]

*Executor offers* are described by executor id and the host on which an executor runs (see <<resource-offers, Resource Offers>> in this document).

Executors can run multiple tasks over its lifetime, both in parallel and sequentially. They track xref:executor:TaskRunner.adoc[running tasks] (by their task ids in <<runningTasks, runningTasks>> internal registry). Consult <<launchTask, Launching Tasks>> section.

Executors use a <<threadPool, Executor task launch worker thread pool>> for <<launchTask, launching tasks>>.

Executors send <<metrics, metrics>> (and heartbeats) using the <<heartbeater, internal heartbeater - Heartbeat Sender Thread>>.

It is recommended to have as many executors as data nodes and as many cores as you can get from the cluster.

Executors are described by their *id*, *hostname*, *environment* (as `SparkEnv`), and *classpath* (and, less importantly, and more for internal optimization, whether they run in xref:spark-local:index.adoc[local] or xref:ROOT:spark-cluster.adoc[cluster] mode).

== [[creating-instance]] Creating Instance

Executor takes the following to be created:

* [[executorId]] Executor ID
* [[executorHostname]] Host name
* [[env]] xref:core:SparkEnv.adoc[]
* <<userClassPath, User-defined jars>>
* <<isLocal, isLocal flag>>
* [[uncaughtExceptionHandler]] Java's UncaughtExceptionHandler (default: `SparkUncaughtExceptionHandler`)

When created, Executor prints out the following INFO messages to the logs:

```
Starting executor ID [executorId] on host [executorHostname]
```

(only for <<isLocal, non-local modes>>) Executor sets `SparkUncaughtExceptionHandler` as the default handler invoked when a thread abruptly terminates due to an uncaught exception.

(only for <<isLocal, non-local modes>>) Executor requests the xref:core:SparkEnv.adoc#blockManager[BlockManager] to xref:storage:BlockManager.adoc#initialize[initialize] (with the xref:ROOT:SparkConf.adoc#getAppId[Spark application id] of the xref:core:SparkEnv.adoc#conf[SparkConf]).

[[creating-instance-BlockManager-shuffleMetricsSource]]
(only for <<isLocal, non-local modes>>) Executor requests the xref:core:SparkEnv.adoc#metricsSystem[MetricsSystem] to xref:metrics:spark-metrics-MetricsSystem.adoc#registerSource[register] the <<executorSource, ExecutorSource>> and xref:storage:BlockManager.adoc#shuffleMetricsSource[shuffleMetricsSource] of the xref:core:SparkEnv.adoc#blockManager[BlockManager].

Executor uses SparkEnv to access the xref:core:SparkEnv.adoc#metricsSystem[MetricsSystem] and xref:core:SparkEnv.adoc#blockManager[BlockManager].

Executor <<createClassLoader, creates a task class loader>> (optionally with <<addReplClassLoaderIfNeeded, REPL support>>) and requests the system Serializer to xref:serializer:Serializer.adoc#setDefaultClassLoader[use as the default classloader] (for deserializing tasks).

Executor <<startDriverHeartbeater, starts sending heartbeats with the metrics of active tasks>>.

Executor is created when:

* CoarseGrainedExecutorBackend xref:executor:CoarseGrainedExecutorBackend.adoc#RegisteredExecutor[receives RegisteredExecutor message]

* (Spark on Mesos) MesosExecutorBackend is requested to xref:spark-on-mesos:spark-executor-backends-MesosExecutorBackend.adoc#registered[registered]

* xref:spark-local:spark-LocalEndpoint.adoc[LocalEndpoint] is created

== [[isLocal]] isLocal Flag

Executor is given a isLocal flag when created. This is how the executor knows whether it runs in local or cluster mode. It is disabled by default.

The flag is turned on for xref:spark-local:index.adoc[Spark local] (via xref:spark-local:spark-LocalEndpoint.adoc[LocalEndpoint]).

== [[userClassPath]] User-Defined Jars

Executor is given user-defined jars when created. There are no jars defined by default.

The jars are specified using xref:ROOT:configuration-properties.adoc#spark.executor.extraClassPath[spark.executor.extraClassPath] configuration property (via xref:executor:CoarseGrainedExecutorBackend.adoc#main[--user-class-path] command-line option of CoarseGrainedExecutorBackend).

== [[runningTasks]] Running Tasks

Executor tracks running tasks in a registry of xref:executor:TaskRunner.adoc[TaskRunners] per task ID.

== [[heartbeatReceiverRef]] HeartbeatReceiver RPC Endpoint Reference

xref:rpc:RpcEndpointRef.adoc[RPC endpoint reference] to xref:ROOT:spark-HeartbeatReceiver.adoc[HeartbeatReceiver] on the xref:ROOT:spark-driver.adoc[driver].

Set when Executor <<creating-instance, is created>>.

Used exclusively when Executor <<reportHeartBeat, reports heartbeats and partial metrics for active tasks to the driver>> (that happens every <<spark.executor.heartbeatInterval, spark.executor.heartbeatInterval>> interval).

== [[updateDependencies]] updateDependencies Method

[source, scala]
----
updateDependencies(
  newFiles: Map[String, Long],
  newJars: Map[String, Long]): Unit
----

updateDependencies...FIXME

updateDependencies is used when TaskRunner is requested to xref:executor:TaskRunner.adoc#run[start] (and run a task).

== [[launchTask]] Launching Task

[source, scala]
----
launchTask(
  context: ExecutorBackend,
  taskDescription: TaskDescription): Unit
----

launchTask simply creates a xref:executor:TaskRunner.adoc[] (with the given xref:executor:ExecutorBackend.adoc[] and the xref:scheduler:spark-scheduler-TaskDescription.adoc[TaskDescription]) and adds it to the <<runningTasks, runningTasks>> internal registry.

In the end, launchTask requests the <<threadPool, "Executor task launch worker" thread pool>> to execute the TaskRunner (sometime in the future).

.Launching tasks on executor using TaskRunners
image::executor-taskrunner-executorbackend.png[align="center"]

launchTask is used when:

* CoarseGrainedExecutorBackend is requested to xref:executor:CoarseGrainedExecutorBackend.adoc#LaunchTask[handle a LaunchTask message]

* LocalEndpoint RPC endpoint (of xref:spark-local:spark-LocalSchedulerBackend.adoc#[LocalSchedulerBackend]) is requested to xref:spark-local:spark-LocalEndpoint.adoc#reviveOffers[reviveOffers]

* MesosExecutorBackend is requested to xref:spark-on-mesos:spark-executor-backends-MesosExecutorBackend.adoc#launchTask[launchTask]

== [[heartbeater]] Heartbeat Sender Thread

heartbeater is a daemon {java-javadoc-url}/java/util/concurrent/ScheduledThreadPoolExecutor.html[ScheduledThreadPoolExecutor] with a single thread.

The name of the thread pool is *driver-heartbeater*.

== [[coarse-grained-executor]] Coarse-Grained Executors

*Coarse-grained executors* are executors that use xref:executor:CoarseGrainedExecutorBackend.adoc[] for task scheduling.

== [[resource-offers]] Resource Offers

Read xref:scheduler:TaskSchedulerImpl.adoc#resourceOffers[resourceOffers] in TaskSchedulerImpl and xref:scheduler:TaskSetManager.adoc#resourceOffers[resourceOffer] in TaskSetManager.

== [[threadPool]] Executor task launch worker Thread Pool

Executor uses threadPool daemon cached thread pool with the name *Executor task launch worker-[ID]* (with `ID` being the task id) for <<launchTask, launching tasks>>.

threadPool is created when <<creating-instance, Executor is created>> and shut down when <<stop, it stops>>.

== [[memory]] Executor Memory

You can control the amount of memory per executor using xref:ROOT:configuration-properties.adoc#spark.executor.memory[spark.executor.memory] configuration property. It sets the available memory equally for all executors per application.

The amount of memory per executor is looked up when xref:ROOT:SparkContext.adoc#creating-instance[SparkContext is created].

You can change the assigned memory per executor per node in xref:spark-standalone:index.adoc[standalone cluster] using xref:ROOT:SparkContext.adoc#environment-variables[SPARK_EXECUTOR_MEMORY] environment variable.

You can find the value displayed as *Memory per Node* in xref:spark-standalone:spark-standalone-Master.adoc[web UI for standalone Master] (as depicted in the figure below).

.Memory per Node in Spark Standalone's web UI
image::spark-standalone-webui-memory-per-node.png[align="center"]

The above figure shows the result of running xref:tools:spark-shell.adoc[Spark shell] with the amount of memory per executor defined explicitly (on command line), i.e.

```
./bin/spark-shell --master spark://localhost:7077 -c spark.executor.memory=2g
```

== [[metrics]] Metrics

Every executor registers its own xref:executor:ExecutorSource.adoc[] to xref:metrics:spark-metrics-MetricsSystem.adoc#report[report metrics].

== [[stop]] Stopping Executor

[source, scala]
----
stop(): Unit
----

stop requests xref:core:SparkEnv.adoc#metricsSystem[MetricsSystem] for a xref:metrics:spark-metrics-MetricsSystem.adoc#report[report].

stop shuts <<heartbeater, driver-heartbeater thread>> down (and waits at most 10 seconds).

stop shuts <<threadPool, Executor task launch worker thread pool>> down.

(only when <<isLocal, not local>>) stop xref:core:SparkEnv.adoc#stop[requests `SparkEnv` to stop].

stop is used when xref:executor:CoarseGrainedExecutorBackend.adoc#Shutdown[CoarseGrainedExecutorBackend] and xref:spark-local:spark-LocalEndpoint.adoc#StopExecutor[LocalEndpoint] are requested to stop their managed executors.

== [[computeTotalGcTime]] computeTotalGcTime Method

[source, scala]
----
computeTotalGcTime(): Long
----

computeTotalGcTime...FIXME

computeTotalGcTime is used when:

* TaskRunner is requested to xref:executor:TaskRunner.adoc#collectAccumulatorsAndResetStatusOnFailure[collectAccumulatorsAndResetStatusOnFailure] and xref:executor:TaskRunner.adoc#run[run]

* Executor is requested to <<reportHeartBeat, heartbeat with partial metrics for active tasks to the driver>>

== [[createClassLoader]] createClassLoader Method

[source, scala]
----
createClassLoader(): MutableURLClassLoader
----

createClassLoader...FIXME

createClassLoader is used when...FIXME

== [[addReplClassLoaderIfNeeded]] addReplClassLoaderIfNeeded Method

[source, scala]
----
addReplClassLoaderIfNeeded(
  parent: ClassLoader): ClassLoader
----

addReplClassLoaderIfNeeded...FIXME

addReplClassLoaderIfNeeded is used when...FIXME

== [[reportHeartBeat]] Heartbeating With Partial Metrics For Active Tasks To Driver

[source, scala]
----
reportHeartBeat(): Unit
----

reportHeartBeat collects xref:executor:TaskRunner.adoc[TaskRunners] for <<runningTasks, currently running tasks>> (aka _active tasks_) with their xref:executor:TaskRunner.adoc#task[tasks] deserialized (i.e. either ready for execution or already started).

xref:executor:TaskRunner.adoc[] has xref:TaskRunner.adoc#task[task] deserialized when it xref:executor:TaskRunner.adoc#run[runs the task].

For every running task, reportHeartBeat takes its xref:scheduler:Task.adoc#metrics[TaskMetrics] and:

* Requests xref:executor:TaskMetrics.adoc#mergeShuffleReadMetrics[ShuffleRead metrics to be merged]
* xref:executor:TaskMetrics.adoc#setJvmGCTime[Sets jvmGCTime metrics]

reportHeartBeat then records the latest values of xref:executor:TaskMetrics.adoc#accumulators[internal and external accumulators] for every task.

NOTE: Internal accumulators are a task's metrics while external accumulators are a Spark application's accumulators that a user has created.

reportHeartBeat sends a blocking xref:ROOT:spark-HeartbeatReceiver.adoc#Heartbeat[Heartbeat] message to <<heartbeatReceiverRef, `HeartbeatReceiver` endpoint>> (running on the driver). reportHeartBeat uses the value of xref:ROOT:configuration-properties.adoc#spark.executor.heartbeatInterval[spark.executor.heartbeatInterval] configuration property for the RPC timeout.

NOTE: A `Heartbeat` message contains the executor identifier, the accumulator updates, and the identifier of the xref:storage:BlockManager.adoc[].

If the response (from <<heartbeatReceiverRef, `HeartbeatReceiver` endpoint>>) is to re-register the `BlockManager`, you should see the following INFO message in the logs and reportHeartBeat requests the BlockManager to xref:storage:BlockManager.adoc#reregister[re-register] (which will register the blocks the `BlockManager` manages with the driver).

[source,plaintext]
----
Told to re-register on heartbeat
----

HeartbeatResponse requests the BlockManager to re-register when either xref:scheduler:TaskScheduler.adoc#executorHeartbeatReceived[TaskScheduler] or xref:ROOT:spark-HeartbeatReceiver.adoc#Heartbeat[HeartbeatReceiver] know nothing about the executor.

When posting the `Heartbeat` was successful, reportHeartBeat resets <<heartbeatFailures, heartbeatFailures>> internal counter.

In case of a non-fatal exception, you should see the following WARN message in the logs (followed by the stack trace).

```
Issue communicating with driver in heartbeater
```

Every failure reportHeartBeat increments <<heartbeatFailures, heartbeat failures>> up to xref:ROOT:configuration-properties.adoc#spark.executor.heartbeat.maxFailures[spark.executor.heartbeat.maxFailures] configuration property. When the heartbeat failures reaches the maximum, you should see the following ERROR message in the logs and the executor terminates with the error code: `56`.

```
Exit as unable to send heartbeats to driver more than [HEARTBEAT_MAX_FAILURES] times
```

reportHeartBeat is used when Executor is requested to <<startDriverHeartbeater, schedule reporting heartbeat and partial metrics for active tasks to the driver>> (that happens every xref:ROOT:configuration-properties.adoc#spark.executor.heartbeatInterval[spark.executor.heartbeatInterval]).

== [[startDriverHeartbeater]][[heartbeats-and-active-task-metrics]] Sending Heartbeats and Active Tasks Metrics

Executors keep sending <<metrics, metrics for active tasks>> to the driver every <<spark.executor.heartbeatInterval, spark.executor.heartbeatInterval>> (defaults to `10s` with some random initial delay so the heartbeats from different executors do not pile up on the driver).

.Executors use HeartbeatReceiver endpoint to report task metrics
image::executor-heartbeatReceiver-endpoint.png[align="center"]

An executor sends heartbeats using the <<heartbeater, internal heartbeater -- Heartbeat Sender Thread>>.

.HeartbeatReceiver's Heartbeat Message Handler
image::HeartbeatReceiver-Heartbeat.png[align="center"]

For each xref:scheduler:Task.adoc[task] in xref:executor:TaskRunner.adoc[] (in <<runningTasks, runningTasks>> internal registry), the task's metrics are computed (i.e. `mergeShuffleReadMetrics` and `setJvmGCTime`) that become part of the heartbeat (with accumulators).

NOTE: Executors track the xref:executor:TaskRunner.adoc[] that run xref:scheduler:Task.adoc[tasks]. A xref:executor:TaskRunner.adoc#run[task might not be assigned to a TaskRunner yet] when the executor sends a heartbeat.

A blocking xref:ROOT:spark-HeartbeatReceiver.adoc#Heartbeat[Heartbeat] message that holds the executor id, all accumulator updates (per task id), and xref:storage:BlockManagerId.adoc[] is sent to xref:ROOT:spark-HeartbeatReceiver.adoc[HeartbeatReceiver RPC endpoint] (with <<spark.executor.heartbeatInterval, spark.executor.heartbeatInterval>> timeout).

If the response xref:ROOT:spark-HeartbeatReceiver.adoc#Heartbeat[requests to reregister BlockManager], you should see the following INFO message in the logs:

```
Told to re-register on heartbeat
```

BlockManager is requested to xref:storage:BlockManager.adoc#reregister[reregister].

The internal <<heartbeatFailures, heartbeatFailures>> counter is reset (i.e. becomes `0`).

If there are any issues with communicating with the driver, you should see the following WARN message in the logs:

[source,plaintext]
----
Issue communicating with driver in heartbeater
----

The internal <<heartbeatFailures, heartbeatFailures>> is incremented and checked to be less than the <<spark.executor.heartbeat.maxFailures, acceptable number of failures>> (i.e. `spark.executor.heartbeat.maxFailures` Spark property). If the number is greater, the following ERROR is printed out to the logs:

```
Exit as unable to send heartbeats to driver more than [HEARTBEAT_MAX_FAILURES] times
```

The executor exits (using `System.exit` and exit code 56).

== [[logging]] Logging

Enable `ALL` logging level for `org.apache.spark.executor.Executor` logger to see what happens inside.

Add the following line to `conf/log4j.properties`:

[source,plaintext]
----
log4j.logger.org.apache.spark.executor.Executor=ALL
----

Refer to xref:ROOT:spark-logging.adoc[Logging].

== [[internal-properties]] Internal Properties

=== [[executorSource]] ExecutorSource

xref:executor:ExecutorSource.adoc[]

=== [[heartbeatFailures]] heartbeatFailures

=== [[maxDirectResultSize]] maxDirectResultSize

=== [[maxResultSize]] maxResultSize

Used exclusively when TaskRunner is requested to xref:executor:TaskRunner.adoc#run[run] (and creates a serialized `ByteBuffer` result that is a `IndirectTaskResult`)
