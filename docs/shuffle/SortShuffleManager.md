= SortShuffleManager

*SortShuffleManager* is the default and only known xref:ShuffleManager.adoc[ShuffleManager] in Apache Spark (with the short name `sort` or `tungsten-sort`).

NOTE: Use xref:ROOT:configuration-properties.adoc#spark.shuffle.manager[spark.shuffle.manager] configuration property to activate custom xref:ShuffleManager.adoc[ShuffleManager].

SortShuffleManager is <<creating-instance, created>> when SparkEnv is xref:core:SparkEnv.adoc#ShuffleManager[created] (on the driver and executors at the very beginning of a Spark application's lifecycle).

.SortShuffleManager and SparkEnv (Driver and Executors)
image::SortShuffleManager.png[align="center"]

== [[creating-instance]] Creating Instance

[[conf]]
SortShuffleManager takes a single xref:ROOT:SparkConf.adoc[SparkConf] to be created.

== [[MAX_SHUFFLE_OUTPUT_PARTITIONS_FOR_SERIALIZED_MODE]] Maximum Number of Partition Identifiers

SortShuffleManager allows for `(1 << 24)` partition identifiers that can be encoded (i.e. `16777216`).

== [[numMapsForShuffle]] numMapsForShuffle

Lookup table with the number of mappers producing the output for a shuffle (as {java-javadoc-url}/java/util/concurrent/ConcurrentHashMap.html[java.util.concurrent.ConcurrentHashMap])

== [[shuffleBlockResolver]] IndexShuffleBlockResolver

[source, scala]
----
shuffleBlockResolver: ShuffleBlockResolver
----

shuffleBlockResolver is an xref:shuffle:IndexShuffleBlockResolver.adoc[IndexShuffleBlockResolver] that is created immediately when SortShuffleManager is.

shuffleBlockResolver is used when SortShuffleManager is requested for a <<getWriter, ShuffleWriter for a given partition>>, to <<unregisterShuffle, unregister a shuffle metadata>> and <<stop, stop>>.

shuffleBlockResolver is part of the xref:shuffle:ShuffleManager.adoc#shuffleBlockResolver[ShuffleManager] abstraction.

== [[getWriter]] Getting ShuffleWriter For Partition and ShuffleHandle

[source, scala]
----
getWriter[K, V](
  handle: ShuffleHandle,
  mapId: Int,
  context: TaskContext): ShuffleWriter[K, V]
----

