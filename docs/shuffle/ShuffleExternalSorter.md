= [[ShuffleExternalSorter]] ShuffleExternalSorter

*ShuffleExternalSorter* is a specialized cache-efficient sorter that sorts arrays of compressed record pointers and partition ids. By using only 8 bytes of space per record in the sorting array, ShuffleExternalSorter can fit more of the array into cache.

.ShuffleExternalSorter and UnsafeShuffleWriter
image::ShuffleExternalSorter.png[align="center"]

== [[creating-instance]] Creating Instance

ShuffleExternalSorter takes the following to be created:

* [[memoryManager]] xref:memory:TaskMemoryManager.adoc[]
* [[blockManager]] xref:storage:BlockManager.adoc[]
* [[taskContext]] xref:scheduler:spark-TaskContext.adoc[]
* [[initialSize]] Initial size
* [[numPartitions]] Number of partitions
* [[conf]] xref:ROOT:SparkConf.adoc[]
* [[writeMetrics]] xref:executor:ShuffleWriteMetrics.adoc[]

[[fileBufferSizeBytes]]
ShuffleExternalSorter uses xref:ROOT:configuration-properties.adoc#spark.shuffle.file.buffer[spark.shuffle.file.buffer] (for `fileBufferSizeBytes`) and xref:ROOT:configuration-properties.adoc#spark.shuffle.spill.numElementsForceSpillThreshold[spark.shuffle.spill.numElementsForceSpillThreshold] (for `numElementsForSpillThreshold`) Spark properties.

ShuffleExternalSorter creates a <<inMemSorter, ShuffleInMemorySorter>> (with `spark.shuffle.sort.useRadixSort` Spark property enabled by default).

ShuffleExternalSorter is created for xref:shuffle:UnsafeShuffleWriter.adoc[UnsafeShuffleWriter].

== [[inMemSorter]] ShuffleInMemorySorter

ShuffleExternalSorter manages a xref:shuffle:ShuffleInMemorySorter.adoc[ShuffleInMemorySorter]:

* ShuffleInMemorySorter is created immediately when ShuffleExternalSorter is

* ShuffleInMemorySorter is requested to xref:shuffle:ShuffleInMemorySorter.adoc#free[free up memory] and dereferenced (``null``ed) when ShuffleExternalSorter is requested to <<cleanupResources, cleanupResources>> and <<closeAndGetSpills, closeAndGetSpills>>

ShuffleExternalSorter uses the ShuffleInMemorySorter when requested for the following:

* <<writeSortedFile, writeSortedFile>>

* <<spill, spill>>

* <<getMemoryUsage, getMemoryUsage>>

* <<growPointerArrayIfNecessary, growPointerArrayIfNecessary>>

* <<insertRecord, insertRecord>>

== [[MemoryConsumer]] ShuffleExternalSorter as MemoryConsumer

ShuffleExternalSorter is a xref:memory:MemoryConsumer.adoc[MemoryConsumer] that can <<spill, spill to disk to free up execution memory>>.

== [[pageSize]] Page Size

ShuffleExternalSorter uses the xref:memory:MemoryConsumer.adoc#pageSize[page size] to be the minimum of `PackedRecordPointer.MAXIMUM_PAGE_SIZE_BYTES` and xref:memory:TaskMemoryManager.adoc#pageSizeBytes[pageSizeBytes], and Tungsten memory mode).

== [[allocatedPages]] allocatedPages

ShuffleExternalSorter uses...FIXME

== [[getMemoryUsage]] getMemoryUsage Internal Method

[source, java]
----
long getMemoryUsage()
----

getMemoryUsage...FIXME

getMemoryUsage is used when...FIXME

== [[writeSortedFile]] writeSortedFile Method

[source, java]
----
void writeSortedFile(
  boolean isLastFile)
----

writeSortedFile...FIXME

writeSortedFile is used when ShuffleExternalSorter is requested to <<spill, spill>> and <<closeAndGetSpills, closeAndGetSpills>>.

== [[cleanupResources]] cleanupResources Internal Method

[source, java]
----
void cleanupResources()
----

cleanupResources...FIXME

cleanupResources is used when...FIXME

== [[spill]] Spilling To Disk

[source, java]
----
long spill(
  long size,
  MemoryConsumer trigger)
----

spill prints out the following INFO message to the logs:

```
Thread [threadId] spilling sort data of [memoryUsage] to disk ([spillsSize] [time|times] so far)
```

spill <<writeSortedFile, writeSortedFile>> (with the `isLastFile` flag disabled).

spill <<freeMemory, frees execution memory>> (and records the memory bytes spilled as `spillSize`).

spill then requests the <<inMemSorter, ShuffleInMemorySorter>> to xref:shuffle:ShuffleInMemorySorter.adoc#reset[reset] followed by requesting the xref:scheduler:spark-TaskContext.adoc#taskMetrics[TaskMetrics] (of the <<taskContext, TaskContext>>) to xref:executor:TaskMetrics.adoc#incMemoryBytesSpilled[increase the memory bytes spilled].

In the end, spill returns the memory bytes spilled (_spill size_).

[NOTE]
====
spill returns `0` when one of the following holds:

* The given `trigger` is not the current ShuffleExternalSorter

* <<inMemSorter, ShuffleInMemorySorter>> is not assigned

* <<inMemSorter, ShuffleInMemorySorter>> manages no xref:shuffle:ShuffleInMemorySorter.adoc#numRecords[records]
====

spill is part of the xref:memory:MemoryConsumer.adoc#spill[MemoryConsumer] contract.

== [[growPointerArrayIfNecessary]] growPointerArrayIfNecessary Method

[source, java]
----
void growPointerArrayIfNecessary()
----

growPointerArrayIfNecessary...FIXME

growPointerArrayIfNecessary is used when...FIXME

== [[closeAndGetSpills]] closeAndGetSpills Method

[source, java]
----
SpillInfo[] closeAndGetSpills()
----

closeAndGetSpills...FIXME

closeAndGetSpills is used when...FIXME

== [[insertRecord]] Inserting Serialized Record Into ShuffleInMemorySorter

[source, java]
----
void insertRecord(
  Object recordBase,
  long recordOffset,
  int length,
  int partitionId)
----

insertRecord requires that the <<inMemSorter, ShuffleInMemorySorter>> is available.

insertRecord...FIXME

insertRecord is used when...FIXME

== [[freeMemory]] freeMemory Method

[source, java]
----
long freeMemory()
----

freeMemory...FIXME

freeMemory is used when...FIXME

== [[getPeakMemoryUsedBytes]] getPeakMemoryUsedBytes Method

[source, java]
----
long getPeakMemoryUsedBytes()
----

getPeakMemoryUsedBytes...FIXME

getPeakMemoryUsedBytes is used when...FIXME

== [[logging]] Logging

Enable `ALL` logging levels for `org.apache.spark.shuffle.sort.ShuffleExternalSorter` logger to see what happens in ShuffleExternalSorter.

Add the following line to `conf/log4j.properties`:

[source,plaintext]
----
log4j.logger.org.apache.spark.shuffle.sort.ShuffleExternalSorter=ALL
----

Refer to xref:ROOT:spark-logging.adoc[Logging].
