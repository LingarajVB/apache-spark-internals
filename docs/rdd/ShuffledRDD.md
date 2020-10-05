= [[ShuffledRDD]] ShuffledRDD

*ShuffledRDD* is an xref:rdd:RDD.adoc[RDD] of key-value pairs that represents a *shuffle step* in a xref:spark-rdd-lineage.adoc[RDD lineage].

ShuffledRDD is given an <<prev, RDD>> of key-value pairs of K and V types, respectively, when <<creating-instance, created>> and <<compute, computes>> key-value pairs of K and C types, respectively.

ShuffledRDD is <<creating-instance, created>> for the following RDD transformations:

* xref:spark-rdd-OrderedRDDFunctions.adoc#sortByKey[OrderedRDDFunctions.sortByKey] and xref:spark-rdd-OrderedRDDFunctions.adoc#repartitionAndSortWithinPartitions[OrderedRDDFunctions.repartitionAndSortWithinPartitions]

* xref:rdd:PairRDDFunctions.adoc#combineByKeyWithClassTag[PairRDDFunctions.combineByKeyWithClassTag] and xref:rdd:PairRDDFunctions.adoc#partitionBy[PairRDDFunctions.partitionBy]

* xref:spark-rdd-transformations.adoc#coalesce[RDD.coalesce] (with `shuffle` flag enabled)

ShuffledRDD uses custom <<ShuffledRDDPartition, ShuffledRDDPartition>> partitions.

[[isBarrier]]
ShuffledRDD has xref:rdd:RDD.adoc#isBarrier[isBarrier] flag always disabled (`false`).

== [[creating-instance]] Creating Instance

ShuffledRDD takes the following to be created:

* [[prev]] Previous xref:rdd:RDD.adoc[RDD] of key-value pairs (`RDD[_ <: Product2[K, V]]`)
* [[part]] xref:rdd:Partitioner.adoc[Partitioner]

== [[mapSideCombine]][[setMapSideCombine]] Map-Side Combine Flag

ShuffledRDD uses a *map-side combine* flag to create a xref:rdd:ShuffleDependency.adoc[ShuffleDependency] when requested for the <<getDependencies, dependencies>> (there is always only one).

The flag is disabled (`false`) by default and can be changed using setMapSideCombine method.

[source,scala]
----
setMapSideCombine(
  mapSideCombine: Boolean): ShuffledRDD[K, V, C]
----

setMapSideCombine is used for xref:rdd:PairRDDFunctions.adoc#combineByKeyWithClassTag[PairRDDFunctions.combineByKeyWithClassTag] transformation (which defaults to the flag enabled).

== [[compute]] Computing Partition

[source, scala]
----
compute(
  split: Partition,
  context: TaskContext): Iterator[(K, C)]
----

compute requests the only xref:rdd:RDD.adoc#dependencies[dependency] (that is assumed a xref:rdd:ShuffleDependency.adoc[ShuffleDependency]) for the xref:rdd:ShuffleDependency.adoc#shuffleHandle[ShuffleHandle].

compute uses the xref:core:SparkEnv.adoc[SparkEnv] to access the xref:core:SparkEnv.adoc#shuffleManager[ShuffleManager].

compute requests the xref:shuffle:ShuffleManager.adoc#shuffleManager[ShuffleManager] for the xref:shuffle:ShuffleManager.adoc#getReader[ShuffleReader] (for the ShuffleHandle, the xref:rdd:spark-rdd-Partition.adoc[partition]).

In the end, compute requests the ShuffleReader to xref:shuffle:spark-shuffle-ShuffleReader.adoc#read[read] the combined key-value pairs (of type `(K, C)`).

compute is part of the xref:rdd:RDD.adoc#compute[RDD] abstraction.

== [[getPreferredLocations]] Placement Preferences of Partition

[source, scala]
----
getPreferredLocations(
  partition: Partition): Seq[String]
----

