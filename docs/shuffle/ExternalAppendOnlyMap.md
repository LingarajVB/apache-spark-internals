= [[ExternalAppendOnlyMap]] ExternalAppendOnlyMap

*ExternalAppendOnlyMap* is a xref:shuffle:Spillable.adoc[Spillable] of SizeTrackers.

`ExternalAppendOnlyMap[K, V, C]` is a parameterized type of `K` keys, `V` values, and `C` combiner (partial) values.

== [[creating-instance]] Creating Instance

ExternalAppendOnlyMap takes the following to be created:

* [[createCombiner]] createCombiner function (`V => C`)
* [[mergeValue]] mergeValue function (`(C, V) => C`)
* [[mergeCombiners]] mergeCombiners function (`(C, C) => C`)
* [[serializer]] Optional xref:serializer:Serializer.adoc[Serializer] (default: xref:core:SparkEnv.adoc#serializer[system Serializer])
* [[blockManager]] Optional xref:storage:BlockManager.adoc[BlockManager] (default: xref:core:SparkEnv.adoc#blockManager[system BlockManager])
* [[context]] Optional xref:scheduler:spark-TaskContext.adoc[TaskContext] (default: xref:scheduler:spark-TaskContext.adoc#get[current TaskContext])
* [[serializerManager]] Optional xref:serializer:SerializerManager.adoc[SerializerManager] (default: xref:core:SparkEnv.adoc#serializerManager[system SerializerManager])

ExternalAppendOnlyMap is created when:

* Aggregator is requested to xref:rdd:Aggregator.adoc#combineValuesByKey[combineValuesByKey] and xref:rdd:Aggregator.adoc#combineCombinersByKey[combineCombinersByKey]

* CoGroupedRDD is requested to xref:rdd:spark-rdd-CoGroupedRDD.adoc#compute[compute a partition]

== [[currentMap]] SizeTrackingAppendOnlyMap

ExternalAppendOnlyMap manages a SizeTrackingAppendOnlyMap.

A SizeTrackingAppendOnlyMap is created immediately when ExternalAppendOnlyMap is and every time when <<insertAll, insertAll>> and <<forceSpill, forceSpill>> spilled to disk.

SizeTrackingAppendOnlyMap are dereferenced (``null``ed) for the memory to be garbage-collected when <<forceSpill, forceSpill>> and <<freeCurrentMap, freeCurrentMap>>.

SizeTrackingAppendOnlyMap is used when <<insertAll, insertAll>>, <<spill, spill>>, <<forceSpill, forceSpill>> and <<iterator, iterator>>.

== [[insertAll]] Inserting All Key-Value Pairs (from Iterator)

[source, scala]
----
insertAll(
  entries: Iterator[Product2[K, V]]): Unit
----

[[insertAll-update-function]]
insertAll creates an update function that uses the <<mergeValue, mergeValue>> function for an existing value or the <<createCombiner, createCombiner>> function for a new value.

For every key-value pair (from the input iterator), insertAll does the following:

* Requests the <<currentMap, SizeTrackingAppendOnlyMap>> for the estimated size and, if greater than the <<_peakMemoryUsedBytes, _peakMemoryUsedBytes>> metric, updates it.

* xref:shuffle:Spillable.adoc#maybeSpill[Spills to a disk if necessary] and, if spilled, creates a new <<currentMap, SizeTrackingAppendOnlyMap>>

* Requests the <<currentMap, SizeTrackingAppendOnlyMap>> to change value for the current value (with the <<insertAll-update-function, update>> function)

* xref:shuffle:Spillable.adoc#addElementsRead[Increments the elements read counter]

=== [[insertAll-usage]] Usage

insertAll is used when:

* Aggregator is requested to xref:rdd:Aggregator.adoc#combineValuesByKey[combineValuesByKey] and xref:rdd:Aggregator.adoc#combineCombinersByKey[combineCombinersByKey]

* CoGroupedRDD is requested to xref:rdd:spark-rdd-CoGroupedRDD.adoc#compute[compute a partition]

* ExternalAppendOnlyMap is requested to <<insert, insert a key-value pair>>

=== [[insertAll-requirements]] Requirements

insertAll throws an IllegalStateException when the <<currentMap, currentMap>> internal registry is `null`:

[source,plaintext]
----
Cannot insert new elements into a map after calling iterator
----

== [[iterator]] Iterator of "Combined" Pairs

[source, scala]
----
iterator: Iterator[(K, C)]
----

iterator...FIXME

iterator is used when...FIXME

== [[spill]] Spilling to Disk if Necessary

[source, scala]
----
spill(
  collection: SizeTracker): Unit
----

spill...FIXME

spill is used when...FIXME

== [[forceSpill]] Forcing Disk Spilling

[source, scala]
----
forceSpill(): Boolean
----

forceSpill returns a flag to indicate whether spilling to disk has really happened (`true`) or not (`false`).

forceSpill branches off per the current state it is in (and should rather use a state-aware implementation).

When a <<readingIterator, SpillableIterator>> is in use, forceSpill requests it to spill and, if it did, dereferences (``null``ify) the <<currentMap, SizeTrackingAppendOnlyMap>>. forceSpill returns whatever the spilling of the <<readingIterator, SpillableIterator>> returned.

When there is at least one element in the <<currentMap, SizeTrackingAppendOnlyMap>>, forceSpill <<spill, spills>> it. forceSpill then creates a new <<currentMap, SizeTrackingAppendOnlyMap>> and always returns `true`.

In other cases, forceSpill simply returns `false`.

forceSpill is part of the xref:shuffle:Spillable.adoc[Spillable] abstraction.

== [[freeCurrentMap]] Freeing Up SizeTrackingAppendOnlyMap and Releasing Memory

[source, scala]
----
freeCurrentMap(): Unit
----

freeCurrentMap dereferences (``null``ify) the <<currentMap, SizeTrackingAppendOnlyMap>> (if there still was one) followed by xref:shuffle:Spillable.adoc#releaseMemory[releasing all memory].

freeCurrentMap is used when SpillableIterator is requested to destroy itself.

== [[spillMemoryIteratorToDisk]] spillMemoryIteratorToDisk Method

[source, scala]
----
spillMemoryIteratorToDisk(
  inMemoryIterator: Iterator[(K, C)]): DiskMapIterator
----

spillMemoryIteratorToDisk...FIXME

spillMemoryIteratorToDisk is used when...FIXME
