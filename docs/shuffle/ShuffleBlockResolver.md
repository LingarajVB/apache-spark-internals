= [[ShuffleBlockResolver]] ShuffleBlockResolver

*ShuffleBlockResolver* is an <<contract, abstraction>> of <<implementations, shuffle block resolvers>> that xref:storage:BlockManager.adoc[BlockManager] uses to <<getBlockData, retrieve a shuffle block data>> for a logical shuffle block identifier (i.e. map, reduce, and shuffle).

NOTE: Shuffle block data files are often referred to as *map outputs files*.

[[implementations]]
NOTE: xref:shuffle:IndexShuffleBlockResolver.adoc[IndexShuffleBlockResolver] is the default and only known ShuffleBlockResolver in Apache Spark.

[[contract]]
.ShuffleBlockResolver Contract
[cols="1m,3",options="header",width="100%"]
|===
| Method
| Description

| getBlockData
a| [[getBlockData]]

[source, scala]
----
getBlockData(
  blockId: ShuffleBlockId): ManagedBuffer
----

Retrieves the data (as a xref:network:ManagedBuffer.adoc[]) for the given xref:storage:BlockId.adoc#ShuffleBlockId[block] (a tuple of `shuffleId`, `mapId` and `reduceId`).

Used when `BlockManager` is requested to retrieve a xref:storage:BlockManager.adoc#getLocalBytes[block data from a local block manager] and xref:storage:BlockManager.adoc#getBlockData[block data]

| stop
a| [[stop]]

[source, scala]
----
stop(): Unit
----

Stops the `ShuffleBlockResolver`

Used when `SortShuffleManager` is requested to xref:SortShuffleManager.adoc#stop[stop]

|===
