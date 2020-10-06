= [[DiskBlockObjectWriter]] DiskBlockObjectWriter

*DiskBlockObjectWriter* is a custom {java-javadoc-url}/java/io/OutputStream.html[java.io.OutputStream] that xref:storage:BlockManager.adoc#getDiskWriter[BlockManager] offers for <<write, writing data blocks to disk>>.

DiskBlockObjectWriter is used when:

* BypassMergeSortShuffleWriter is requested for xref:shuffle:BypassMergeSortShuffleWriter.adoc#partitionWriters[partition writers]

* UnsafeSorterSpillWriter is requested for a xref:memory:UnsafeSorterSpillWriter.adoc#writer[partition writer]

* ShuffleExternalSorter is requested to xref:shuffle:ShuffleExternalSorter.adoc#writeSortedFile[writeSortedFile]

* ExternalSorter is requested to xref:shuffle:ExternalSorter.adoc#spillMemoryIteratorToDisk[spillMemoryIteratorToDisk]

== [[creating-instance]] Creating Instance

DiskBlockObjectWriter takes the following to be created:

* [[file]] Java {java-javadoc-url}/java/io/File.html[File]
* [[serializerManager]] xref:serializer:SerializerManager.adoc[]
* [[serializerInstance]] xref:serializer:SerializerInstance.adoc[]
* [[bufferSize]] Buffer size
* [[syncWrites]] syncWrites flag
* [[writeMetrics]] xref:executor:ShuffleWriteMetrics.adoc[]
* [[blockId]] xref:storage:BlockId.adoc[] (default: `null`)

DiskBlockObjectWriter is created when:

* BlockManager is requested for xref:storage:BlockManager.adoc#getDiskWriter[one]

* BypassMergeSortShuffleWriter is requested to xref:shuffle:BypassMergeSortShuffleWriter.adoc#write[write records] (as xref:shuffle:BypassMergeSortShuffleWriter.adoc#partitionWriters[partition writers])

== [[objOut]] SerializationStream

DiskBlockObjectWriter manages a xref:serializer:SerializationStream.adoc[SerializationStream] for <<write, writing a key-value record>>:

* Opens it when requested to <<open, open>>

* Closes it when requested to <<commitAndGet, commitAndGet>>

* Dereferences it (``null``s it) when <<closeResources, closeResources>>

== [[states]][[streamOpen]] States

DiskBlockObjectWriter can be in the following states (that match the state of the underlying output streams):

. Initialized
. Open
. Closed

== [[write]] Writing Key and Value (of Record)

[source, scala]
----
write(
  key: Any,
  value: Any): Unit
----

write <<open, opens the underlying stream>> unless <<streamOpen, open>> already.

write requests the <<objOut, SerializationStream>> to xref:serializer:SerializationStream.adoc#writeKey[write the key] and then the xref:serializer:SerializationStream.adoc#writeValue[value].

In the end, write <<recordWritten, updates the write metrics>>.

write is used when:

* BypassMergeSortShuffleWriter is requested to xref:shuffle:BypassMergeSortShuffleWriter.adoc#write[write records of a partition]

* ExternalAppendOnlyMap is requested to xref:shuffle:ExternalAppendOnlyMap.adoc#spillMemoryIteratorToDisk[spillMemoryIteratorToDisk]

* ExternalSorter is requested to xref:shuffle:ExternalSorter.adoc#writePartitionedFile[write all records into a partitioned file]
** SpillableIterator is requested to spill

* WritablePartitionedPairCollection is requested for a destructiveSortedWritablePartitionedIterator

== [[commitAndGet]] commitAndGet Method

[source, scala]
----
commitAndGet(): FileSegment
----

commitAndGet...FIXME

commitAndGet is used when...FIXME

== [[close]] Committing Writes and Closing Resources

[source, scala]
----
close(): Unit
----

close...FIXME

close is used when...FIXME

== [[revertPartialWritesAndClose]] revertPartialWritesAndClose Method

[source, scala]
----
revertPartialWritesAndClose(): File
----

revertPartialWritesAndClose...FIXME

revertPartialWritesAndClose is used when...FIXME

== [[updateBytesWritten]] updateBytesWritten Method

CAUTION: FIXME

== [[initialize]] initialize Method

CAUTION: FIXME

== [[write-bytes]] Writing Bytes (From Byte Array Starting From Offset)

[source, scala]
----
write(kvBytes: Array[Byte], offs: Int, len: Int): Unit
----

write...FIXME

CAUTION: FIXME

== [[recordWritten]] recordWritten Method

CAUTION: FIXME

== [[open]] Opening DiskBlockObjectWriter

[source, scala]
----
open(): DiskBlockObjectWriter
----

`open` opens DiskBlockObjectWriter, i.e. <<initialize, initializes>> and re-sets <<bs, bs>> and <<objOut, objOut>> internal output streams.

Internally, `open` makes sure that DiskBlockObjectWriter is not closed (i.e. <<hasBeenClosed, hasBeenClosed>> flag is disabled). If it was, `open` throws a `IllegalStateException`:

```
Writer already closed. Cannot be reopened.
```

Unless DiskBlockObjectWriter has already been initialized (i.e. <<initialized, initialized>> flag is enabled), `open` <<initialize, initializes>> it (and turns <<initialized, initialized>> flag on).

Regardless of whether DiskBlockObjectWriter was already initialized or not, `open` xref:serializer:SerializerManager.adoc#wrapStream[requests `SerializerManager` to wrap `mcs` output stream for encryption and compression] (for <<blockId, blockId>>) and sets it as <<bs, bs>>.

`open` requests the <<serializerInstance, SerializerInstance>> to xref:serializer:SerializerInstance.adoc#serializeStream[serialize `bs` output stream] and sets it as <<objOut, objOut>>.

NOTE: `open` uses `SerializerInstance` that was specified when <<creating-instance, DiskBlockObjectWriter was created>>

In the end, `open` turns <<streamOpen, streamOpen>> flag on.

NOTE: `open` is used exclusively when DiskBlockObjectWriter <<write, writes a key-value pair>> or <<write-bytes, bytes from a specified byte array>> but the <<streamOpen, stream is not open yet>>.

== [[internal-properties]] Internal Properties

[cols="30m,70",options="header",width="100%"]
|===
| Name
| Description

| initialized
| [[initialized]] Internal flag...FIXME

Used when...FIXME

| hasBeenClosed
| [[hasBeenClosed]] Internal flag...FIXME

Used when...FIXME

| mcs
| [[mcs]] FIXME

Used when...FIXME

| bs
| [[bs]] FIXME

Used when...FIXME

|===
