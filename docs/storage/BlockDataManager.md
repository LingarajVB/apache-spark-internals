= BlockDataManager

*BlockDataManager* is an <<contract, abstraction>> of <<implementations, block data managers>> that manage storage for blocks of data (aka _block storage management API_).

BlockDataManager uses xref:storage:BlockId.adoc[] to uniquely identify blocks of data and xref:network:ManagedBuffer.adoc[] to represent them.

BlockDataManager is used to initialize a xref:storage:BlockTransferService.adoc#init[].

BlockDataManager is used to create a xref:storage:NettyBlockRpcServer.adoc[].

== [[contract]] Contract

=== [[getBlockData]] getBlockData

[source,scala]
----
getBlockData(
  blockId: BlockId): ManagedBuffer
----

Fetches a block data (as a xref:network:ManagedBuffer.adoc[]) for the given xref:storage:BlockId.adoc[]

Used when:

* NettyBlockRpcServer is requested to xref:storage:NettyBlockRpcServer.adoc#OpenBlocks[handle a OpenBlocks message]

* ShuffleBlockFetcherIterator is requested to xref:storage:ShuffleBlockFetcherIterator.adoc#fetchLocalBlocks[fetchLocalBlocks]

=== [[putBlockData]] putBlockData

[source, scala]
----
putBlockData(
  blockId: BlockId,
  data: ManagedBuffer,
  level: StorageLevel,
  classTag: ClassTag[_]): Boolean
----

Stores (_puts_) a block data (as a xref:network:ManagedBuffer.adoc[]) for the given xref:storage:BlockId.adoc[]. Returns `true` when completed successfully or `false` when failed.

Used when NettyBlockRpcServer is requested to xref:storage:NettyBlockRpcServer.adoc#UploadBlock[handle an UploadBlock message]

=== [[putBlockDataAsStream]] putBlockDataAsStream

[source, scala]
----
putBlockDataAsStream(
  blockId: BlockId,
  level: StorageLevel,
  classTag: ClassTag[_]): StreamCallbackWithID
----

Stores a block data that will be received as a stream

Used when NettyBlockRpcServer is requested to xref:storage:NettyBlockRpcServer.adoc#receiveStream[receiveStream]

=== [[releaseLock]] releaseLock

[source, scala]
----
releaseLock(
  blockId: BlockId,
  taskAttemptId: Option[Long]): Unit
----

Releases a lock

Used when:

* TorrentBroadcast is requested to xref:core:TorrentBroadcast.adoc#releaseLock[releaseLock]

* BlockManager is requested to xref:storage:BlockManager.adoc#handleLocalReadFailure[handleLocalReadFailure], xref:storage:BlockManager.adoc#getLocalValues[getLocalValues], xref:storage:BlockManager.adoc#getOrElseUpdate[getOrElseUpdate], xref:storage:BlockManager.adoc#doPut[doPut], and xref:storage:BlockManager.adoc#releaseLockAndDispose[releaseLockAndDispose]

== [[implementations]] Available BlockDataManagers

xref:storage:BlockManager.adoc[] is the default and only known BlockDataManager in Apache Spark.