getPreferredLocations requests `MapOutputTrackerMaster` for the xref:scheduler:MapOutputTrackerMaster.adoc#getPreferredLocationsForShuffle[preferred locations] of the given xref:rdd:spark-rdd-Partition.adoc[partition] (xref:storage:BlockManager.adoc[BlockManagers] with the most map outputs).

getPreferredLocations uses SparkEnv to access the current xref:core:SparkEnv.adoc#mapOutputTracker[MapOutputTrackerMaster].

getPreferredLocations is part of the xref:rdd:RDD.adoc#compute[RDD] abstraction.

== [[getDependencies]] Dependencies

[source, scala]
----
getDependencies: Seq[Dependency[_]]
----

getDependencies uses the <<userSpecifiedSerializer, user-specified Serializer>> if defined or requests the current xref:serializer:SerializerManager.adoc[SerializerManager] for xref:serializer:SerializerManager.adoc#getSerializer[one].

getDependencies uses the <<mapSideCombine, mapSideCombine>> internal flag for the types of the keys and values (i.e. `K` and `C` or `K` and `V` when the flag is enabled or not, respectively).

In the end, getDependencies returns a single xref:rdd:ShuffleDependency.adoc[ShuffleDependency] (with the <<prev, previous RDD>>, the <<part, Partitioner>>, and the Serializer).

getDependencies is part of the xref:rdd:RDD.adoc#getDependencies[RDD] abstraction.

== [[ShuffledRDDPartition]] ShuffledRDDPartition

ShuffledRDDPartition gets an `index` to be created (that in turn is the index of partitions as calculated by the xref:rdd:Partitioner.adoc[Partitioner] of a <<ShuffledRDD, ShuffledRDD>>).

== Demos

=== Demo: ShuffledRDD and coalesce

[source,plaintext]
----
val data = sc.parallelize(0 to 9)
val coalesced = data.coalesce(numPartitions = 4, shuffle = true)
scala> println(coalesced.toDebugString)
(4) MapPartitionsRDD[9] at coalesce at <pastie>:75 []
 |  CoalescedRDD[8] at coalesce at <pastie>:75 []
 |  ShuffledRDD[7] at coalesce at <pastie>:75 []
 +-(16) MapPartitionsRDD[6] at coalesce at <pastie>:75 []
    |   ParallelCollectionRDD[5] at parallelize at <pastie>:74 []
----

=== Demo: ShuffledRDD and sortByKey

[source,plaintext]
----
val data = sc.parallelize(0 to 9)
val grouped = rdd.groupBy(_ % 2)
val sorted = grouped.sortByKey(numPartitions = 2)
scala> println(sorted.toDebugString)
(2) ShuffledRDD[15] at sortByKey at <console>:74 []
 +-(4) ShuffledRDD[12] at groupBy at <console>:74 []
    +-(4) MapPartitionsRDD[11] at groupBy at <console>:74 []
       |  MapPartitionsRDD[9] at coalesce at <pastie>:75 []
       |  CoalescedRDD[8] at coalesce at <pastie>:75 []
       |  ShuffledRDD[7] at coalesce at <pastie>:75 []
       +-(16) MapPartitionsRDD[6] at coalesce at <pastie>:75 []
          |   ParallelCollectionRDD[5] at parallelize at <pastie>:74 []
----

== [[internal-properties]] Internal Properties

[cols="30m,70",options="header",width="100%"]
|===
| Name
| Description

| userSpecifiedSerializer
a| [[userSpecifiedSerializer]] User-specified xref:serializer:Serializer.adoc[Serializer] for the single xref:rdd:ShuffleDependency.adoc[ShuffleDependency] dependency

[source, scala]
----
userSpecifiedSerializer: Option[Serializer] = None
----

`userSpecifiedSerializer` is undefined (`None`) by default and can be changed using `setSerializer` method (that is used for xref:rdd:PairRDDFunctions.adoc#combineByKeyWithClassTag[PairRDDFunctions.combineByKeyWithClassTag] transformation).

|===
