# ShuffleManager

`ShuffleManager` is an abstraction of <<implementations, shuffle systems>> that manage shuffle data.

`ShuffleManager` is selected using [spark.shuffle.manager](configuration-properties.md#spark.shuffle.manager) configuration property.

`ShuffleManager` is used to create a [BlockManager](../storage/BlockManager.md#shuffleManager).

== [[implementations]] Available ShuffleManagers

xref:shuffle:SortShuffleManager.adoc[SortShuffleManager] is the default and only known ShuffleManager in Apache Spark.

== [[SparkEnv]] Accessing ShuffleManager using SparkEnv

The driver and executor access the ShuffleManager instance using xref:core:SparkEnv.adoc#shuffleManager[SparkEnv.shuffleManager].

[source, scala]
----
val shuffleManager = SparkEnv.get.shuffleManager
----

== [[getReader]] Getting ShuffleReader for ShuffleHandle

[source, scala]
----
getReader[K, C](
  handle: ShuffleHandle,
  startPartition: Int,
  endPartition: Int,
  context: TaskContext): ShuffleReader[K, C]
----

Gives xref:shuffle:spark-shuffle-ShuffleReader.adoc[ShuffleReader] to read shuffle data in the xref:shuffle:spark-shuffle-ShuffleHandle.adoc[ShuffleHandle]

Used when the following RDDs are requested to xref:rdd:RDD.adoc#compute[compute a partition]:

* xref:rdd:spark-rdd-CoGroupedRDD.adoc[CoGroupedRDD]

* xref:rdd:ShuffledRDD.adoc[ShuffledRDD]

* xref:rdd:spark-rdd-SubtractedRDD.adoc[SubtractedRDD]

== [[getWriter]] Getting ShuffleWriter for ShuffleHandle

[source, scala]
----
getWriter[K, V](
  handle: ShuffleHandle,
  mapId: Int,
  context: TaskContext): ShuffleWriter[K, V]
----

Gives xref:shuffle:ShuffleWriter.adoc[ShuffleWriter] to write shuffle data in the xref:shuffle:spark-shuffle-ShuffleHandle.adoc[ShuffleHandle]

Used exclusively when ShuffleMapTask is requested to xref:scheduler:ShuffleMapTask.adoc#runTask[run] (and requests the xref:shuffle:ShuffleWriter.adoc[ShuffleWriter] to write records for a partition)

== [[registerShuffle]] Registering Shuffle of ShuffleDependency (and Getting ShuffleHandle)

[source, scala]
----
registerShuffle[K, V, C](
  shuffleId: Int,
  numMaps: Int,
  dependency: ShuffleDependency[K, V, C]): ShuffleHandle
----

Registers a shuffle (by the given shuffleId and xref:rdd:ShuffleDependency.adoc[ShuffleDependency]) and returns a xref:shuffle:spark-shuffle-ShuffleHandle.adoc[ShuffleHandle]

Used when ShuffleDependency is xref:rdd:ShuffleDependency.adoc#shuffleHandle[created] (and registers with the shuffle system)

== [[shuffleBlockResolver]] Getting ShuffleBlockResolver

[source, scala]
----
shuffleBlockResolver: ShuffleBlockResolver
----

Gives xref:shuffle:ShuffleBlockResolver.adoc[ShuffleBlockResolver] of the shuffle system

Used when:

* SortShuffleManager is requested for a xref:shuffle:SortShuffleManager.adoc#getWriter[ShuffleWriter for a ShuffleHandle], to xref:shuffle:SortShuffleManager.adoc#unregisterShuffle[unregister a shuffle] and xref:shuffle:SortShuffleManager.adoc#stop[stop]

* BlockManager is requested to xref:storage:BlockManager.adoc#getBlockData[get shuffle data] and xref:storage:BlockManager.adoc#getLocalBytes[getLocalBytes]

== [[stop]] Stopping ShuffleManager

[source, scala]
----
stop(): Unit
----

Stops the shuffle system

Used when SparkEnv is requested to xref:core:SparkEnv.adoc#stop[stop]

== [[unregisterShuffle]] Unregistering Shuffle

[source, scala]
----
unregisterShuffle(
  shuffleId: Int): Boolean
----

Unregisters a given shuffle

Used when BlockManagerSlaveEndpoint is requested to xref:storage:BlockManagerSlaveEndpoint.adoc#RemoveShuffle[handle a RemoveShuffle message]
