= TaskRunner

TaskRunner is a *thread of execution* (a {java-javadoc-url}/java/lang/Runnable.html[java.lang.Runnable]) of a <<taskDescription, task>>.

.Executor creates and runs TaskRunner
image::TaskRunner.png[align="center"]

Once <<creating-instance, created>>, TaskRunner can be <<run, started>> and optionally <<kill, killed>> (which starts and kills a given <<taskDescription, task>>, respectively).

TaskRunner is an internal class of the xref:executor:Executor.adoc[] class and has access to its internals (properties and methods).

== [[creating-instance]] Creating Instance

TaskRunner takes the following to be created:

* [[execBackend]] xref:executor:ExecutorBackend.adoc[] (that manages the parent xref:executor:Executor.adoc[])
* [[taskDescription]] xref:scheduler:spark-scheduler-TaskDescription.adoc[TaskDescription]

TaskRunner is created when Executor is requested to xref:executor:Executor.adoc#launchTask[launch a task].

== [[threadName]] Thread Name

TaskRunner uses *Executor task launch worker for task [taskId]* as the thread name.

== [[run]] Running Task

[source, scala]
----
run(): Unit
----

When executed, run initializes <<threadId, threadId>> as the current thread identifier (using Java's link:++https://docs.oracle.com/javase/8/docs/api/java/lang/Thread.html#getId--++[Thread])

run then sets the name of the current thread as <<threadName, threadName>> (using Java's link:++https://docs.oracle.com/javase/8/docs/api/java/lang/Thread.html#setName-java.lang.String-++[Thread]).

run xref:memory:TaskMemoryManager.adoc#creating-instance[creates a `TaskMemoryManager`] (using the current xref:memory:MemoryManager.adoc[MemoryManager] and <<taskId, taskId>>).

NOTE: run uses xref:core:SparkEnv.adoc#memoryManager[`SparkEnv` to access the current `MemoryManager`].

run starts tracking the time to deserialize a task.

run sets the current thread's context classloader (with <<replClassLoader, replClassLoader>>).

run xref:serializer:Serializer.adoc#newInstance[creates a closure `Serializer`].

NOTE: run uses `SparkEnv` xref:core:SparkEnv.adoc#closureSerializer[to access the current closure `Serializer`].

You should see the following INFO message in the logs:

```
Running [taskName] (TID [taskId])
```

run xref:executor:ExecutorBackend.adoc#statusUpdate[notifies `ExecutorBackend`] that <<taskId, taskId>> is in `TaskState.RUNNING` state.

NOTE: run uses `ExecutorBackend` that was specified when TaskRunner <<creating-instance, was created>>.

run <<computeTotalGcTime, computes `startGCTime`>>.

run <<updateDependencies, updates dependencies>>.

NOTE: run uses xref:spark-scheduler-TaskDescription.adoc[TaskDescription] that is specified when TaskRunner <<creating-instance, is created>>.

run xref:serializer:SerializerInstance.adoc#deserialize[deserializes the task] (using the context class loader) and sets its `localProperties` and `TaskMemoryManager`. run sets the <<task, task>> internal reference to hold the deserialized task.

NOTE: run uses `TaskDescription` xref:spark-scheduler-TaskDescription.adoc#serializedTask[to access serialized task].

If <<killed, killed>> flag is enabled, run throws a `TaskKilledException`.

You should see the following DEBUG message in the logs:

```
Task [taskId]'s epoch is [task.epoch]
```

run xref:scheduler:MapOutputTracker.adoc#updateEpoch[notifies `MapOutputTracker` about the epoch of the task].

NOTE: run uses xref:core:SparkEnv.adoc#mapOutputTracker[`SparkEnv` to access the current `MapOutputTracker`].

run records the current time as the task's start time (as `taskStart`).

run xref:scheduler:Task.adoc#run[runs the task] (with `taskAttemptId` as <<taskId, taskId>>, `attemptNumber` from `TaskDescription`, and `metricsSystem` as the current xref:metrics:spark-metrics-MetricsSystem.adoc[MetricsSystem]).

NOTE: run uses xref:core:SparkEnv.adoc#metricsSystem[`SparkEnv` to access the current `MetricsSystem`].

NOTE: The task runs inside a "monitored" block (i.e. `try-finally` block) to detect any memory and lock leaks after the task's run finishes regardless of the final outcome - the computed value or an exception thrown.

After the task's run has finished (inside the "finally" block of the "monitored" block), run xref:BlockManager.adoc#releaseAllLocksForTask[requests `BlockManager` to release all locks of the task] (for the task's <<taskId, taskId>>). The locks are later used for lock leak detection.

run then xref:memory:TaskMemoryManager.adoc#cleanUpAllAllocatedMemory[requests `TaskMemoryManager` to clean up allocated memory] (that helps finding memory leaks).

If run detects memory leak of the managed memory (i.e. the memory freed is greater than `0`) and xref:ROOT:configuration-properties.adoc#spark.unsafe.exceptionOnMemoryLeak[spark.unsafe.exceptionOnMemoryLeak] Spark property is enabled (it is not by default) and no exception was reported while the task ran, run reports a `SparkException`:

```
Managed memory leak detected; size = [freedMemory] bytes, TID = [taskId]
```

Otherwise, if xref:ROOT:configuration-properties.adoc#spark.unsafe.exceptionOnMemoryLeak[spark.unsafe.exceptionOnMemoryLeak] is disabled, you should see the following ERROR message in the logs instead:

```
Managed memory leak detected; size = [freedMemory] bytes, TID = [taskId]
```

NOTE: If run detects a memory leak, it leads to a `SparkException` or ERROR message in the logs.

If run detects lock leaking (i.e. the number of locks released) and xref:ROOT:configuration-properties.adoc#spark.storage.exceptionOnPinLeak[spark.storage.exceptionOnPinLeak] configuration property is enabled (it is not by default) and no exception was reported while the task ran, run reports a `SparkException`:

```
[releasedLocks] block locks were not released by TID = [taskId]:
[releasedLocks separated by comma]
```

Otherwise, if xref:ROOT:configuration-properties.adoc#spark.storage.exceptionOnPinLeak[spark.storage.exceptionOnPinLeak] is disabled or the task reported an exception, you should see the following INFO message in the logs instead:

```
[releasedLocks] block locks were not released by TID = [taskId]:
[releasedLocks separated by comma]
```

NOTE: If run detects any lock leak, it leads to a `SparkException` or INFO message in the logs.

Rigth after the "monitored" block, run records the current time as the task's finish time (as `taskFinish`).

If the xref:scheduler:Task.adoc#kill[task was killed] (while it was running), run reports a `TaskKilledException` (and the TaskRunner exits).

run xref:serializer:Serializer.adoc#newInstance[creates a `Serializer`] and xref:serializer:Serializer.adoc#serialize[serializes the task's result]. run measures the time to serialize the result.

NOTE: run uses `SparkEnv` xref:core:SparkEnv.adoc#serializer[to access the current `Serializer`]. `SparkEnv` was specified when xref:executor:Executor.adoc#creating-instance[the owning `Executor` was created].

IMPORTANT: This is when `TaskExecutor` serializes the computed value of a task to be sent back to the driver.

run records the xref:scheduler:Task.adoc#metrics[task metrics]:

* xref:executor:TaskMetrics.adoc#setExecutorDeserializeTime[executorDeserializeTime]
* xref:executor:TaskMetrics.adoc#setExecutorDeserializeCpuTime[executorDeserializeCpuTime]
* xref:executor:TaskMetrics.adoc#setExecutorRunTime[executorRunTime]
* xref:executor:TaskMetrics.adoc#setExecutorCpuTime[executorCpuTime]
* xref:executor:TaskMetrics.adoc#setJvmGCTime[jvmGCTime]
* xref:executor:TaskMetrics.adoc#setResultSerializationTime[resultSerializationTime]

run xref:scheduler:Task.adoc#collectAccumulatorUpdates[collects the latest values of internal and external accumulators used in the task].

run creates a xref:spark-scheduler-TaskResult.adoc#DirectTaskResult[DirectTaskResult] (with the serialized result and the latest values of accumulators).

run xref:serializer:Serializer.adoc#serialize[serializes the `DirectTaskResult`] and gets the byte buffer's limit.

NOTE: A serialized `DirectTaskResult` is Java's https://docs.oracle.com/javase/8/docs/api/java/nio/ByteBuffer.html[java.nio.ByteBuffer].

run selects the proper serialized version of the result before xref:executor:ExecutorBackend.adoc#statusUpdate[sending it to `ExecutorBackend`].

run branches off based on the serialized `DirectTaskResult` byte buffer's limit.

When xref:executor:Executor.adoc#maxResultSize[maxResultSize] is greater than `0` and the serialized `DirectTaskResult` buffer limit exceeds it, the following WARN message is displayed in the logs:

```
Finished [taskName] (TID [taskId]). Result is larger than maxResultSize ([resultSize] > [maxResultSize]), dropping it.
```

TIP: Read about xref:ROOT:configuration-properties.adoc#spark.driver.maxResultSize[spark.driver.maxResultSize].

```
$ ./bin/spark-shell -c spark.driver.maxResultSize=1m

scala> sc.version
res0: String = 2.0.0-SNAPSHOT

scala> sc.getConf.get("spark.driver.maxResultSize")
res1: String = 1m

scala> sc.range(0, 1024 * 1024 + 10, 1).collect
WARN Executor: Finished task 4.0 in stage 0.0 (TID 4). Result is larger than maxResultSize (1031.4 KB > 1024.0 KB), dropping it.
...
ERROR TaskSetManager: Total size of serialized results of 1 tasks (1031.4 KB) is bigger than spark.driver.maxResultSize (1024.0 KB)
...
org.apache.spark.SparkException: Job aborted due to stage failure: Total size of serialized results of 1 tasks (1031.4 KB) is bigger than spark.driver.maxResultSize (1024.0 KB)
  at org.apache.spark.scheduler.DAGScheduler.org$apache$spark$scheduler$DAGScheduler$$failJobAndIndependentStages(DAGScheduler.scala:1448)
...
```

In this case, run creates a xref:spark-scheduler-TaskResult.adoc#IndirectTaskResult[IndirectTaskResult] (with a `TaskResultBlockId` for the task's <<taskId, taskId>> and `resultSize`) and xref:serializer:Serializer.adoc#serialize[serializes it].

[[run-result-sent-via-blockmanager]]
When `maxResultSize` is not positive or `resultSize` is smaller than `maxResultSize` but greater than xref:executor:Executor.adoc#maxDirectResultSize[maxDirectResultSize], run creates a `TaskResultBlockId` for the task's <<taskId, taskId>> and xref:BlockManager.adoc#putBytes[stores the serialized `DirectTaskResult` in `BlockManager`] (as the `TaskResultBlockId` with `MEMORY_AND_DISK_SER` storage level).

You should see the following INFO message in the logs:

```
Finished [taskName] (TID [taskId]). [resultSize] bytes result sent via BlockManager)
```

In this case, run creates a xref:spark-scheduler-TaskResult.adoc#IndirectTaskResult[IndirectTaskResult] (with a `TaskResultBlockId` for the task's <<taskId, taskId>> and `resultSize`) and xref:serializer:Serializer.adoc#serialize[serializes it].

NOTE: The difference between the two above cases is that the result is dropped or stored in `BlockManager` with `MEMORY_AND_DISK_SER` storage level.

When the two cases above do not hold, you should see the following INFO message in the logs:

```
Finished [taskName] (TID [taskId]). [resultSize] bytes result sent to driver
```

run uses the serialized `DirectTaskResult` byte buffer as the final `serializedResult`.

NOTE: The final `serializedResult` is either a xref:spark-scheduler-TaskResult.adoc#IndirectTaskResult[IndirectTaskResult] (possibly with the block stored in `BlockManager`) or a xref:spark-scheduler-TaskResult.adoc#DirectTaskResult[DirectTaskResult].

run xref:executor:ExecutorBackend.adoc#statusUpdate[notifies `ExecutorBackend`] that <<taskId, taskId>> is in `TaskState.FINISHED` state with the serialized result and removes <<taskId, taskId>> from the owning executor's xref:executor:Executor.adoc#runningTasks[ runningTasks] registry.

NOTE: run uses `ExecutorBackend` that is specified when TaskRunner <<creating-instance, is created>>.

NOTE: TaskRunner is Java's https://docs.oracle.com/javase/8/docs/api/java/lang/Runnable.html[Runnable] and the contract requires that once a TaskRunner has completed execution it must not be restarted.

When run catches a exception while executing the task, run acts according to its type (as presented in the following "run's Exception Cases" table and the following sections linked from the table).

.run's Exception Cases, TaskState and Serialized ByteBuffer
[cols="1,1,2",options="header",width="100%"]
|===
| Exception Type
| TaskState
| Serialized ByteBuffer

| <<run-FetchFailedException, FetchFailedException>>
| `FAILED`
| `TaskFailedReason`

| <<run-TaskKilledException, TaskKilledException>>
| `KILLED`
| `TaskKilled`

| <<run-InterruptedException, InterruptedException>>
| `KILLED`
| `TaskKilled`

| <<run-CommitDeniedException, CommitDeniedException>>
| `FAILED`
| `TaskFailedReason`

| <<run-Throwable, Throwable>>
| `FAILED`
| `ExceptionFailure`

|===

run is part of {java-javadoc-url}/java/lang/Runnable.html[java.lang.Runnable] contract.

=== [[run-FetchFailedException]] FetchFailedException

When xref:shuffle:FetchFailedException.adoc[FetchFailedException] is reported while running a task, run <<setTaskFinishedAndClearInterruptStatus, setTaskFinishedAndClearInterruptStatus>>.

run xref:shuffle:FetchFailedException.adoc#toTaskFailedReason[requests `FetchFailedException` for the `TaskFailedReason`], serializes it and xref:executor:ExecutorBackend.adoc#statusUpdate[notifies `ExecutorBackend` that the task has failed] (with <<taskId, taskId>>, `TaskState.FAILED`, and a serialized reason).

NOTE: `ExecutorBackend` was specified when <<creating-instance, TaskRunner was created>>.

NOTE:  run uses a closure xref:serializer:Serializer.adoc[Serializer] to serialize the failure reason. The `Serializer` was created before run ran the task.

=== [[run-TaskKilledException]] TaskKilledException

When `TaskKilledException` is reported while running a task, you should see the following INFO message in the logs:

```
Executor killed [taskName] (TID [taskId]), reason: [reason]
```

run then <<setTaskFinishedAndClearInterruptStatus, setTaskFinishedAndClearInterruptStatus>> and xref:executor:ExecutorBackend.adoc#statusUpdate[notifies `ExecutorBackend` that the task has been killed] (with <<taskId, taskId>>, `TaskState.KILLED`, and a serialized `TaskKilled` object).

=== [[run-InterruptedException]] InterruptedException (with Task Killed)

When `InterruptedException` is reported while running a task, and the task has been killed, you should see the following INFO message in the logs:

```
Executor interrupted and killed [taskName] (TID [taskId]), reason: [killReason]
```

run then <<setTaskFinishedAndClearInterruptStatus, setTaskFinishedAndClearInterruptStatus>> and xref:executor:ExecutorBackend.adoc#statusUpdate[notifies `ExecutorBackend` that the task has been killed] (with <<taskId, taskId>>, `TaskState.KILLED`, and a serialized `TaskKilled` object).

NOTE: The difference between this `InterruptedException` and <<run-TaskKilledException, TaskKilledException>> is the INFO message in the logs.

=== [[run-CommitDeniedException]] CommitDeniedException

When `CommitDeniedException` is reported while running a task, run <<setTaskFinishedAndClearInterruptStatus, setTaskFinishedAndClearInterruptStatus>> and xref:executor:ExecutorBackend.adoc#statusUpdate[notifies `ExecutorBackend` that the task has failed] (with <<taskId, taskId>>, `TaskState.FAILED`, and a serialized `TaskKilled` object).

NOTE: The difference between this `CommitDeniedException` and <<run-FetchFailedException, FetchFailedException>> is just the reason being sent to `ExecutorBackend`.

=== [[run-Throwable]] Throwable

When run catches a `Throwable`, you should see the following ERROR message in the logs (followed by the exception).

```
Exception in [taskName] (TID [taskId])
```

run then records the following task metrics (only when <<task, Task>> is available):

* xref:executor:TaskMetrics.adoc#setExecutorRunTime[executorRunTime]
* xref:executor:TaskMetrics.adoc#setJvmGCTime[jvmGCTime]

run then xref:scheduler:Task.adoc#collectAccumulatorUpdates[collects the latest values of internal and external accumulators] (with `taskFailed` flag enabled to inform that the collection is for a failed task).

Otherwise, when <<task, Task>> is not available, the accumulator collection is empty.

run converts the task accumulators to collection of `AccumulableInfo`, creates a `ExceptionFailure` (with the accumulators), and xref:serializer:Serializer.adoc#serialize[serializes them].

NOTE: run uses a closure xref:serializer:Serializer.adoc[Serializer] to serialize the `ExceptionFailure`.

CAUTION: FIXME Why does run create `new ExceptionFailure(t, accUpdates).withAccums(accums)`, i.e. accumulators occur twice in the object.

run <<setTaskFinishedAndClearInterruptStatus, setTaskFinishedAndClearInterruptStatus>> and xref:executor:ExecutorBackend.adoc#statusUpdate[notifies `ExecutorBackend` that the task has failed] (with <<taskId, taskId>>, `TaskState.FAILED`, and the serialized `ExceptionFailure`).

run may also trigger `SparkUncaughtExceptionHandler.uncaughtException(t)` if this is a fatal error.

NOTE: The difference between this most `Throwable` case and other `FAILED` cases (i.e. <<run-FetchFailedException, FetchFailedException>> and <<run-CommitDeniedException, CommitDeniedException>>) is just the serialized `ExceptionFailure` vs a reason being sent to `ExecutorBackend`, respectively.

== [[kill]] Killing Task

[source, scala]
----
kill(
  interruptThread: Boolean,
  reason: String): Unit
----

`kill` marks the TaskRunner as <<killed, killed>> and xref:scheduler:Task.adoc#kill[kills the task] (if available and not <<finished, finished>> already).

NOTE: `kill` passes the input `interruptThread` on to the task itself while killing it.

When executed, you should see the following INFO message in the logs:

```
Executor is trying to kill [taskName] (TID [taskId]), reason: [reason]
```

NOTE: <<killed, killed>> flag is checked periodically in <<run, run>> to stop executing the task. Once killed, the task will eventually stop.

== [[collectAccumulatorsAndResetStatusOnFailure]] collectAccumulatorsAndResetStatusOnFailure Method

[source, scala]
----
collectAccumulatorsAndResetStatusOnFailure(
  taskStartTime: Long): (Seq[AccumulatorV2[_, _]], Seq[AccumulableInfo])
----

collectAccumulatorsAndResetStatusOnFailure...FIXME

collectAccumulatorsAndResetStatusOnFailure is used when TaskRunner is requested to <<run, run>>.

== [[hasFetchFailure]] hasFetchFailure Method

[source, scala]
----
hasFetchFailure: Boolean
----

hasFetchFailure...FIXME

hasFetchFailure is used when TaskRunner is requested to <<run, run>>.

== [[setTaskFinishedAndClearInterruptStatus]] setTaskFinishedAndClearInterruptStatus Method

[source, scala]
----
setTaskFinishedAndClearInterruptStatus(): Unit
----

setTaskFinishedAndClearInterruptStatus...FIXME

setTaskFinishedAndClearInterruptStatus is used when TaskRunner is requested to <<run, run>>.

== [[logging]] Logging

Enable `ALL` logging level for `org.apache.spark.executor.Executor` logger to see what happens inside (since TaskRunner is an internal class of Executor).

Add the following line to `conf/log4j.properties`:

[source,plaintext]
----
log4j.logger.org.apache.spark.executor.Executor=ALL
----

Refer to xref:ROOT:spark-logging.adoc[Logging].

== [[internal-properties]] Internal Properties

=== [[finished]][[isFinished]] finished Flag

finished flag says whether the <<taskDescription, task>> has finished (`true`) or not (`false`)

Default: `false`

Enabled (`true`) after TaskRunner has been requested to <<setTaskFinishedAndClearInterruptStatus, setTaskFinishedAndClearInterruptStatus>>

Used when TaskRunner is requested to <<kill, kill the task>>

=== [[reasonIfKilled]] reasonIfKilled

Reason to <<kill, kill the task>> (and avoid <<run, executing it>>)

Default: `(empty)` (`None`)

=== [[startGCTime]] startGCTime Timestamp

Timestamp (which is really the xref:executor:Executor.adoc#computeTotalGcTime[total amount of time this Executor JVM process has already spent in garbage collection]) that is used to mark the GC "zero" time (when <<run, run the task>>) and then compute the *JVM GC time metric* when:

* TaskRunner is requested to <<collectAccumulatorsAndResetStatusOnFailure, collectAccumulatorsAndResetStatusOnFailure>> and <<run, run>>

* `Executor` is requested to xref:executor:Executor.adoc#reportHeartBeat[reportHeartBeat]

=== [[task]] Task

Deserialized xref:scheduler:Task.adoc[task] to execute

Used when:

* TaskRunner is requested to <<kill, kill the task>>, <<collectAccumulatorsAndResetStatusOnFailure, collectAccumulatorsAndResetStatusOnFailure>>, <<run, run the task>>, <<hasFetchFailure, hasFetchFailure>>

* `Executor` is requested to xref:executor:Executor.adoc#reportHeartBeat[reportHeartBeat]

=== [[taskId]] Task Id

The <<spark-scheduler-TaskDescription.adoc#taskId, task ID>> (of the <<taskDescription, TaskDescription>>)

Used when:

* TaskRunner is requested to <<run, run>> (to create a xref:memory:TaskMemoryManager.adoc[TaskMemoryManager] and serialize a `IndirectTaskResult` for a large task result) and <<kill, kill>> the task and for the <<threadName, threadName>>

* `Executor` is requested to xref:executor:Executor.adoc#reportHeartBeat[reportHeartBeat]

=== [[taskName]] Task Name

The <<spark-scheduler-TaskDescription.adoc#name, name of the task>> (of the <<taskDescription, TaskDescription>>) that is used exclusively for <<logging, logging>> purposes when TaskRunner is requested to <<run, run>> and <<kill, kill>> the task

=== [[threadId]][[getThreadId]] Thread Id

Current thread ID

Default: `-1`

Set immediately when TaskRunner is requested to <<run, run the task>> and used exclusively when `TaskReaper` is requested for the thread info of the current thread (aka _thread dump_)
