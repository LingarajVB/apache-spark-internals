= BlockManagerMasterEndpoint -- BlockManagerMaster RPC Endpoint
:navtitle: BlockManagerMasterEndpoint

*BlockManagerMasterEndpoint* is a xref:rpc:RpcEndpoint.adoc#ThreadSafeRpcEndpoint[ThreadSafeRpcEndpoint] for xref:storage:BlockManagerMaster.adoc[BlockManagerMaster].

BlockManagerMasterEndpoint is registered under *BlockManagerMaster* name.

BlockManagerMasterEndpoint tracks status of the xref:storage:BlockManager.adoc[BlockManagers] (on the executors) in a Spark application.

== [[creating-instance]] Creating Instance

BlockManagerMasterEndpoint takes the following to be created:

* [[rpcEnv]] xref:rpc:RpcEnv.adoc[]
* [[isLocal]] Flag whether BlockManagerMasterEndpoint works in local or cluster mode
* [[conf]] xref:ROOT:SparkConf.adoc[]
* [[listenerBus]] xref:scheduler:LiveListenerBus.adoc[]

BlockManagerMasterEndpoint is created for the xref:core:SparkEnv.adoc#create[SparkEnv] on the driver (to create a xref:storage:BlockManagerMaster.adoc[] for a xref:storage:BlockManager.adoc#master[BlockManager]).

When created, BlockManagerMasterEndpoint prints out the following INFO message to the logs:

[source,plaintext]
----
BlockManagerMasterEndpoint up
----

== [[messages]][[receiveAndReply]] Messages

As an xref:rpc:RpcEndpoint.adoc[], BlockManagerMasterEndpoint handles RPC messages.

=== [[BlockManagerHeartbeat]] BlockManagerHeartbeat

[source, scala]
----
BlockManagerHeartbeat(
  blockManagerId: BlockManagerId)
----

When received, BlockManagerMasterEndpoint...FIXME

Posted when...FIXME

=== [[GetLocations]] GetLocations

[source, scala]
----
GetLocations(
  blockId: BlockId)
----

When received, BlockManagerMasterEndpoint replies with the <<getLocations, locations>> of `blockId`.

Posted when xref:BlockManagerMaster.adoc#getLocations-block[`BlockManagerMaster` requests the block locations of a single block].

=== [[GetLocationsAndStatus]] GetLocationsAndStatus

[source, scala]
----
GetLocationsAndStatus(
  blockId: BlockId)
----

When received, BlockManagerMasterEndpoint...FIXME

Posted when...FIXME

=== [[GetLocationsMultipleBlockIds]] GetLocationsMultipleBlockIds

[source, scala]
----
GetLocationsMultipleBlockIds(
  blockIds: Array[BlockId])
----

When received, BlockManagerMasterEndpoint replies with the <<getLocationsMultipleBlockIds, getLocationsMultipleBlockIds>> for the given xref:storage:BlockId.adoc[].

Posted when xref:BlockManagerMaster.adoc#getLocations[`BlockManagerMaster` requests the block locations for multiple blocks].

=== [[GetPeers]] GetPeers

[source, scala]
----
GetPeers(
  blockManagerId: BlockManagerId)
----

When received, BlockManagerMasterEndpoint replies with the <<getPeers, peers>> of `blockManagerId`.

