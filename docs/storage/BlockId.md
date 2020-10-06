= BlockId

*BlockId* is an <<contract, abstraction>> of <<implementations, data block identifiers>> based on an unique <<name, name>>.

BlockId is a Scala sealed abstract class and so all the possible <<implementations, implementations>> are in the single Scala file alongside BlockId.

== [[contract]] Contract

=== [[name]][[toString]] Unique Name

[source, scala]
----
name: String
----

Used when:

* NettyBlockTransferService is requested to xref:storage:NettyBlockTransferService.adoc#uploadBlock[upload a block]

* AppStatusListener is requested to xref:core:AppStatusListener.adoc#updateRDDBlock[updateRDDBlock], xref:core:AppStatusListener.adoc#updateStreamBlock[updateStreamBlock]

* BlockManager is requested to xref:storage:BlockManager.adoc#putBlockDataAsStream[putBlockDataAsStream]

* UpdateBlockInfo is requested to xref:storage:BlockManagerMasterEndpoint.adoc#UpdateBlockInfo[writeExternal]

* DiskBlockManager is requested to xref:storage:DiskBlockManager.adoc#getFile[getFile] and xref:storage:DiskBlockManager.adoc#containsBlock[containsBlock]

* DiskStore is requested to xref:storage:DiskStore.adoc#getBytes[getBytes]

== [[implementations]] Available BlockIds

=== [[BroadcastBlockId]] BroadcastBlockId

BlockId for xref:ROOT:Broadcast.adoc[]s with `broadcastId` identifier and optional `field` name (default: `empty`)

Uses `broadcast_` prefix for the <<name, name>>

Used when:

* TorrentBroadcast is xref:core:TorrentBroadcast.adoc#broadcastId[created], requested to xref:core:TorrentBroadcast.adoc#writeBlocks[store a broadcast and the blocks in a local BlockManager], and <<readBlocks, read blocks>>

* BlockManager is requested to xref:storage:BlockManager.adoc#removeBroadcast[remove all the blocks of a broadcast variable]

* AppStatusListener is requested to xref:core:AppStatusListener.adoc#updateBroadcastBlock[updateBroadcastBlock] (when xref:core:AppStatusListener.adoc#onBlockUpdated[onBlockUpdated] for a `BroadcastBlockId`)

xref:serializer:SerializerManager.adoc#shouldCompress[Compressed] when xref:core:BroadcastManager.adoc#spark.broadcast.compress[spark.broadcast.compress] configuration property is enabled

=== [[RDDBlockId]] RDDBlockId

BlockId for RDD partitions with `rddId` and `splitIndex` identifiers

Uses `rdd_` prefix for the <<name, name>>

Used when:

* `StorageStatus` is requested to <<spark-blockmanager-StorageStatus.adoc#addBlock, register the status of a data block>>, <<spark-blockmanager-StorageStatus.adoc#getBlock, get the status of a data block>>, <<spark-blockmanager-StorageStatus.adoc#updateStorageInfo, updateStorageInfo>>

* `LocalCheckpointRDD` is requested to `compute` a partition

* LocalRDDCheckpointData is requested to xref:rdd:LocalRDDCheckpointData.adoc#doCheckpoint[doCheckpoint]

* `RDD` is requested to xref:rdd:RDD.adoc#getOrCompute[getOrCompute]

* `DAGScheduler` is requested for the xref:scheduler:DAGScheduler.adoc#getCacheLocs[BlockManagers (executors) for cached RDD partitions]

* `AppStatusListener` is requested to xref:core:AppStatusListener.adoc#updateRDDBlock[updateRDDBlock] (when xref:core:AppStatusListener.adoc#onBlockUpdated[onBlockUpdated] for a `RDDBlockId`)

xref:serializer:SerializerManager.adoc#shouldCompress[Compressed] when xref:ROOT:configuration-properties.adoc#spark.rdd.compress[spark.rdd.compress] configuration property is enabled (default: `false`)

=== [[ShuffleBlockId]] ShuffleBlockId

BlockId for _FIXME_ with `shuffleId`, `mapId`, and `reduceId` identifiers

Uses `shuffle_` prefix for the <<name, name>>

Used when:

* `ShuffleBlockFetcherIterator` is requested to xref:storage:ShuffleBlockFetcherIterator.adoc#throwFetchFailedException[throwFetchFailedException]

* `MapOutputTracker` object is requested to xref:scheduler:MapOutputTracker.adoc#convertMapStatuses[convertMapStatuses]

* `SortShuffleWriter` is requested to xref:shuffle:SortShuffleWriter.adoc#write[write partition records]

* `ShuffleBlockResolver` is requested for a xref:shuffle:ShuffleBlockResolver.adoc#getBlockData[ManagedBuffer to read shuffle block data file]

xref:serializer:SerializerManager.adoc#shouldCompress[Compressed] when xref:ROOT:configuration-properties.adoc#spark.shuffle.compress[spark.shuffle.compress] configuration property is enabled (default: `true`)

=== [[ShuffleDataBlockId]] ShuffleDataBlockId

=== [[ShuffleIndexBlockId]] ShuffleIndexBlockId

=== [[StreamBlockId]] StreamBlockId

=== [[TaskResultBlockId]] TaskResultBlockId

=== [[TempLocalBlockId]] TempLocalBlockId

=== [[TempShuffleBlockId]] TempShuffleBlockId

== [[apply]] apply Factory Method

[source, scala]
----
apply(
  name: String): BlockId
----

apply creates one of the available <<implementations, BlockIds>> by the given name (that uses a prefix to differentiate between different BlockIds).

apply is used when:

* NettyBlockRpcServer is requested to xref:storage:NettyBlockRpcServer.adoc#receive[handle an RPC message] and xref:storage:NettyBlockRpcServer.adoc#receiveStream[receiveStream]

* UpdateBlockInfo is requested to deserialize (readExternal)

* DiskBlockManager is requested for xref:storage:DiskBlockManager.adoc#getAllBlocks[all the blocks (from files stored on disk)]

* ShuffleBlockFetcherIterator is requested to xref:storage:ShuffleBlockFetcherIterator.adoc#sendRequest[sendRequest]

* JsonProtocol utility is used to xref:spark-history-server:JsonProtocol.adoc#accumValueFromJson[accumValueFromJson], xref:spark-history-server:JsonProtocol.adoc#taskMetricsFromJson[taskMetricsFromJson] and xref:spark-history-server:JsonProtocol.adoc#blockUpdatedInfoFromJson[blockUpdatedInfoFromJson]