getWriter registers the given ShuffleHandle (by the xref:spark-shuffle-ShuffleHandle.adoc#shuffleId[shuffleId] and xref:spark-shuffle-BaseShuffleHandle.adoc#numMaps[numMaps]) in the <<numMapsForShuffle, numMapsForShuffle>> internal registry unless already done.

NOTE: getWriter expects that the input ShuffleHandle is of type xref:spark-shuffle-BaseShuffleHandle.adoc[BaseShuffleHandle]. Moreover, getWriter further expects that in two (out of three cases) it is a more specialized xref:IndexShuffleBlockResolver.adoc[IndexShuffleBlockResolver].

getWriter then creates a new ShuffleWriter based on the type of the given ShuffleHandle.

[cols="2",options="header",width="100%"]
|===
| ShuffleHandle
| ShuffleWriter

| xref:shuffle:SerializedShuffleHandle.adoc[SerializedShuffleHandle]
| xref:shuffle:UnsafeShuffleWriter.adoc[UnsafeShuffleWriter]

| xref:shuffle:BypassMergeSortShuffleHandle.adoc[BypassMergeSortShuffleHandle]
| xref:shuffle:BypassMergeSortShuffleWriter.adoc[BypassMergeSortShuffleWriter]

| xref:shuffle:spark-shuffle-BaseShuffleHandle.adoc[BaseShuffleHandle]
| xref:shuffle:SortShuffleWriter.adoc[SortShuffleWriter]

|===

getWriter is part of the xref:shuffle:ShuffleManager.adoc#getWriter[ShuffleManager] abstraction.

== [[unregisterShuffle]] Unregistering Shuffle

[source, scala]
----
unregisterShuffle(
  shuffleId: Int): Boolean
----

unregisterShuffle tries to remove the given `shuffleId` from the <<numMapsForShuffle, numMapsForShuffle>> internal registry.

If the given `shuffleId` was registered, unregisterShuffle requests the <<shuffleBlockResolver, IndexShuffleBlockResolver>> to <<IndexShuffleBlockResolver.adoc#removeDataByMap, remove the shuffle index and data files>> one by one (up to the number of mappers producing the output for the shuffle).

unregisterShuffle is part of the xref:shuffle:ShuffleManager.adoc#unregisterShuffle[ShuffleManager] abstraction.

== [[registerShuffle]] Creating ShuffleHandle (For ShuffleDependency)

[source, scala]
----
registerShuffle[K, V, C](
  shuffleId: Int,
  numMaps: Int,
  dependency: ShuffleDependency[K, V, C]): ShuffleHandle
----

CAUTION: FIXME Copy the conditions

`registerShuffle` returns a new `ShuffleHandle` that can be one of the following:

1. xref:shuffle:BypassMergeSortShuffleHandle.adoc[BypassMergeSortShuffleHandle] (with `ShuffleDependency[K, V, V]`) when xref:shuffle:SortShuffleWriter.adoc#shouldBypassMergeSort[shouldBypassMergeSort] condition holds.

2. xref:shuffle:SerializedShuffleHandle.adoc[SerializedShuffleHandle] (with `ShuffleDependency[K, V, V]`) when <<canUseSerializedShuffle, canUseSerializedShuffle condition holds>>.

3. xref:shuffle:spark-shuffle-BaseShuffleHandle.adoc[BaseShuffleHandle]

registerShuffle is part of the xref:shuffle:ShuffleManager.adoc#registerShuffle[ShuffleManager] abstraction.

== [[getReader]] Creating BlockStoreShuffleReader For ShuffleHandle And Reduce Partitions

[source, scala]
----
getReader[K, C](
  handle: ShuffleHandle,
  startPartition: Int,
  endPartition: Int,
  context: TaskContext): ShuffleReader[K, C]
----

getReader returns a new xref:shuffle:BlockStoreShuffleReader.adoc[BlockStoreShuffleReader] passing all the input parameters on to it.

getReader assumes that the input `ShuffleHandle` is of type xref:shuffle:spark-shuffle-BaseShuffleHandle.adoc[BaseShuffleHandle].

getReader is part of the xref:shuffle:ShuffleManager.adoc#getReader[ShuffleManager] abstraction.

== [[stop]] Stopping SortShuffleManager

[source, scala]
----
stop(): Unit
----

stop simply requests the <<shuffleBlockResolver, IndexShuffleBlockResolver>> to xref:shuffle:IndexShuffleBlockResolver.adoc#stop[stop] (which actually does nothing).

stop is part of the xref:shuffle:ShuffleManager.adoc#stop[ShuffleManager] abstraction.

== [[canUseSerializedShuffle]] Requirements of SerializedShuffleHandle (as ShuffleHandle)

[source, scala]
----
canUseSerializedShuffle(
  dependency: ShuffleDependency[_, _, _]): Boolean
----

canUseSerializedShuffle returns `true` when all of the following hold:

. xref:rdd:ShuffleDependency.adoc#serializer[Serializer] (of the given xref:rdd:ShuffleDependency.adoc[ShuffleDependency]) xref:serializer:Serializer.adoc#supportsRelocationOfSerializedObjects[supports relocation of serialized objects]

. No map-side aggregation (the xref:rdd:ShuffleDependency.adoc#mapSideCombine[mapSideCombine] flag of the given xref:rdd:ShuffleDependency.adoc[ShuffleDependency] is off)

. xref:rdd:Partitioner.adoc#numPartitions[Number of partitions] (of the xref:rdd:ShuffleDependency.adoc#partitioner[Partitioner] of the given xref:rdd:ShuffleDependency.adoc[ShuffleDependency]) is not greater than the <<MAX_SHUFFLE_OUTPUT_PARTITIONS_FOR_SERIALIZED_MODE, supported maximum number>> (i.e. `(1 << 24) - 1`, i.e. `16777215`)

canUseSerializedShuffle prints out the following DEBUG message to the logs:

[source,plaintext]
----
Can use serialized shuffle for shuffle [shuffleId]
----

Otherwise, canUseSerializedShuffle does not hold and prints out one of the following DEBUG messages:

[source,plaintext]
----
Can't use serialized shuffle for shuffle [id] because the serializer, [name], does not support object relocation

Can't use serialized shuffle for shuffle [id] because an aggregator is defined

Can't use serialized shuffle for shuffle [id] because it has more than [number] partitions
----

shouldBypassMergeSort is used when SortShuffleManager is requested to xref:shuffle:SortShuffleManager.adoc#registerShuffle[register a shuffle (and creates a ShuffleHandle)].

== [[logging]] Logging

Enable `ALL` logging level for `org.apache.spark.shuffle.sort.SortShuffleManager` logger to see what happens inside.

Add the following line to `conf/log4j.properties`:

[source,plaintext]
----
log4j.logger.org.apache.spark.shuffle.sort.SortShuffleManager=ALL
----

Refer to xref:ROOT:spark-logging.adoc[Logging].
