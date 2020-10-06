= UnsafeShuffleWriter

*UnsafeShuffleWriter* is a xref:shuffle:ShuffleWriter.adoc[ShuffleWriter] for xref:shuffle:SerializedShuffleHandle.adoc[SerializedShuffleHandles].

UnsafeShuffleWriter is <<creating-instance, created>> when SortShuffleManager is requested for a xref:shuffle:SortShuffleManager.adoc#getWriter[ShuffleWriter] for a <<handle, SerializedShuffleHandle>>.

.UnsafeShuffleWriter, SortShuffleManager and SerializedShuffleHandle
image::UnsafeShuffleWriter.png[align="center"]

UnsafeShuffleWriter <<open, opens resources>> (a <<sorter, ShuffleExternalSorter>> and the buffers) immediately while being <<creating-instance, created>>.

.UnsafeShuffleWriter and ShuffleExternalSorter
image::UnsafeShuffleWriter-ShuffleExternalSorter.png[align="center"]

When requested to <<write, write key-value records of a partition>>, UnsafeShuffleWriter simply <<insertRecordIntoSorter, inserts every record into ShuffleExternalSorter>> followed by <<closeAndWriteOutput, close internal resources and merge spill files>> (that, among other things, creates the <<mapStatus, MapStatus>>).

When requested to <<stop, stop>>, UnsafeShuffleWriter records the peak execution memory metric and returns the <<mapStatus, mapStatus>> (that was created when requested to <<write, write>>).

== [[creating-instance]] Creating Instance

UnsafeShuffleWriter takes the following to be created:

* [[blockManager]] xref:storage:BlockManager.adoc[BlockManager]
* <<shuffleBlockResolver, IndexShuffleBlockResolver>>
* [[memoryManager]] xref:memory:TaskMemoryManager.adoc[TaskMemoryManager]
* [[handle]] xref:shuffle:SerializedShuffleHandle.adoc[SerializedShuffleHandle]
* [[mapId]] Map ID
* [[taskContext]] xref:scheduler:spark-TaskContext.adoc[TaskContext]
* [[sparkConf]] xref:ROOT:SparkConf.adoc[SparkConf]

UnsafeShuffleWriter requests the <<handle, SerializedShuffleHandle>> for the <<spark-shuffle-BaseShuffleHandle.adoc#dependency, ShuffleDependency>> that is then requested for the xref:rdd:ShuffleDependency.adoc#partitioner[Partitioner] and, in the end, for the xref:rdd:Partitioner.adoc#numPartitions[number of partitions]. UnsafeShuffleWriter makes sure that the number of shuffle output partitions is below xref:SortShuffleManager.adoc#MAX_SHUFFLE_OUTPUT_PARTITIONS_FOR_SERIALIZED_MODE[`(1 << 24)` partition identifiers that can be encoded] and throws an `IllegalArgumentException` if not met:

[source,plaintext]
----
UnsafeShuffleWriter can only be used for shuffles with at most 16777215 reduce partitions
----

NOTE: The number of shuffle output partitions is first enforced when xref:SortShuffleManager.adoc#canUseSerializedShuffle[`SortShuffleManager` checks if `SerializedShuffleHandle` can be used for `ShuffleHandle`] (that eventually leads to UnsafeShuffleWriter).

In the end, UnsafeShuffleWriter <<open, creates a ShuffleExternalSorter and a SerializationStream>>.

== [[shuffleBlockResolver]] IndexShuffleBlockResolver

UnsafeShuffleWriter is given a xref:shuffle:IndexShuffleBlockResolver.adoc[IndexShuffleBlockResolver] to be created.

UnsafeShuffleWriter uses the IndexShuffleBlockResolver for...FIXME

== [[DEFAULT_INITIAL_SER_BUFFER_SIZE]] DEFAULT_INITIAL_SER_BUFFER_SIZE

UnsafeShuffleWriter uses a fixed buffer size for the <<serBuffer, output stream of serialized data written into a byte array>> (default: `1024 * 1024`).

== [[inputBufferSizeInBytes]] inputBufferSizeInBytes

UnsafeShuffleWriter uses the xref:ROOT:configuration-properties.adoc#spark.shuffle.file.buffer[spark.shuffle.file.buffer] configuration property for...FIXME

== [[outputBufferSizeInBytes]] outputBufferSizeInBytes

