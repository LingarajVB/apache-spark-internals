= ShuffleMapTask

ShuffleMapTask is one of the two types of xref:scheduler:Task.adoc[tasks] that, when <<runTask, executed>>, writes the result of executing a <<taskBinary, serialized task code>> over the records (of a <<partition, RDD partition>>) to the xref:shuffle:ShuffleManager.adoc[shuffle system] and returns a xref:scheduler:MapStatus.adoc[MapStatus] (information about the xref:storage:BlockManager.adoc[BlockManager] and estimated size of the result shuffle blocks).

.ShuffleMapTask and DAGScheduler
image::ShuffleMapTask.png[align="center"]

== [[creating-instance]] Creating Instance

ShuffleMapTask takes the following to be created:

* [[stageId]] Stage ID
* [[stageAttemptId]] Stage attempt ID
* <<taskBinary, Broadcast variable with a serialized task binary>>
* [[partition]] xref:rdd:spark-rdd-Partition.adoc[Partition]
* [[locs]] xref:scheduler:TaskLocation.adoc[TaskLocations]
* [[localProperties]] Task-specific local properties
* [[serializedTaskMetrics]] Serialized task metrics (`Array[Byte]`)
* [[jobId]] Optional xref:scheduler:spark-scheduler-ActiveJob.adoc[job ID] (default: `None`)
* [[appId]] Optional application ID (default: `None`)
* [[appAttemptId]] Optional application attempt ID (default: `None`)
* [[isBarrier]] isBarrier flag (default: `false`)

ShuffleMapTask is created when DAGScheduler is requested to xref:scheduler:DAGScheduler.adoc#submitMissingTasks[submit tasks for all missing partitions of a ShuffleMapStage].

== [[taskBinary]] Broadcast Variable and Serialized Task Binary

ShuffleMapTask is given a xref:ROOT:Broadcast.adoc[] with a reference to a serialized task binary (`Broadcast[Array[Byte]]`).

<<runTask, runTask>> expects that the serialized task binary is a tuple of an xref:rdd:RDD.adoc[RDD] and a xref:rdd:ShuffleDependency.adoc[ShuffleDependency].

== [[runTask]] Running Task

[source, scala]
----
runTask(
  context: TaskContext): MapStatus
----

runTask writes the result (_records_) of executing the <<taskBinary, serialized task code>> over the records (in the <<partition, RDD partition>>) to the xref:shuffle:ShuffleManager.adoc[shuffle system] and returns a xref:scheduler:MapStatus.adoc[MapStatus] (with the xref:storage:BlockManager.adoc[BlockManager] and an estimated size of the result shuffle blocks).

Internally, runTask requests the xref:core:SparkEnv.adoc[SparkEnv] for the new instance of xref:core:SparkEnv.adoc#closureSerializer[closure serializer] and requests it to xref:serializer:Serializer.adoc#deserialize[deserialize] the <<taskBinary, taskBinary>> (into a tuple of a xref:rdd:RDD.adoc[RDD] and a xref:rdd:ShuffleDependency.adoc[ShuffleDependency]).

runTask measures the xref:scheduler:Task.adoc#_executorDeserializeTime[thread] and xref:scheduler:Task.adoc#_executorDeserializeCpuTime[CPU] deserialization times.

runTask requests the xref:core:SparkEnv.adoc[SparkEnv] for the xref:core:SparkEnv.adoc#shuffleManager[ShuffleManager] and requests it for a xref:shuffle:ShuffleManager.adoc#getWriter[ShuffleWriter] (for the xref:rdd:ShuffleDependency.adoc#shuffleHandle[ShuffleHandle], the xref:scheduler:Task.adoc#partitionId[RDD partition], and the xref:scheduler:spark-TaskContext.adoc[TaskContext]).

runTask then requests the <<rdd, RDD>> for the xref:rdd:RDD.adoc#iterator[records] (of the <<partition, partition>>) that the `ShuffleWriter` is requested to xref:shuffle:ShuffleWriter.adoc#write[write out] (to the shuffle system).

In the end, runTask requests the `ShuffleWriter` to xref:shuffle:ShuffleWriter.adoc#stop[stop] (with the `success` flag on) and returns the xref:scheduler:MapStatus.adoc[shuffle map output status].

NOTE: This is the moment in ``Task``'s lifecycle (and its corresponding RDD) when a xref:rdd:index.adoc#iterator[RDD partition is computed] and in turn becomes a sequence of records (i.e. real data) on an executor.

In case of any exceptions, runTask requests the `ShuffleWriter` to xref:shuffle:ShuffleWriter.adoc#stop[stop] (with the `success` flag off) and (re)throws the exception.

runTask may also print out the following DEBUG message to the logs when the `ShuffleWriter` could not be xref:shuffle:ShuffleWriter.adoc#stop[stopped].

[source,plaintext]
----
Could not stop writer
----

runTask is part of xref:scheduler:Task.adoc#runTask[Task] abstraction.

== [[preferredLocations]] preferredLocations Method

[source, scala]
----
preferredLocations: Seq[TaskLocation]
----

preferredLocations simply returns the <<preferredLocs, preferredLocs>> internal property.

preferredLocations is part of xref:scheduler:Task.adoc#preferredLocations[Task] abstraction.

== [[logging]] Logging

Enable `ALL` logging level for `org.apache.spark.scheduler.ShuffleMapTask` logger to see what happens inside.

Add the following line to `conf/log4j.properties`:

[source,plaintext]
----
log4j.logger.org.apache.spark.scheduler.ShuffleMapTask=ALL
----

Refer to xref:ROOT:spark-logging.adoc[Logging].

== [[preferredLocs]] Preferred Locations

xref:scheduler:TaskLocation.adoc[TaskLocations] that are the unique entries in the given <<locs, locs>> with the only rule that when `locs` is not defined, it is empty, and no task location preferences are defined.

Initialized when ShuffleMapTask is <<creating-instance, created>>

Used exclusively when ShuffleMapTask is requested for the <<preferredLocations, preferred locations>>
