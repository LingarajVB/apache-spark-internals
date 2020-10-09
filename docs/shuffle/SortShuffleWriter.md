# SortShuffleWriter

`SortShuffleWriter` is a concrete [ShuffleWriter](ShuffleWriter.md) that is used when shuffle:SortShuffleManager.md#getWriter[`SortShuffleManager` returns a `ShuffleWriter` for `ShuffleHandle`] (and the more specialized shuffle:BypassMergeSortShuffleWriter.md[BypassMergeSortShuffleWriter] and shuffle:UnsafeShuffleWriter.md[UnsafeShuffleWriter] could not be used).

SortShuffleWriter is created when SortShuffleManager.md#getWriter[`SortShuffleManager` returns a `ShuffleWriter` for the fallback `BaseShuffleHandle`].

`SortShuffleWriter[K, V, C]` is a parameterized type with `K` keys, `V` values, and `C` combiner values.

== [[creating-instance]] Creating Instance

SortShuffleWriter takes the following to be created:

* [[shuffleBlockResolver]] shuffle:IndexShuffleBlockResolver.md[IndexShuffleBlockResolver]
* [[handle]] shuffle:spark-shuffle-BaseShuffleHandle.md[BaseShuffleHandle]
* [[mapId]] Map ID
* [[context]] scheduler:spark-TaskContext.md[TaskContext]

== [[mapStatus]] MapStatus

SortShuffleWriter uses an internal variable for a scheduler:MapStatus.md[MapStatus] after <<write, writing records>>.

<<write, Writing records>> itself does not return a value, SortShuffleWriter uses the variable when requested to <<stop, stop>> (which allows returning a MapStatus).

== [[write]] Writing Records (Into Shuffle Partitioned File In Disk Store)

[source, scala]
----
write(
  records: Iterator[Product2[K, V]]): Unit
----

write creates an shuffle:ExternalSorter.md[ExternalSorter] based on the shuffle:spark-shuffle-BaseShuffleHandle.md#dependency[ShuffleDependency] (of the <<handle, BaseShuffleHandle>>), namely the rdd:ShuffleDependency.md#mapSideCombine[Map-Size Partial Aggregation] flag. The ExternalSorter uses the aggregator and keyOrdering when the flag is enabled.

write requests the ExternalSorter to shuffle:ExternalSorter.md#insertAll[inserts all the records].

write requests the <<shuffleBlockResolver, IndexShuffleBlockResolver>> for a shuffle:IndexShuffleBlockResolver.md#getDataFile[shuffle output file] (for the shuffle and the <<mapId, map>> IDs) and creates a temporary file alongside the shuffle output file (in the same directory).

write creates a storage:BlockId.md#ShuffleBlockId[ShuffleBlockId] (for the shuffle and the <<mapId, map>> IDs).

write requests ExternalSorter to shuffle:ExternalSorter.md#writePartitionedFile[write all the records (previously inserted in) into the temporary partitioned file in the disk store]. ExternalSorter returns the length of every partition.

write requests <<shuffleBlockResolver, IndexShuffleBlockResolver>> to shuffle:IndexShuffleBlockResolver.md#writeIndexFileAndCommit[write an index file and commit] (with the shuffle and the <<mapId, map>> IDs, the temporary shuffle output file).

write creates a scheduler:MapStatus.md[MapStatus] (with the storage:BlockManager.md#shuffleServerId[location of the shuffle server] that serves the shuffle files and the sizes of the shuffle partitions). The `MapStatus` is later available as the <<mapStatus, mapStatus>> internal attribute.

write does not handle exceptions so when they occur, they will break the processing.

In the end, write deletes the temporary shuffle output file. write prints out the following ERROR message to the logs if the file count not be deleted:

```
Error while deleting temp file [path]
```

write is part of the shuffle:ShuffleWriter.md#write[ShuffleWriter] abstraction.

== [[stop]] Closing SortShuffleWriter (and Calculating MapStatus)

[source, scala]
----
stop(
  success: Boolean): Option[MapStatus]
----

stop turns <<stopping, stopping>> flag on and returns the internal <<mapStatus, mapStatus>> if the input `success` is enabled.

Otherwise, when <<stopping, stopping>> flag is already enabled or the input `success` is disabled, stop returns no `MapStatus` (i.e. `None`).

In the end, stop requests the ExternalSorter to shuffle:ExternalSorter.md#stop[stop] and increments the shuffle write time task metrics.

stop is part of the shuffle:ShuffleWriter.md#contract[ShuffleWriter] abstraction.

== [[shouldBypassMergeSort]] Requirements of BypassMergeSortShuffleHandle (as ShuffleHandle)

[source, scala]
----
shouldBypassMergeSort(
  conf: SparkConf,
  dep: ShuffleDependency[_, _, _]): Boolean
----

shouldBypassMergeSort returns `true` when all of the following hold:

. No map-side aggregation (the rdd:ShuffleDependency.md#mapSideCombine[mapSideCombine] flag of the given rdd:ShuffleDependency.md[ShuffleDependency] is off)

. rdd:Partitioner.md#numPartitions[Number of partitions] (of the rdd:ShuffleDependency.md#partitioner[Partitioner] of the given rdd:ShuffleDependency.md[ShuffleDependency]) is not greater than ROOT:configuration-properties.md#spark.shuffle.sort.bypassMergeThreshold[spark.shuffle.sort.bypassMergeThreshold] configuration property

Otherwise, shouldBypassMergeSort does not hold (i.e. `false`).

shouldBypassMergeSort is used when SortShuffleManager is requested to shuffle:SortShuffleManager.md#registerShuffle[register a shuffle (and creates a ShuffleHandle)].

== [[logging]] Logging

Enable `ALL` logging level for `org.apache.spark.shuffle.sort.SortShuffleWriter` logger to see what happens inside.

Add the following line to `conf/log4j.properties`:

[source]
----
log4j.logger.org.apache.spark.shuffle.sort.SortShuffleWriter=ALL
----

Refer to ROOT:spark-logging.md[Logging].

== [[internal-properties]] Internal Properties

[cols="30m,70",options="header",width="100%"]
|===
| Name
| Description

| [[stopping]] `stopping`
| Internal flag to mark that <<stop, SortShuffleWriter is closed>>.

|===