UnsafeShuffleWriter uses the xref:ROOT:configuration-properties.adoc#spark.shuffle.unsafe.file.output.buffer[spark.shuffle.unsafe.file.output.buffer] configuration property (default: `32k`) for...FIXME

== [[transferToEnabled]] transferToEnabled

UnsafeShuffleWriter can use a <<mergeSpillsWithTransferTo, specialized NIO-based fast merge procedure>> that avoids extra serialization/deserialization when xref:ROOT:configuration-properties.adoc#spark.file.transferTo[spark.file.transferTo] configuration property is enabled.

== [[initialSortBufferSize]][[DEFAULT_INITIAL_SORT_BUFFER_SIZE]] initialSortBufferSize

UnsafeShuffleWriter uses the <<initialSortBufferSize, initial buffer size for sorting>> (default: `4096`) when creating a <<sorter, ShuffleExternalSorter>> (when requested to <<open, open>>).

TIP: Use xref:ROOT:configuration-properties.adoc#spark.shuffle.sort.initialBufferSize[spark.shuffle.sort.initialBufferSize] configuration property to change the default buffer size.

== [[mergeSpillsWithTransferTo]] mergeSpillsWithTransferTo Internal Method

[source, java]
----
long[] mergeSpillsWithTransferTo(
  SpillInfo[] spills,
  File outputFile)
----

mergeSpillsWithTransferTo...FIXME

mergeSpillsWithTransferTo is used when UnsafeShuffleWriter is requested to <<mergeSpills, mergeSpills>> (with the <<transferToEnabled, transferToEnabled>> flag enabled and no encryption).

== [[mergeSpills]] Merging Spills

[source, java]
----
long[] mergeSpills(
  SpillInfo[] spills,
  File outputFile)
----

=== [[mergeSpills-many-spills]] Many Spills

With multiple SpillInfos to merge, mergeSpills selects between fast and <<mergeSpillsWithFileStream, slow merge strategies>>. The fast merge strategy can be <<mergeSpillsWithTransferTo, transferTo>>- or <<mergeSpillsWithFileStream, fileStream>>-based.

mergeSpills uses the xref:ROOT:configuration-properties.adoc#spark.shuffle.unsafe.fastMergeEnabled[spark.shuffle.unsafe.fastMergeEnabled] configuration property to consider one of the fast merge strategies.

A fast merge strategy is supported when xref:ROOT:configuration-properties.adoc#spark.shuffle.compress[spark.shuffle.compress] configuration property is disabled or the IO compression codec xref:io:CompressionCodec.adoc#supportsConcatenationOfSerializedStreams[supports decompression of concatenated compressed streams].

With xref:ROOT:configuration-properties.adoc#spark.shuffle.compress[spark.shuffle.compress] configuration property enabled, mergeSpills will always use the slow merge strategy.

With fast merge strategy enabled and supported, <<transferToEnabled, transferToEnabled>> enabled and encryption disabled, mergeSpills prints out the following DEBUG message to the logs and <<mergeSpillsWithTransferTo, mergeSpillsWithTransferTo>>.

[source,plaintext]
----
Using transferTo-based fast merge
----

With fast merge strategy enabled and supported, no <<transferToEnabled, transferToEnabled>> or encryption enabled, mergeSpills prints out the following DEBUG message to the logs and <<mergeSpillsWithFileStream, mergeSpillsWithFileStream>> (with no compression codec).

[source,plaintext]
----
Using fileStream-based fast merge
----

For slow merge, mergeSpills prints out the following DEBUG message to the logs and <<mergeSpillsWithFileStream, mergeSpillsWithFileStream>> (with the compression codec).

[source,plaintext]
----
Using slow merge
----

In the end, mergeSpills requests the <<writeMetrics, ShuffleWriteMetrics>> to xref:executor:ShuffleWriteMetrics.adoc#decBytesWritten[decBytesWritten] and xref:executor:ShuffleWriteMetrics.adoc#incBytesWritten[incBytesWritten], and returns the partition length array.

=== [[mergeSpills-one-spill]] One Spill

With one SpillInfo to merge, mergeSpills simply renames the spill file to be the output file and returns the partition length array of the one spill.

=== [[mergeSpills-no-spills]] No Spills

With no SpillInfos to merge, mergeSpills creates an empty output file and returns an array of ``0``s of size of the xref:rdd:Partitioner.adoc#numPartitions[numPartitions] of the <<partitioner, Partitioner>>.

=== [[mergeSpills-usage]] Usage

