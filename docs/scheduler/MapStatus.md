= MapStatus

*MapStatus* is an <<contract, abstraction>> of <<implementations, shuffle map output statuses>> that are metadata of shuffle map outputs with <<getSizeForBlock, estimated size for the reduce block>> and <<location, block location>>.

MapStatus is <<apply, created>> as the result of xref:scheduler:ShuffleMapTask.adoc#runTask[executing a ShuffleMapTask] (after a xref:shuffle:ShuffleManager.adoc#getWriter[ShuffleWriter] has xref:shuffle:ShuffleWriter.adoc#stop[finished writing partition records successfully]).

After a xref:scheduler:ShuffleMapTask.adoc#runTask[ShuffleMapTask has finished execution successfully], `DAGScheduler` is requested to xref:scheduler:DAGScheduler.adoc#handleTaskCompletion[handleTaskCompletion] (of the `ShuffleMapTask`) that requests the xref:scheduler:DAGScheduler.adoc#mapOutputTracker[MapOutputTrackerMaster] to xref:scheduler:MapOutputTrackerMaster.adoc#registerMapOutput[register the MapStatus].

== [[contract]] Contract

[cols="30,70",options="header",width="100%"]
|===
| Method
| Description

| getSizeForBlock
a| [[getSizeForBlock]]

[source, scala]
----
getSizeForBlock(reduceId: Int): Long
----

Estimated size for the reduce block (in bytes)

Used when:

* `MapOutputTrackerMaster` is requested for a xref:scheduler:MapOutputTrackerMaster.adoc#getStatistics[MapOutputStatistics] and xref:scheduler:MapOutputTrackerMaster.adoc#getLocationsWithLargestOutputs[locations with largest number of shuffle map outputs]

* `MapOutputTracker` object is requested to xref:scheduler:MapOutputTracker.adoc#convertMapStatuses[convert MapStatuses To BlockManagerIds with ShuffleBlockIds and Their Sizes]

| location
a| [[location]]

[source, scala]
----
location: BlockManagerId
----

Block location, i.e. the <<BlockManager.adoc#, BlockManager>> where a `ShuffleMapTask` ran and the result is stored.

Used when:

* `DAGScheduler` is requested to xref:scheduler:DAGScheduler.adoc#handleTaskCompletion[handleTaskCompletion] (of a `ShuffleMapTask`)

* `ShuffleStatus` is requested to `removeMapOutput` and `removeOutputsByFilter`

* `MapOutputTrackerMaster` is requested for xref:scheduler:MapOutputTrackerMaster.adoc#getLocationsWithLargestOutputs[locations with largest number of shuffle map outputs]

* `MapOutputTracker` object is requested to xref:scheduler:MapOutputTracker.adoc#convertMapStatuses[convert MapStatuses To BlockManagerIds with ShuffleBlockIds and Their Sizes]

|===

== [[implementations]] MapStatuses

[cols="30,70",options="header",width="100%"]
|===
| MapStatus
| Description

| CompressedMapStatus
| [[CompressedMapStatus]] Default MapStatus that compresses the <<getSizeForBlock, estimated map output size>> to 8 bits (`Byte`) for efficient reporting

| HighlyCompressedMapStatus
| [[HighlyCompressedMapStatus]] Stores the average size of non-empty blocks, and a compressed bitmap for tracking which blocks are empty. Used when the number of partitions is above the <<minPartitionsToUseHighlyCompressMapStatus, spark.shuffle.minNumPartitionsToHighlyCompress>> threshold

|===

== [[minPartitionsToUseHighlyCompressMapStatus]] Minimum Number of Partitions Threshold

MapStatus object uses xref:ROOT:configuration-properties.adoc#spark.shuffle.minNumPartitionsToHighlyCompress[spark.shuffle.minNumPartitionsToHighlyCompress] internal configuration property for the *minimum number of partitions* threshold to create a <<HighlyCompressedMapStatus, HighlyCompressedMapStatus>> when requested to <<apply, create a MapStatus>>.

== [[apply]] Creating MapStatus

[source, scala]
----
apply(
  loc: BlockManagerId,
  uncompressedSizes: Array[Long]): MapStatus
----

apply creates a concrete <<MapStatus, MapStatus>> per the size of the given `uncompressedSizes` array:

* <<HighlyCompressedMapStatus, HighlyCompressedMapStatus>> when above the <<minPartitionsToUseHighlyCompressMapStatus, minPartitionsToUseHighlyCompressMapStatus>> threshold

* <<CompressedMapStatus, CompressedMapStatus>> otherwise

apply is used when:

* SortShuffleWriter is requested to xref:shuffle:SortShuffleWriter.adoc#write[write records (into shuffle partitioned file in disk store)]

* BypassMergeSortShuffleWriter is requested to xref:shuffle:BypassMergeSortShuffleWriter.adoc#write[write records (into one single shuffle block data file)]

* UnsafeShuffleWriter is requested to xref:shuffle:UnsafeShuffleWriter.adoc#closeAndWriteOutput[close the internal resources and write out merged spill files]
