= [[ShuffleDependency]] ShuffleDependency

*ShuffleDependency* is a xref:rdd:spark-rdd-Dependency.adoc[Dependency] on the output of a xref:scheduler:ShuffleMapStage.adoc[ShuffleMapStage] for a <<rdd, key-value pair RDD>>.

ShuffleDependency uses the <<rdd, RDD>> to know the number of (map-side/pre-shuffle) partitions and the <<partitioner, Partitioner>> for the number of (reduce-size/post-shuffle) partitions.

ShuffleDependency is created as a dependency of xref:rdd:ShuffledRDD.adoc[ShuffledRDD]. ShuffleDependency can also be created as a dependency of xref:rdd:spark-rdd-CoGroupedRDD.adoc[CoGroupedRDD] and xref:rdd:spark-rdd-SubtractedRDD.adoc[SubtractedRDD].

== [[creating-instance]] Creating Instance

ShuffleDependency takes the following to be created:

* <<rdd, RDD>> of key-value pairs (`RDD[_ <: Product2[K, V]]`)
* <<partitioner, Partitioner>>
* [[serializer]] xref:serializer:Serializer.adoc[Serializer]
* [[keyOrdering]] Ordering for K keys (`Option[Ordering[K]]`)
* <<aggregator, Aggregator>> (`Option[Aggregator[K, V, C]]`)
* <<mapSideCombine, mapSideCombine>> flag (default: `false`)

When created, ShuffleDependency gets xref:ROOT:SparkContext.adoc#nextShuffleId[shuffle id] (as `shuffleId`).

NOTE: ShuffleDependency uses the xref:rdd:index.adoc#context[input RDD to access `SparkContext`] and so the `shuffleId`.

ShuffleDependency xref:shuffle:ShuffleManager.adoc#registerShuffle[registers itself with `ShuffleManager`] and gets a `ShuffleHandle` (available as <<shuffleHandle, shuffleHandle>> property).

NOTE: ShuffleDependency accesses xref:core:SparkEnv.adoc#shuffleManager[`ShuffleManager` using `SparkEnv`].

In the end, ShuffleDependency xref:core:ContextCleaner.adoc#registerShuffleForCleanup[registers itself for cleanup with `ContextCleaner`].

NOTE: ShuffleDependency accesses the xref:ROOT:SparkContext.adoc#cleaner[optional `ContextCleaner` through `SparkContext`].

NOTE: ShuffleDependency is created when xref:ShuffledRDD.adoc#getDependencies[ShuffledRDD], link:spark-rdd-CoGroupedRDD.adoc#getDependencies[CoGroupedRDD], and link:spark-rdd-SubtractedRDD.adoc#getDependencies[SubtractedRDD] return their RDD dependencies.

== [[shuffleId]] Shuffle ID

Every ShuffleDependency has a unique application-wide *shuffle ID* that is assigned when <<creating-instance, ShuffleDependency is created>> (and is used throughout Spark's code to reference a ShuffleDependency).

Shuffle IDs are tracked by xref:ROOT:SparkContext.adoc#nextShuffleId[SparkContext].

== [[rdd]] Parent RDD

ShuffleDependency is given the parent xref:rdd:RDD.adoc[RDD] of key-value pairs (`RDD[_ <: Product2[K, V]]`).

The parent RDD is available as rdd property that is part of the xref:rdd:spark-rdd-Dependency.adoc#rdd[Dependency] abstraction.

[source,scala]
----
rdd: RDD[Product2[K, V]]
----

== [[partitioner]] Partitioner

ShuffleDependency is given a xref:rdd:Partitioner.adoc[Partitioner] that is used to partition the shuffle output (when xref:shuffle:SortShuffleWriter.adoc[SortShuffleWriter], xref:shuffle:BypassMergeSortShuffleWriter.adoc[BypassMergeSortShuffleWriter] and xref:shuffle:UnsafeShuffleWriter.adoc[UnsafeShuffleWriter] are requested to write).

== [[shuffleHandle]] ShuffleHandle

[source, scala]
----
shuffleHandle: ShuffleHandle
----

shuffleHandle is the `ShuffleHandle` of a ShuffleDependency as assigned eagerly when <<creating-instance, ShuffleDependency was created>>.

shuffleHandle is used to compute link:spark-rdd-CoGroupedRDD.adoc#compute[CoGroupedRDDs], xref:ShuffledRDD.adoc#compute[ShuffledRDD], link:spark-rdd-SubtractedRDD.adoc#compute[SubtractedRDD], and link:spark-sql-ShuffledRowRDD.adoc[ShuffledRowRDD] (to get a link:spark-shuffle-ShuffleReader.adoc[ShuffleReader] for a ShuffleDependency) and when a xref:scheduler:ShuffleMapTask.adoc#runTask[`ShuffleMapTask` runs] (to get a `ShuffleWriter` for a ShuffleDependency).

== [[mapSideCombine]] Map-Size Partial Aggregation Flag

ShuffleDependency uses a mapSideCombine flag that controls whether to perform *map-side partial aggregation* (_map-side combine_) using an <<aggregator, Aggregator>>.

mapSideCombine is disabled (`false`) by default and can be enabled (`true`) for some use cases of xref:rdd:ShuffledRDD.adoc#mapSideCombine[ShuffledRDD].

ShuffleDependency requires that the optional <<aggregator, Aggregator>> is defined when the flag is enabled.

mapSideCombine is used when:

* BlockStoreShuffleReader is requested to xref:shuffle:BlockStoreShuffleReader.adoc#read[read combined records for a reduce task]

* SortShuffleManager is requested to xref:shuffle:SortShuffleManager.adoc#registerShuffle[register a shuffle]

* SortShuffleWriter is requested to xref:shuffle:SortShuffleWriter.adoc#write[write records]

== [[aggregator]] Optional Aggregator

[source, scala]
----
aggregator: Option[Aggregator[K, V, C]] = None
----

`aggregator` is a xref:rdd:Aggregator.adoc[map/reduce-side Aggregator] (for a RDD's shuffle).

`aggregator` is by default undefined (i.e. `None`) when <<creating-instance, ShuffleDependency is created>>.

NOTE: `aggregator` is used when xref:shuffle:SortShuffleWriter.adoc#write[`SortShuffleWriter` writes records] and xref:shuffle:BlockStoreShuffleReader.adoc#read[`BlockStoreShuffleReader` reads combined key-values for a reduce task].