mergeSpills is used when UnsafeShuffleWriter is requested to <<closeAndWriteOutput, close internal resources and merge spill files>>.

== [[updatePeakMemoryUsed]] updatePeakMemoryUsed Internal Method

[source, java]
----
void updatePeakMemoryUsed()
----

updatePeakMemoryUsed...FIXME

updatePeakMemoryUsed is used when UnsafeShuffleWriter is requested for the <<getPeakMemoryUsedBytes, peak memory used>> and to <<closeAndWriteOutput, close internal resources and merge spill files>>.

== [[write]] Writing Key-Value Records of Partition

[source, java]
----
void write(
  Iterator<Product2<K, V>> records)
----

write traverses the input sequence of records (for a RDD partition) and <<insertRecordIntoSorter, insertRecordIntoSorter>> one by one. When all the records have been processed, write <<closeAndWriteOutput, closes internal resources and merges spill files>>.

In the end, write xref:shuffle:ShuffleExternalSorter.adoc#cleanupResources[requests `ShuffleExternalSorter` to clean after itself].

CAUTION: FIXME

write is part of the xref:shuffle:ShuffleWriter.adoc#write[ShuffleWriter] abstraction.

== [[stop]] Stopping ShuffleWriter

[source, java]
----
Option<MapStatus> stop(
  boolean success)
----

stop...FIXME

stop is part of the xref:shuffle:ShuffleWriter.adoc#stop[ShuffleWriter] abstraction.

== [[insertRecordIntoSorter]] Inserting Record Into ShuffleExternalSorter

[source, java]
----
void insertRecordIntoSorter(
  Product2<K, V> record)
----

insertRecordIntoSorter requires that the <<sorter, ShuffleExternalSorter>> is available.

insertRecordIntoSorter requests the <<serBuffer, MyByteArrayOutputStream>> to reset (so that all currently accumulated output in the output stream is discarded and reusing the already allocated buffer space).

