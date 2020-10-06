= BlockFetchStarter

*BlockFetchStarter* is the <<contract, contract>> of...FIXME...to <<createAndStart, createAndStart>>.

[[contract]]
[[createAndStart]]
[source, java]
----
void createAndStart(String[] blockIds, BlockFetchingListener listener)
   throws IOException, InterruptedException;
----

`createAndStart` is used when:

* `ExternalShuffleClient` is requested to xref:storage:ExternalShuffleClient.adoc#fetchBlocks[fetchBlocks] (when xref:network:TransportConf.adoc#io.maxRetries[maxIORetries] is `0`)

* `NettyBlockTransferService` is requested to xref:storage:NettyBlockTransferService.adoc#fetchBlocks[fetchBlocks] (when xref:network:TransportConf.adoc#io.maxRetries[maxIORetries] is `0`)

* `RetryingBlockFetcher` is requested to xref:core:RetryingBlockFetcher.adoc#fetchAllOutstanding[fetchAllOutstanding]
