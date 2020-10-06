= SparkListener

*SparkListener* is a mechanism in Spark to intercept events from the Spark scheduler that are emitted over the course of execution of a Spark application.

SparkListener extends <<SparkListenerInterface, SparkListenerInterface>> with all the _callback methods_ being no-op/do-nothing.

Spark <<builtin-implementations, relies on `SparkListeners` internally>> to manage communication between internal components in the distributed environment for a Spark application, e.g. link:spark-webui.adoc[web UI], xref:spark-history-server:EventLoggingListener.adoc[event persistence] (for History Server), link:spark-ExecutorAllocationManager.adoc[dynamic allocation of executors], link:spark-HeartbeatReceiver.adoc[keeping track of executors (using `HeartbeatReceiver`)] and others.

You can develop your own custom SparkListener and register it using xref:ROOT:SparkContext.adoc#addSparkListener[SparkContext.addSparkListener] method or xref:ROOT:configuration-properties.adoc#spark.extraListeners[spark.extraListeners] configuration property.

With SparkListener you can focus on Spark events of your liking and process a subset of all scheduling events.

[TIP]
====
Enable `INFO` logging level for `org.apache.spark.SparkContext` logger to see when custom Spark listeners are registered.

```
INFO SparkContext: Registered listener org.apache.spark.scheduler.StatsReportListener
```

See xref:ROOT:SparkContext.adoc[].
====

== [[SparkListenerInterface]] SparkListenerInterface -- Internal Contract for Spark Listeners

`SparkListenerInterface` is an `private[spark]` contract for Spark listeners to intercept events from the Spark scheduler.

NOTE: <<SparkListener, SparkListener>> and <<SparkFirehoseListener, SparkFirehoseListener>> Spark listeners are direct implementations of `SparkListenerInterface` contract to help developing more sophisticated Spark listeners.

.SparkListenerInterface Methods
[cols="1,1,2",options="header",width="100%"]
|===
| Method
| Event
| Reason

| `onApplicationEnd`
| [[SparkListenerApplicationEnd]] `SparkListenerApplicationEnd` | `SparkContext` does `postApplicationEnd`

| [[onApplicationStart]] `onApplicationStart`
| [[SparkListenerApplicationStart]] `SparkListenerApplicationStart`
| `SparkContext` does `postApplicationStart`

| [[onBlockManagerAdded]] `onBlockManagerAdded`
| [[SparkListenerBlockManagerAdded]] `SparkListenerBlockManagerAdded`
| `BlockManagerMasterEndpoint` xref:storage:BlockManagerMasterEndpoint.adoc#register[has registered a `BlockManager`].

| [[onBlockManagerRemoved]] `onBlockManagerRemoved`
| [[SparkListenerBlockManagerRemoved]] `SparkListenerBlockManagerRemoved`
| `BlockManagerMasterEndpoint` xref:storage:BlockManagerMasterEndpoint.adoc#removeBlockManager[has removed a `BlockManager`] (which is when...FIXME)

| [[onBlockUpdated]] `onBlockUpdated`
| [[SparkListenerBlockUpdated]] `SparkListenerBlockUpdated`
| `BlockManagerMasterEndpoint` receives a xref:storage:BlockManagerMasterEndpoint.adoc#UpdateBlockInfo[UpdateBlockInfo] event (which is when `BlockManager` xref:storage:BlockManager.adoc#tryToReportBlockStatus[reports a block status update to driver]).

| `onEnvironmentUpdate`
| [[SparkListenerEnvironmentUpdate]] `SparkListenerEnvironmentUpdate`
| `SparkContext` does `postEnvironmentUpdate`.

| `onExecutorMetricsUpdate`
| [[SparkListenerExecutorMetricsUpdate]] `SparkListenerExecutorMetricsUpdate`
|

| `onExecutorAdded`
| [[SparkListenerExecutorAdded]] `SparkListenerExecutorAdded`
| [[onExecutorAdded]] `DriverEndpoint` RPC endpoint (of `CoarseGrainedSchedulerBackend`) xref:scheduler:CoarseGrainedSchedulerBackend-DriverEndpoint.adoc#RegisterExecutor[receives `RegisterExecutor` message], `MesosFineGrainedSchedulerBackend` does `resourceOffers`, and `LocalSchedulerBackendEndpoint` starts.

| [[onExecutorBlacklisted]] `onExecutorBlacklisted`
| [[SparkListenerExecutorBlacklisted]] `SparkListenerExecutorBlacklisted`
| FIXME

| [[onExecutorRemoved]] `onExecutorRemoved`
| [[SparkListenerExecutorRemoved]] `SparkListenerExecutorRemoved`
| `DriverEndpoint` RPC endpoint (of `CoarseGrainedSchedulerBackend`) does
xref:scheduler:CoarseGrainedSchedulerBackend-DriverEndpoint.adoc#removeExecutor[removeExecutor] and `MesosFineGrainedSchedulerBackend` does `removeExecutor`.

| [[onExecutorUnblacklisted]] `onExecutorUnblacklisted`
| [[SparkListenerExecutorUnblacklisted]] `SparkListenerExecutorUnblacklisted`
| FIXME