insertRecordIntoSorter requests the <<serOutputStream, SerializationStream>> to write out the record (write the xref:serializer:SerializationStream.adoc#writeKey[key] and the xref:serializer:SerializationStream.adoc#writeValue[value]) and to xref:serializer:SerializationStream.adoc#flush[flush].

[[insertRecordIntoSorter-serializedRecordSize]]
insertRecordIntoSorter requests the <<serBuffer, MyByteArrayOutputStream>> for the length of the buffer.

[[insertRecordIntoSorter-partitionId]]
insertRecordIntoSorter requests the <<partitioner, Partitioner>> for the xref:rdd:Partitioner.adoc#getPartition[partition] for the given record (by the key).

In the end, insertRecordIntoSorter requests the <<sorter, ShuffleExternalSorter>> to xref:shuffle:ShuffleExternalSorter.adoc#insertRecord[insert] the <<serBuffer, MyByteArrayOutputStream>> as a byte array (with the <<insertRecordIntoSorter-serializedRecordSize, length>> and the <<insertRecordIntoSorter-partitionId, partition>>).

insertRecordIntoSorter is used when UnsafeShuffleWriter is requested to <<write, write records>>.

== [[closeAndWriteOutput]] Closing and Writing Output (Merging Spill Files)

[source, java]
----
void closeAndWriteOutput()
----

closeAndWriteOutput asserts that the <<sorter, ShuffleExternalSorter>> is available (non-``null``).

closeAndWriteOutput <<updatePeakMemoryUsed, updates peak memory used>>.

closeAndWriteOutput removes the references to the <<serBuffer, ByteArrayOutputStream>> and <<serOutputStream, SerializationStream>> output streams (``null``s them).

closeAndWriteOutput requests the <<sorter, ShuffleExternalSorter>> to xref:shuffle:ShuffleExternalSorter.adoc#closeAndGetSpills[close and return spill metadata].

closeAndWriteOutput removes the reference to the <<sorter, ShuffleExternalSorter>> (``null``s it).

closeAndWriteOutput requests the <<shuffleBlockResolver, IndexShuffleBlockResolver>> for the xref:shuffle:IndexShuffleBlockResolver.adoc#getDataFile[output data file] for the <<shuffleId, shuffle>> and <<mapId, map>> IDs.

[[closeAndWriteOutput-partitionLengths]][[closeAndWriteOutput-tmp]]
closeAndWriteOutput creates a temporary file (along the data output file) and uses it to <<mergeSpills, merge spill files>> (that gives a partition length array). All spill files are then deleted.

closeAndWriteOutput requests the <<shuffleBlockResolver, IndexShuffleBlockResolver>> to xref:shuffle:IndexShuffleBlockResolver.adoc#writeIndexFileAndCommit[write shuffle index and data files] (for the <<shuffleId, shuffle>> and <<mapId, map>> IDs, the <<closeAndWriteOutput-partitionLengths, partition length array>> and the <<closeAndWriteOutput-tmp, temporary output data file>>).

In the end, closeAndWriteOutput creates a xref:scheduler:MapStatus.adoc[MapStatus] with the xref:storage:BlockManager.adoc#shuffleServerId[location of the local BlockManager] and the <<closeAndWriteOutput-partitionLengths, partition length array>>.

closeAndWriteOutput prints out the following ERROR message to the logs if there is an issue with deleting spill files:

[source,plaintext]
----
Error while deleting spill file [path]
----

closeAndWriteOutput prints out the following ERROR message to the logs if there is an issue with deleting the <<closeAndWriteOutput-tmp, temporary output data file>>:

[source,plaintext]
----
Error while deleting temp file [path]
----

closeAndWriteOutput is used when UnsafeShuffleWriter is requested to <<write, write records>>.

== [[mergeSpillsWithFileStream]] mergeSpillsWithFileStream Method

[source, java]
----
long[] mergeSpillsWithFileStream(
  SpillInfo[] spills,
  File outputFile,
  CompressionCodec compressionCodec)
----

mergeSpillsWithFileStream will be given an xref:io:CompressionCodec.adoc[IO compression codec] when shuffle compression is enabled.

mergeSpillsWithFileStream...FIXME

mergeSpillsWithFileStream requires that there are at least two spills to merge.

mergeSpillsWithFileStream is used when UnsafeShuffleWriter is requested to <<mergeSpills, merge spills>>.

== [[getPeakMemoryUsedBytes]] Getting Peak Memory Used

[source, java]
----
long getPeakMemoryUsedBytes()
----

getPeakMemoryUsedBytes simply <<updatePeakMemoryUsed, updatePeakMemoryUsed>> and returns the internal <<peakMemoryUsedBytes, peakMemoryUsedBytes>> registry.

getPeakMemoryUsedBytes is used when UnsafeShuffleWriter is requested to <<stop, stop>>.

== [[open]] Opening UnsafeShuffleWriter and Buffers

[source, java]
----
void open()
----

open requires that there is no <<sorter, ShuffleExternalSorter>> available.

open creates a xref:shuffle:ShuffleExternalSorter.adoc[ShuffleExternalSorter].

open creates a <<serBuffer, serialized buffer>> with the capacity of <<DEFAULT_INITIAL_SER_BUFFER_SIZE, 1M>>.

open requests the <<serializer, SerializerInstance>> for a xref:serializer:SerializerInstance.adoc#serializeStream[SerializationStream] to the <<serBuffer, serBuffer>> (available internally as the <<serOutputStream, serOutputStream>> reference).

open is used when UnsafeShuffleWriter is <<creating-instance, created>>.

== [[logging]] Logging

Enable `ALL` logging level for `org.apache.spark.shuffle.sort.UnsafeShuffleWriter` logger to see what happens inside.

Add the following line to `conf/log4j.properties`:

[source,plaintext]
----
log4j.logger.org.apache.spark.shuffle.sort.UnsafeShuffleWriter=ALL
----

Refer to xref:ROOT:spark-logging.adoc[Logging].

== [[internal-properties]] Internal Properties

=== [[mapStatus]] mapStatus

xref:scheduler:MapStatus.adoc[MapStatus]

Created when UnsafeShuffleWriter is requested to <<closeAndWriteOutput, close internal resources and merge spill files>> (with the xref:storage:BlockManagerId.adoc[] of the <<blockManager, BlockManager>> and `partitionLengths`)

Returned when UnsafeShuffleWriter is requested to <<stop, stop>>

=== [[partitioner]] partitioner

xref:rdd:Partitioner.adoc[Partitioner] (as used by the xref:shuffle:spark-shuffle-BaseShuffleHandle.adoc#dependency[ShuffleDependency] of the <<handle, SerializedShuffleHandle>>)

Used when UnsafeShuffleWriter is requested for the following:

* <<open, open>> (and create a xref:shuffle:ShuffleExternalSorter.adoc[ShuffleExternalSorter] with the given xref:rdd:Partitioner.adoc#numPartitions[number of partitions])

* <<insertRecordIntoSorter, insertRecordIntoSorter>> (and request the xref:rdd:Partitioner.adoc#getPartition[partition for the key])

* <<mergeSpills, mergeSpills>>, <<mergeSpillsWithFileStream, mergeSpillsWithFileStream>> and <<mergeSpillsWithTransferTo, mergeSpillsWithTransferTo>> (for the xref:rdd:Partitioner.adoc#numPartitions[number of partitions] to create partition lengths)

=== [[peakMemoryUsedBytes]] peakMemoryUsedBytes

Peak memory used (in bytes) that is updated exclusively in <<updatePeakMemoryUsed, updatePeakMemoryUsed>> (after requesting the <<sorter, ShuffleExternalSorter>> for xref:shuffle:ShuffleExternalSorter.adoc#getPeakMemoryUsedBytes[getPeakMemoryUsedBytes])

Use <<getPeakMemoryUsedBytes, getPeakMemoryUsedBytes>> to access the current value

=== [[serBuffer]] ByteArrayOutputStream for Serialized Data

{java-javadoc-url}/java/io/ByteArrayOutputStream.html[java.io.ByteArrayOutputStream] of serialized data (written into a byte array of <<DEFAULT_INITIAL_SER_BUFFER_SIZE, 1MB>> initial size)

Used when UnsafeShuffleWriter is requested for the following:

* <<open, open>> (and create the internal <<serOutputStream, SerializationStream>>)

* <<insertRecordIntoSorter, insertRecordIntoSorter>>

Destroyed (`null`) when requested to <<closeAndWriteOutput, close internal resources and merge spill files>>.

=== [[serializer]] serializer

xref:serializer:SerializerInstance.adoc[SerializerInstance] (that is a new instance of the xref:rdd:ShuffleDependency.adoc#serializer[Serializer] of the xref:shuffle:spark-shuffle-BaseShuffleHandle.adoc#dependency[ShuffleDependency] of the <<handle, SerializedShuffleHandle>>)

Used exclusively when UnsafeShuffleWriter is requested to <<open, open>> (and creates the <<serOutputStream, SerializationStream>>)

=== [[serOutputStream]] serOutputStream

xref:serializer:SerializationStream.adoc[SerializationStream] (that is created when the <<serializer, SerializerInstance>> is requested to xref:serializer:SerializerInstance.adoc#serializeStream[serializeStream] with the <<serBuffer, ByteArrayOutputStream>>)

Used when UnsafeShuffleWriter is requested to <<insertRecordIntoSorter, insertRecordIntoSorter>>

Destroyed (`null`) when requested to <<closeAndWriteOutput, close internal resources and merge spill files>>.

=== [[shuffleId]] shuffleId

xref:rdd:ShuffleDependency.adoc#shuffleId[Shuffle ID] (of the <<spark-shuffle-BaseShuffleHandle.adoc#dependency, ShuffleDependency>> of the <<handle, SerializedShuffleHandle>>)

Used exclusively when requested to <<closeAndWriteOutput, close internal resources and merge spill files>>

=== [[sorter]] ShuffleExternalSorter

UnsafeShuffleWriter uses a xref:shuffle:ShuffleExternalSorter.adoc[ShuffleExternalSorter].

ShuffleExternalSorter is created when UnsafeShuffleWriter is requested to <<open, open>> (while being <<creating-instance, created>>) and dereferenced (``null``ed) when requested to <<closeAndWriteOutput, close internal resources and merge spill files>>.

Used when UnsafeShuffleWriter is requested for the following:

* <<updatePeakMemoryUsed, Updating peak memory used>>

* <<write, Writing records>>

* <<closeAndWriteOutput, Closing internal resources and merging spill files>>

* <<insertRecordIntoSorter, Inserting a record>>

* <<stop, Stopping>>

=== [[writeMetrics]] writeMetrics

xref:executor:ShuffleWriteMetrics.adoc[] (of the xref:scheduler:spark-TaskContext.adoc#taskMetrics[TaskMetrics] of the <<taskContext, TaskContext>>)

Used when UnsafeShuffleWriter is requested for the following:

* <<open, open>> (and creates the <<sorter, ShuffleExternalSorter>>)

* <<mergeSpills, mergeSpills>>

* <<mergeSpillsWithFileStream, mergeSpillsWithFileStream>>

* <<mergeSpillsWithTransferTo, mergeSpillsWithTransferTo>>
