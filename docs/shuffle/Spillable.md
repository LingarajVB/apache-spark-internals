= [[Spillable]] Spillable

*Spillable* is an extension of the xref:memory:MemoryConsumer.adoc[MemoryConsumer] abstraction for <<implementations, collections>> that can <<spill, spill to disk>>.

`Spillable[C]` is a parameterized type of `C` combiner (partial) values.

== [[creating-instance]] Creating Instance

[[taskMemoryManager]]
Spillable takes a single xref:memory:TaskMemoryManager.adoc[TaskMemoryManager] to be created.

`Spillable` is an abstract class and cannot be created directly. It is created indirectly for the <<implementations, concrete Spillables>>.

== [[extensions]] Extensions

.Spillables
[cols="30,70",options="header",width="100%"]
|===
| Spillable
| Description

| xref:shuffle:ExternalAppendOnlyMap.adoc[ExternalAppendOnlyMap]
| [[ExternalAppendOnlyMap]]

| xref:shuffle:ExternalSorter.adoc[ExternalSorter]
| [[ExternalSorter]]

|===

== [[configuration-properties]] Configuration Properties

=== [[numElementsForceSpillThreshold]] spark.shuffle.spill.numElementsForceSpillThreshold

Spillable uses xref:ROOT:configuration-properties.adoc#spark.shuffle.spill.numElementsForceSpillThreshold[spark.shuffle.spill.numElementsForceSpillThreshold] configuration property to force spilling in-memory objects to disk when requested to <<maybeSpill, maybeSpill>>.

=== [[initialMemoryThreshold]] spark.shuffle.spill.initialMemoryThreshold

Spillable uses xref:ROOT:configuration-properties.adoc#spark.shuffle.spill.initialMemoryThreshold[spark.shuffle.spill.initialMemoryThreshold] configuration property as the initial threshold for the size of a collection (and the minimum memory required to operate properly).

Spillable uses it when requested to <<spill, spill>> and <<releaseMemory, releaseMemory>>

== [[myMemoryThreshold]] Memory Threshold

Spillable uses a threshold for the memory size (in bytes) to know when to <<maybeSpill, spill to disk>>.

When the size of the in-memory collection is above the threshold, Spillable will try to acquire more memory. Unless given all requested memory, Spillable spills to disk.

The memory threshold starts as <<initialMemoryThreshold, spark.shuffle.spill.initialMemoryThreshold>> configuration property and is increased every time Spillable is requested to <<maybeSpill, spill to disk if needed>>, but managed to acquire required memory. The threshold goes back to the <<initialMemoryThreshold, initial value>> when requested to <<releaseMemory, release all memory>>.

Used when Spillable is requested to <<spill, spill>> and <<releaseMemory, releaseMemory>>.

== [[releaseMemory]] Releasing All Memory

[source, scala]
----
releaseMemory(): Unit
----

releaseMemory...FIXME

releaseMemory is used when:

* ExternalAppendOnlyMap is requested to xref:shuffle:ExternalAppendOnlyMap.adoc#freeCurrentMap[freeCurrentMap]

* ExternalSorter is requested to xref:shuffle:ExternalSorter.adoc#stop[stop]

* Spillable is requested to <<maybeSpill, maybeSpill>> and <<spill, spill>> (and spilled to disk in either case)

== [[spill]] Spilling In-Memory Collection to Disk (to Release Memory)

[source, scala]
----
spill(
  collection: C): Unit
----

spill spills the given in-memory collection to disk to release memory

spill is used when:

* ExternalAppendOnlyMap is requested to xref:shuffle:ExternalAppendOnlyMap.adoc#forceSpill[forceSpill]

* Spillable is requested to <<maybeSpill, maybeSpill>>

== [[forceSpill]] forceSpill Method

[source, scala]
----
forceSpill(): Boolean
----

forceSpill forcefully spills the Spillable to disk to release memory

forceSpill is used when Spillable is requested to <<spill, spill an in-memory collection to disk>>.

== [[maybeSpill]] Spilling to Disk if Necessary

[source, scala]
----
maybeSpill(
  collection: C,
  currentMemory: Long): Boolean
----

maybeSpill...FIXME

maybeSpill is used when:

* ExternalAppendOnlyMap is requested to xref:shuffle:ExternalAppendOnlyMap.adoc#insertAll[insertAll]

* ExternalSorter is requested to xref:shuffle:ExternalSorter.adoc#maybeSpillCollection[attempt to spill an in-memory collection to disk if needed]