*Peers* of a xref:storage:BlockManager.adoc[BlockManager] are the other BlockManagers in a cluster (except the driver's BlockManager). Peers are used to know the available executors in a Spark application.

Posted when xref:BlockManagerMaster.adoc#getPeers[`BlockManagerMaster` requests the peers of a `BlockManager`].

=== [[GetExecutorEndpointRef]] GetExecutorEndpointRef

[source, scala]
----
GetExecutorEndpointRef(
  executorId: String)
----

When received, BlockManagerMasterEndpoint...FIXME

Posted when...FIXME

=== [[GetMemoryStatus]] GetMemoryStatus

[source, scala]
----
GetMemoryStatus
----

When received, BlockManagerMasterEndpoint...FIXME

Posted when...FIXME

=== [[GetStorageStatus]] GetStorageStatus

[source, scala]
----
GetStorageStatus
----

When received, BlockManagerMasterEndpoint...FIXME

Posted when...FIXME

=== [[GetBlockStatus]] GetBlockStatus

[source, scala]
----
GetBlockStatus(
  blockId: BlockId,
  askSlaves: Boolean = true)
----

When received, BlockManagerMasterEndpoint is requested to <<blockStatus, blockStatus>>.

Posted when...FIXME

=== [[GetMatchingBlockIds]] GetMatchingBlockIds

[source, scala]
----
GetMatchingBlockIds(
  filter: BlockId => Boolean,
  askSlaves: Boolean = true)
----

When received, BlockManagerMasterEndpoint...FIXME

Posted when...FIXME

=== [[HasCachedBlocks]] HasCachedBlocks

[source, scala]
----
HasCachedBlocks(
  executorId: String)
----

When received, BlockManagerMasterEndpoint...FIXME

Posted when...FIXME

=== [[RegisterBlockManager]] RegisterBlockManager

[source,scala]
----
RegisterBlockManager(
  blockManagerId: BlockManagerId,
  maxOnHeapMemSize: Long,
  maxOffHeapMemSize: Long,
  sender: RpcEndpointRef)
----

When received, BlockManagerMasterEndpoint is requested to <<register, register the BlockManager>> (by the given xref:storage:BlockManagerId.adoc[]).

Posted when BlockManagerMaster is requested to xref:storage:BlockManagerMaster.adoc#registerBlockManager[register a BlockManager]

=== [[RemoveRdd]] RemoveRdd

[source, scala]
----
RemoveRdd(
  rddId: Int)
----

When received, BlockManagerMasterEndpoint...FIXME

Posted when...FIXME

=== [[RemoveShuffle]] RemoveShuffle

[source, scala]
----
RemoveShuffle(
  shuffleId: Int)
----

When received, BlockManagerMasterEndpoint...FIXME

Posted when...FIXME

=== [[RemoveBroadcast]] RemoveBroadcast

[source, scala]
----
RemoveBroadcast(
  broadcastId: Long,
  removeFromDriver: Boolean = true)
----

When received, BlockManagerMasterEndpoint...FIXME

Posted when...FIXME

=== [[RemoveBlock]] RemoveBlock

[source, scala]
----
RemoveBlock(
  blockId: BlockId)
----

When received, BlockManagerMasterEndpoint...FIXME

Posted when...FIXME

=== [[RemoveExecutor]] RemoveExecutor

[source, scala]
----
RemoveExecutor(
  execId: String)
----

When received, BlockManagerMasterEndpoint <<removeExecutor, executor `execId` is removed>> and the response `true` sent back.

Posted when xref:BlockManagerMaster.adoc#removeExecutor[`BlockManagerMaster` removes an executor].

=== [[StopBlockManagerMaster]] StopBlockManagerMaster

[source, scala]
----
StopBlockManagerMaster
----

When received, BlockManagerMasterEndpoint...FIXME

Posted when...FIXME

=== [[UpdateBlockInfo]] UpdateBlockInfo

[source, scala]
----
UpdateBlockInfo(
  blockManagerId: BlockManagerId,
  blockId: BlockId,
  storageLevel: StorageLevel,
  memSize: Long,
  diskSize: Long)
----

When received, BlockManagerMasterEndpoint...FIXME

Posted when BlockManagerMaster is requested to xref:storage:BlockManagerMaster.adoc#updateBlockInfo[handle a block status update (from BlockManager on an executor)].

== [[storageStatus]] storageStatus Internal Method

[source,scala]
----
storageStatus: Array[StorageStatus]
----

storageStatus...FIXME

storageStatus is used when BlockManagerMasterEndpoint is requested to handle <<GetStorageStatus, GetStorageStatus>> message.

== [[getLocationsMultipleBlockIds]] getLocationsMultipleBlockIds Internal Method

[source,scala]
----
getLocationsMultipleBlockIds(
  blockIds: Array[BlockId]): IndexedSeq[Seq[BlockManagerId]]
----

getLocationsMultipleBlockIds...FIXME

getLocationsMultipleBlockIds is used when BlockManagerMasterEndpoint is requested to handle <<GetLocationsMultipleBlockIds, GetLocationsMultipleBlockIds>> message.

== [[removeShuffle]] removeShuffle Internal Method

[source,scala]
----
removeShuffle(
  shuffleId: Int): Future[Seq[Boolean]]
----

removeShuffle...FIXME

removeShuffle is used when BlockManagerMasterEndpoint is requested to handle <<RemoveShuffle, RemoveShuffle>> message.

== [[getPeers]] getPeers Internal Method

[source, scala]
----
getPeers(
  blockManagerId: BlockManagerId): Seq[BlockManagerId]
----

getPeers finds all the registered `BlockManagers` (using <<blockManagerInfo, blockManagerInfo>> internal registry) and checks if the input `blockManagerId` is amongst them.

If the input `blockManagerId` is registered, getPeers returns all the registered `BlockManagers` but the one on the driver and `blockManagerId`.

Otherwise, getPeers returns no `BlockManagers`.

NOTE: *Peers* of a xref:storage:BlockManager.adoc[BlockManager] are the other BlockManagers in a cluster (except the driver's BlockManager). Peers are used to know the available executors in a Spark application.

getPeers is used when BlockManagerMasterEndpoint is requested to handle <<GetPeers, GetPeers>> message.

== [[register]] register Internal Method

[source, scala]
----
register(
  idWithoutTopologyInfo: BlockManagerId,
  maxOnHeapMemSize: Long,
  maxOffHeapMemSize: Long,
  slaveEndpoint: RpcEndpointRef): BlockManagerId
----

register registers a xref:storage:BlockManager.adoc[] (based on the given xref:storage:BlockManagerId.adoc[]) in the <<blockManagerIdByExecutor, blockManagerIdByExecutor>> and <<blockManagerInfo, blockManagerInfo>> registries and posts a SparkListenerBlockManagerAdded message (to the <<listenerBus, LiveListenerBus>>).

NOTE: The input `maxMemSize` is the xref:storage:BlockManager.adoc#maxMemory[total available on-heap and off-heap memory for storage on a `BlockManager`].

NOTE: Registering a `BlockManager` can only happen once for an executor (identified by `BlockManagerId.executorId` in <<blockManagerIdByExecutor, blockManagerIdByExecutor>> internal registry).

If another `BlockManager` has earlier been registered for the executor, you should see the following ERROR message in the logs:

[source,plaintext]
----
Got two different block manager registrations on same executor - will replace old one [oldId] with new one [id]
----

And then <<removeExecutor, executor is removed>>.

register prints out the following INFO message to the logs:

[source,plaintext]
----
Registering block manager [hostPort] with [bytes] RAM, [id]
----

The `BlockManager` is recorded in the internal registries:

* <<blockManagerIdByExecutor, blockManagerIdByExecutor>>
* <<blockManagerInfo, blockManagerInfo>>

In the end, register requests the <<listenerBus, LiveListenerBus>> to xref:scheduler:LiveListenerBus.adoc#post[post] a xref:ROOT:SparkListener.adoc#SparkListenerBlockManagerAdded[SparkListenerBlockManagerAdded] message.

register is used when BlockManagerMasterEndpoint is requested to handle <<RegisterBlockManager, RegisterBlockManager>> message.

== [[removeExecutor]] removeExecutor Internal Method

[source, scala]
----
removeExecutor(
  execId: String): Unit
----

removeExecutor prints the following INFO message to the logs:

[source,plaintext]
----
Trying to remove executor [execId] from BlockManagerMaster.
----

If the `execId` executor is registered (in the internal <<blockManagerIdByExecutor, blockManagerIdByExecutor>> internal registry), removeExecutor <<removeBlockManager, removes the corresponding `BlockManager`>>.

removeExecutor is used when BlockManagerMasterEndpoint is requested to handle <<RemoveExecutor, RemoveExecutor>> or <<RegisterBlockManager, RegisterBlockManager>> messages.

== [[removeBlockManager]] removeBlockManager Internal Method

[source, scala]
----
removeBlockManager(
  blockManagerId: BlockManagerId): Unit
----

removeBlockManager looks up `blockManagerId` and removes the executor it was working on from the internal registries:

* <<blockManagerIdByExecutor, blockManagerIdByExecutor>>
* <<blockManagerInfo, blockManagerInfo>>

It then goes over all the blocks for the `BlockManager`, and removes the executor for each block from `blockLocations` registry.

xref:ROOT:SparkListener.adoc#SparkListenerBlockManagerRemoved[SparkListenerBlockManagerRemoved(System.currentTimeMillis(), blockManagerId)] is posted to xref:ROOT:SparkContext.adoc#listenerBus[listenerBus].

You should then see the following INFO message in the logs:

[source,plaintext]
----
Removing block manager [blockManagerId]
----

removeBlockManager is used when BlockManagerMasterEndpoint is requested to <<removeExecutor, removeExecutor>> (to handle <<RemoveExecutor, RemoveExecutor>> or <<RegisterBlockManager, RegisterBlockManager>> messages).

== [[getLocations]] getLocations Internal Method

[source, scala]
----
getLocations(
  blockId: BlockId): Seq[BlockManagerId]
----

getLocations looks up the given xref:storage:BlockId.adoc[] in the `blockLocations` internal registry and returns the locations (as a collection of `BlockManagerId`) or an empty collection.

getLocations is used when BlockManagerMasterEndpoint is requested to handle <<GetLocations, GetLocations>> and <<GetLocationsMultipleBlockIds, GetLocationsMultipleBlockIds>> messages.

== [[logging]] Logging

Enable `ALL` logging level for `org.apache.spark.storage.BlockManagerMasterEndpoint` logger to see what happens inside.

Add the following line to `conf/log4j.properties`:

[source]
----
log4j.logger.org.apache.spark.storage.BlockManagerMasterEndpoint=ALL
----

Refer to xref:ROOT:spark-logging.adoc[Logging].

== [[internal-properties]] Internal Properties

=== [[blockManagerIdByExecutor]] blockManagerIdByExecutor Lookup Table

[source,scala]
----
blockManagerIdByExecutor: Map[String, BlockManagerId]
----

Lookup table of xref:storage:BlockManagerId.adoc[]s by executor ID

A new executor is added when BlockManagerMasterEndpoint is requested to handle a <<RegisterBlockManager, RegisterBlockManager>> message (and <<register, registers a new BlockManager>>).

An executor is removed when BlockManagerMasterEndpoint is requested to handle a <<RemoveExecutor, RemoveExecutor>> and a <<RegisterBlockManager, RegisterBlockManager>> messages (via <<removeBlockManager, removeBlockManager>>)

Used when BlockManagerMasterEndpoint is requested to handle <<HasCachedBlocks, HasCachedBlocks>> message, <<removeExecutor, removeExecutor>>, <<register, register>> and <<getExecutorEndpointRef, getExecutorEndpointRef>>.

=== [[blockManagerInfo]] blockManagerInfo Lookup Table

[source,scala]
----
blockManagerIdByExecutor: Map[String, BlockManagerId]
----

Lookup table of xref:storage:BlockManagerInfo.adoc[] by xref:storage:BlockManagerId.adoc[]

A new BlockManagerInfo is added when BlockManagerMasterEndpoint is requested to handle a <<RegisterBlockManager, RegisterBlockManager>> message (and <<register, registers a new BlockManager>>).

A BlockManagerInfo is removed when BlockManagerMasterEndpoint is requested to <<removeBlockManager, remove a BlockManager>> (to handle <<RemoveExecutor, RemoveExecutor>> and <<RegisterBlockManager, RegisterBlockManager>> messages).

=== [[blockLocations]] blockLocations

[source,scala]
----
blockLocations: Map[BlockId, Set[BlockManagerId]]
----

Collection of xref:storage:BlockId.adoc[] and their locations (as `BlockManagerId`).

Used in `removeRdd` to remove blocks for a RDD, removeBlockManager to remove blocks after a BlockManager gets removed, `removeBlockFromWorkers`, `updateBlockInfo`, and <<getLocations, getLocations>>.
