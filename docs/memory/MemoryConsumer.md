= [[MemoryConsumer]] MemoryConsumer

*MemoryConsumer* is an abstraction of <<extensions, spillable memory consumers>> of xref:memory:TaskMemoryManager.adoc#consumers[TaskMemoryManager].

MemoryConsumers correspond to individual operators and data structures within a task. The TaskMemoryManager receives memory allocation requests from MemoryConsumers and issues callbacks to consumers in order to trigger <<spill, spilling>> when running low on memory.

A MemoryConsumer basically tracks <<used, how much memory is allocated>>.

Creating a MemoryConsumer requires a link:TaskMemoryManager.adoc[TaskMemoryManager] with optional `pageSize` and a `MemoryMode`.

== [[creating-instance]] Creating Instance

MemoryConsumer takes the following to be created:

* [[taskMemoryManager]] xref:memory:TaskMemoryManager.adoc[TaskMemoryManager]
* [[pageSize]] Page size (in bytes)
* [[mode]] MemoryMode

MemoryConsumer initializes the <<internal-properties, internal properties>>.

== [[extensions]] Extensions

.MemoryConsumers (Direct Implementations and Extensions Only)
[cols="30,70",options="header",width="100%"]
|===
| MemoryConsumer
| Description

| xref:BytesToBytesMap.adoc[BytesToBytesMap]
| [[BytesToBytesMap]] Used in Spark SQL

| HybridRowQueue
| [[HybridRowQueue]]

| LongToUnsafeRowMap
| [[LongToUnsafeRowMap]]

| RowBasedKeyValueBatch
| [[RowBasedKeyValueBatch]]

| ShuffleExternalSorter
| [[ShuffleExternalSorter]]

| xref:shuffle:Spillable.adoc[Spillable]
| [[Spillable]]

| UnsafeExternalSorter
| [[UnsafeExternalSorter]]

|===

== [[contract]][[spill]] Spilling

[source, java]
----
long spill(
  long size,
  MemoryConsumer trigger)
----

CAUTION: FIXME

NOTE: `spill` is used when link:TaskMemoryManager.adoc#acquireExecutionMemory[`TaskMemoryManager` forces `MemoryConsumers` to release memory when requested to acquire execution memory]

== [[used]][[getUsed]] Memory Allocated (Used)

`used` is the amount of memory in use (i.e. allocated) by the MemoryConsumer.

== [[freePage]] Deallocate MemoryBlock

[source, java]
----
void freePage(
  MemoryBlock page)
----

`freePage` is a protected method to deallocate the `MemoryBlock`.

Internally, it decrements <<used, used>> registry by the size of `page` and link:TaskMemoryManager.adoc#freePage[frees the page].

== [[allocateArray]] Allocating Array

[source, java]
----
LongArray allocateArray(
  long size)
----

`allocateArray` allocates `LongArray` of `size` length.

Internally, it link:TaskMemoryManager.adoc#allocatePage[allocates a page] for the requested `size`. The size is recorded in the internal <<used, used>> counter.

However, if it was not possible to allocate the `size` memory, it link:TaskMemoryManager.adoc#showMemoryUsage[shows the current memory usage] and a `OutOfMemoryError` is thrown.

```
Unable to acquire [required] bytes of memory, got [got]
```

allocateArray is used when:

* BytesToBytesMap is requested to xref:memory:BytesToBytesMap.adoc#allocate[allocate]

* ShuffleExternalSorter is requested to xref:shuffle:ShuffleExternalSorter.adoc#growPointerArrayIfNecessary[growPointerArrayIfNecessary]

* ShuffleInMemorySorter is xref:shuffle:ShuffleInMemorySorter.adoc[created] and xref:shuffle:ShuffleInMemorySorter.adoc#reset[reset]

* UnsafeExternalSorter is requested to xref:UnsafeExternalSorter.adoc#growPointerArrayIfNecessary[growPointerArrayIfNecessary]

* UnsafeInMemorySorter is xref:UnsafeInMemorySorter.adoc[created] and xref:UnsafeInMemorySorter.adoc#reset[reset]

* Spark SQL's UnsafeKVExternalSorter is created

== [[acquireMemory]] Acquiring Execution Memory (Allocating Memory)

[source, java]
----
long acquireMemory(
  long size)
----

acquireMemory requests the <<taskMemoryManager, TaskMemoryManager>> to xref:memory:TaskMemoryManager.adoc#acquireExecutionMemory[acquire execution memory] (of the given size).

The memory allocated is then added to the <<used, used>> internal registry.

acquireMemory is used when:

* Spillable is requested to xref:shuffle:Spillable.adoc#maybeSpill[maybeSpill]

* Spark SQL's LongToUnsafeRowMap is requested to ensureAcquireMemory

== [[allocatePage]] Allocating Memory Block (Page)

[source, java]
----
MemoryBlock allocatePage(
  long required)
----

allocatePage...FIXME

allocatePage is used when...FIXME

== [[throwOom]] Throwing OutOfMemoryError

[source, java]
----
void throwOom(
  MemoryBlock page,
  long required)
----

`throwOom`...FIXME

`throwOom` is used when MemoryConsumer is requested to <<allocateArray, allocate an array>> or a <<allocatePage, memory block (page)>> and failed to acquire enough.
