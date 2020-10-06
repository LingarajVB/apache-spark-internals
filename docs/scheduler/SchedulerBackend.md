= [[SchedulerBackend]] SchedulerBackend

*SchedulerBackend* is an abstraction of <<implementations, task scheduling systems>> that can <<reviveOffers, revive resource offers>> from cluster managers.

SchedulerBackend abstraction allows TaskSchedulerImpl to use variety of cluster managers (with their own resource offers and task scheduling modes).

NOTE: Being a scheduler backend system assumes a http://mesos.apache.org/[Apache Mesos]-like scheduling model in which "an application" gets *resource offers* as machines become available so it is possible to launch tasks on them. Once required resource allocation is obtained, the scheduler backend can start executors.

== [[implementations]] Direct Implementations and Extensions

[cols="30m,70",options="header",width="100%"]
|===
| SchedulerBackend
| Description

| xref:scheduler:CoarseGrainedSchedulerBackend.adoc[CoarseGrainedSchedulerBackend]
| [[CoarseGrainedSchedulerBackend]] Base SchedulerBackend for coarse-grained scheduling systems

| xref:spark-local:spark-LocalSchedulerBackend.adoc[LocalSchedulerBackend]
| [[LocalSchedulerBackend]] Spark local

| MesosFineGrainedSchedulerBackend
| [[MesosFineGrainedSchedulerBackend]] Fine-grained scheduling system for Apache Mesos

|===

== [[start]] Starting SchedulerBackend

[source, scala]
----
start(): Unit
----

Starts the SchedulerBackend

Used when TaskSchedulerImpl is requested to xref:scheduler:TaskSchedulerImpl.adoc#start[start]

== [[contract]] Contract

[cols="30m,70",options="header",width="100%"]
|===
| Method
| Description

| applicationAttemptId
a| [[applicationAttemptId]]

[source, scala]
----
applicationAttemptId(): Option[String]
----

*Execution attempt ID* of the Spark application

Default: `None` (undefined)

Used exclusively when `TaskSchedulerImpl` is requested for the xref:scheduler:TaskSchedulerImpl.adoc#applicationAttemptId[execution attempt ID of a Spark application]

| applicationId
a| [[applicationId]][[appId]]

[source, scala]
----
applicationId(): String
----

*Unique identifier* of the Spark Application

Default: `spark-application-[currentTimeMillis]`

Used exclusively when `TaskSchedulerImpl` is requested for the xref:scheduler:TaskSchedulerImpl.adoc#applicationId[unique identifier of a Spark application]

| defaultParallelism
a| [[defaultParallelism]]

[source, scala]
----
defaultParallelism(): Int
----

*Default parallelism*, i.e. a hint for the number of tasks in stages while sizing jobs

Used exclusively when `TaskSchedulerImpl` is requested for the xref:scheduler:TaskSchedulerImpl.adoc#defaultParallelism[default parallelism]

| getDriverLogUrls
a| [[getDriverLogUrls]]

[source, scala]
----
getDriverLogUrls: Option[Map[String, String]]
----

*Driver log URLs*

Default: `None` (undefined)

Used exclusively when `SparkContext` is requested to xref:ROOT:SparkContext.adoc#postApplicationStart[postApplicationStart]

| isReady
a| [[isReady]]

[source, scala]
----
isReady(): Boolean
----

Controls whether the xref:scheduler:SchedulerBackend.adoc[SchedulerBackend] is ready (`true`) or not (`false`)

Default: `true`

Used exclusively when `TaskSchedulerImpl` is requested to xref:scheduler:TaskSchedulerImpl.adoc#waitBackendReady[wait until scheduling backend is ready]

| killTask
a| [[killTask]]

[source, scala]
----
killTask(
  taskId: Long,
  executorId: String,
  interruptThread: Boolean,
  reason: String): Unit
----

Kills a given task

Default: Throws an `UnsupportedOperationException`

Used when:

* `TaskSchedulerImpl` is requested to xref:scheduler:TaskSchedulerImpl.adoc#killTaskAttempt[killTaskAttempt] and xref:scheduler:TaskSchedulerImpl.adoc#killAllTaskAttempts[killAllTaskAttempts]

* `TaskSetManager` is requested to xref:scheduler:TaskSetManager.adoc#handleSuccessfulTask[handle a successful task attempt]

| maxNumConcurrentTasks
a| [[maxNumConcurrentTasks]]

[source, scala]
----
maxNumConcurrentTasks(): Int
----

*Maximum number of concurrent tasks* that can be launched now

Used exclusively when `SparkContext` is requested to xref:ROOT:SparkContext.adoc#maxNumConcurrentTasks[maxNumConcurrentTasks]

| reviveOffers
a| [[reviveOffers]]

[source, scala]
----
reviveOffers(): Unit
----

Handles resource allocation offers (from the scheduling system)

Used when `TaskSchedulerImpl` is requested to:

* xref:scheduler:TaskSchedulerImpl.adoc#submitTasks[Submit tasks (from a TaskSet)]

* xref:scheduler:TaskSchedulerImpl.adoc#statusUpdate[Handle a task status update]

* xref:scheduler:TaskSchedulerImpl.adoc#handleFailedTask[Notify the TaskSetManager that a task has failed]

* xref:scheduler:TaskSchedulerImpl.adoc#checkSpeculatableTasks[Check for speculatable tasks]

* xref:scheduler:TaskSchedulerImpl.adoc#executorLost[Handle a lost executor event]

| stop
a| [[stop]]

[source, scala]
----
stop(): Unit
----

Stops the SchedulerBackend

Used when:

* `TaskSchedulerImpl` is requested to xref:scheduler:TaskSchedulerImpl.adoc#stop[stop]

* `MesosCoarseGrainedSchedulerBackend` is requested to <<spark-mesos/spark-mesos-MesosCoarseGrainedSchedulerBackend.adoc#stopSchedulerBackend, stopSchedulerBackend>>

|===
