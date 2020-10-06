= BlockFetchingListener

*BlockFetchingListener* is the <<contract, contract>> of <<implementations, EventListeners>> that want to be notified about <<onBlockFetchSuccess, onBlockFetchSuccess>> and <<onBlockFetchFailure, onBlockFetchFailure>>.

BlockFetchingListener is used when:

* xref:storage:ShuffleClient.adoc#fetchBlocks[ShuffleClient], xref:storage:BlockTransferService.adoc#fetchBlocks[BlockTransferService], xref:storage:NettyBlockTransferService.adoc#fetchBlocks[NettyBlockTransferService], and xref:storage:ExternalShuffleClient.adoc#fetchBlocks[ExternalShuffleClient] are requested to fetch a sequence of blocks

* `BlockFetchStarter` is requested to xref:core:BlockFetchStarter.adoc#createAndStart[createAndStart]

* xref:core:RetryingBlockFetcher.adoc[] and xref:storage:OneForOneBlockFetcher.adoc[] are created

[[contract]]
[source, java]
----
package org.apache.spark.network.shuffle;

interface BlockFetchingListener extends EventListener {
  void onBlockFetchSuccess(String blockId, ManagedBuffer data);
  void onBlockFetchFailure(String blockId, Throwable exception);
}
----

.BlockFetchingListener Contract
[cols="1,2",options="header",width="100%"]
|===
| Method
| Description

| `onBlockFetchSuccess`
| [[onBlockFetchSuccess]] Used when...FIXME

| `onBlockFetchFailure`
| [[onBlockFetchFailure]] Used when...FIXME
|===

[[implementations]]
.BlockFetchingListeners
[cols="1,2",options="header",width="100%"]
|===
| BlockFetchingListener
| Description

| xref:core:RetryingBlockFetcher.adoc#RetryingBlockFetchListener[RetryingBlockFetchListener]
| [[RetryingBlockFetchListener]]

| "Unnamed" in xref:storage:ShuffleBlockFetcherIterator.adoc#sendRequest[ShuffleBlockFetcherIterator]
| [[ShuffleBlockFetcherIterator]]

| "Unnamed" in xref:storage:BlockTransferService.adoc#fetchBlockSync[BlockTransferService]
| [[BlockTransferService]]
|===
