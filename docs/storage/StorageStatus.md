== [[StorageStatus]] StorageStatus

`StorageStatus` is a developer API that Spark uses to pass "just enough" information about registered xref:storage:BlockManager.adoc[BlockManagers] in a Spark application between Spark services (mostly for monitoring purposes like link:spark-webui.adoc[web UI] or xref:ROOT:SparkListener.adoc[]s).

[NOTE]
====
There are two ways to access `StorageStatus` about all the known `BlockManagers` in a Spark application:

* xref:ROOT:SparkContext.adoc#getExecutorStorageStatus[SparkContext.getExecutorStorageStatus]

* Being a xref:ROOT:SparkListener.adoc[] and intercepting xref:ROOT:SparkListener.adoc#onBlockManagerAdded[onBlockManagerAdded] and xref:ROOT:SparkListener.adoc#onBlockManagerRemoved[onBlockManagerRemoved] events
====

`StorageStatus` is <<creating-instance, created>> when:

* `BlockManagerMasterEndpoint` xref:storage:BlockManagerMasterEndpoint.adoc#storageStatus[is requested for storage status] (of every xref:storage:BlockManager.adoc[BlockManager] in a Spark application)

* `StorageStatusListener` link:spark-webui-StorageStatusListener.adoc#onBlockManagerAdded[gets notified about a new `BlockManager`] (in a Spark application)

[[internal-registries]]
.StorageStatus's Internal Registries and Counters
[cols="1,2",options="header",width="100%"]
|===
| Name
| Description

| [[_nonRddBlocks]] `_nonRddBlocks`
| Lookup table of `BlockIds` per `BlockId`.

Used when...FIXME

| [[_rddBlocks]] `_rddBlocks`
| Lookup table of `BlockIds` with `BlockStatus` per RDD id.

Used when...FIXME
|===

=== [[updateStorageInfo]] `updateStorageInfo` Internal Method

[source, scala]
----
updateStorageInfo(
  blockId: BlockId,
  newBlockStatus: BlockStatus): Unit
----

`updateStorageInfo`...FIXME

NOTE: `updateStorageInfo` is used when...FIXME

=== [[creating-instance]] Creating StorageStatus Instance

`StorageStatus` takes the following when created:

* [[blockManagerId]] xref:storage:BlockManagerId.adoc[]
* [[maxMem]] Maximum memory -- xref:storage:BlockManager.adoc#maxMemory[total available on-heap and off-heap memory for storage on the `BlockManager`]

`StorageStatus` initializes the <<internal-registries, internal registries and counters>>.

=== [[rddBlocksById]] Getting RDD Blocks For RDD -- `rddBlocksById` Method

[source, scala]
----
rddBlocksById(rddId: Int): Map[BlockId, BlockStatus]
----

`rddBlocksById` gives the blocks (as `BlockId` with their status as `BlockStatus`) that belong to `rddId` RDD.

[NOTE]
====
`rddBlocksById` is used when:

* `StorageStatusListener` link:spark-webui-StorageStatusListener.adoc#updateStorageStatus-unpersistedRDD[removes the RDD blocks of an unpersisted RDD].

* `AllRDDResource` does `getRDDStorageInfo`
* `StorageUtils` does `getRddBlockLocations`
====

=== [[removeBlock]] Removing Block (From Internal Registries) -- `removeBlock` Internal Method

[source, scala]
----
removeBlock(blockId: BlockId): Option[BlockStatus]
----

`removeBlock` removes `blockId` from <<_rddBlocks, _rddBlocks>> registry and returns it.

Internally, `removeBlock` <<updateStorageInfo, updates block status>> of `blockId` (to be empty, i.e. removed).

`removeBlock` branches off per the type of xref:storage:BlockId.adoc[], i.e. `RDDBlockId` or not.

For a `RDDBlockId`, `removeBlock` finds the RDD in <<_rddBlocks, _rddBlocks>> and removes the `blockId`. `removeBlock` removes the RDD (from <<_rddBlocks, _rddBlocks>>) completely, if there are no more blocks registered.

For a non-``RDDBlockId``, `removeBlock` removes `blockId` from <<_nonRddBlocks, _nonRddBlocks>> registry.

NOTE: `removeBlock` is used when `StorageStatusListener` link:spark-webui-StorageStatusListener.adoc#updateStorageStatus-unpersistedRDD[removes RDD blocks for an unpersisted RDD] or link:spark-webui-StorageStatusListener.adoc#updateStorageStatus-executor[updates storage status for an executor].

=== [[addBlock]] Registering Status of Data Block -- `addBlock` Method

[source, scala]
----
addBlock(
  blockId: BlockId,
  blockStatus: BlockStatus): Unit
----

`addBlock`...FIXME

NOTE: `addBlock` is used when...FIXME

=== [[getBlock]] `getBlock` Method

[source, scala]
----
getBlock(blockId: BlockId): Option[BlockStatus]
----

`getBlock`...FIXME

NOTE: `getBlock` is used when...FIXME