| `onJobEnd`
| [[SparkListenerJobEnd]] `SparkListenerJobEnd`
| `DAGScheduler` does `cleanUpAfterSchedulerStop`, `handleTaskCompletion`, `failJobAndIndependentStages`, and markMapStageJobAsFinished.

| [[onJobStart]] `onJobStart`
| [[SparkListenerJobStart]] `SparkListenerJobStart`
| `DAGScheduler` handles xref:scheduler:DAGSchedulerEventProcessLoop.adoc#handleJobSubmitted[JobSubmitted] and xref:scheduler:DAGSchedulerEventProcessLoop.adoc#handleMapStageSubmitted[MapStageSubmitted] messages

| [[onNodeBlacklisted]] `onNodeBlacklisted`
| [[SparkListenerNodeBlacklisted]] `SparkListenerNodeBlacklisted`
| FIXME

| [[onNodeUnblacklisted]] `onNodeUnblacklisted`
| [[SparkListenerNodeUnblacklisted]] `SparkListenerNodeUnblacklisted`
| FIXME

| [[onStageCompleted]] `onStageCompleted`
| [[SparkListenerStageCompleted]] `SparkListenerStageCompleted`
| `DAGScheduler` xref:scheduler:DAGScheduler.adoc#markStageAsFinished[marks a stage as finished].

| [[onStageSubmitted]] `onStageSubmitted`
| [[SparkListenerStageSubmitted]] `SparkListenerStageSubmitted`
| `DAGScheduler` xref:scheduler:DAGScheduler.adoc#submitMissingTasks[submits the missing tasks of a stage (in a Spark job)].

| [[onTaskEnd]] `onTaskEnd`
| [[SparkListenerTaskEnd]] `SparkListenerTaskEnd`
| `DAGScheduler` xref:scheduler:DAGScheduler.adoc#handleTaskCompletion[handles a task completion]

| `onTaskGettingResult`
| [[SparkListenerTaskGettingResult]] `SparkListenerTaskGettingResult`
| `DAGScheduler` xref:scheduler:DAGSchedulerEventProcessLoop.adoc#handleGetTaskResult[handles `GettingResultEvent` event]

| [[onTaskStart]] `onTaskStart`
| [[SparkListenerTaskStart]] `SparkListenerTaskStart`
| `DAGScheduler` is informed that a xref:scheduler:DAGSchedulerEventProcessLoop.adoc#handleBeginEvent[task is about to start].

| [[onUnpersistRDD]] `onUnpersistRDD`
| [[SparkListenerUnpersistRDD]] `SparkListenerUnpersistRDD`
| `SparkContext` xref:ROOT:SparkContext.adoc#unpersistRDD[unpersists an RDD], i.e. removes RDD blocks from `BlockManagerMaster` (that can be triggered xref:ROOT:SparkContext.adoc#unpersist[explicitly] or xref:core:ContextCleaner.adoc#doCleanupRDD[implicitly]).

| [[onOtherEvent]] `onOtherEvent`
| [[SparkListenerEvent]] `SparkListenerEvent`
| Catch-all callback that is often used in Spark SQL to handle custom events.
|===

== [[builtin-implementations]] Built-In Spark Listeners

.Built-In Spark Listeners
[cols="1,2",options="header",width="100%"]
|===
| Spark Listener | Description
| xref:spark-history-server:EventLoggingListener.adoc[EventLoggingListener] | Logs JSON-encoded events to a file that can later be read by xref:spark-history-server:index.adoc[History Server]
| link:spark-SparkListener-StatsReportListener.adoc[StatsReportListener] |
| [[SparkFirehoseListener]] `SparkFirehoseListener` | Allows users to receive all <<SparkListenerEvent, SparkListenerEvent>> events by overriding the single `onEvent` method only.
| link:spark-SparkListener-ExecutorAllocationListener.adoc[ExecutorAllocationListener] |
| link:spark-HeartbeatReceiver.adoc[HeartbeatReceiver] |
| link:spark-streaming/spark-streaming-streaminglisteners.adoc#StreamingJobProgressListener[StreamingJobProgressListener] |
| link:spark-webui-executors-ExecutorsListener.adoc[ExecutorsListener] | Prepares information for link:spark-webui-executors.adoc[Executors tab] in link:spark-webui.adoc[web UI]
| link:spark-webui-StorageStatusListener.adoc[StorageStatusListener], link:spark-webui-RDDOperationGraphListener.adoc[RDDOperationGraphListener], link:spark-webui-EnvironmentListener.adoc[EnvironmentListener], link:spark-webui-BlockStatusListener.adoc[BlockStatusListener] and link:spark-webui-StorageListener.adoc[StorageListener] | For link:spark-webui.adoc[web UI]
| `SpillListener` |
| `ApplicationEventListener` |
| link:spark-sql-streaming-StreamingQueryListenerBus.adoc[StreamingQueryListenerBus] |
| link:spark-sql-SQLListener.adoc[SQLListener] / xref:spark-history-server:SQLHistoryListener.adoc[SQLHistoryListener] | Support for xref:spark-history-server:index.adoc[History Server]
| link:spark-streaming/spark-streaming-jobscheduler.adoc#StreamingListenerBus[StreamingListenerBus] |
| link:spark-webui-JobProgressListener.adoc[JobProgressListener] |
|===
