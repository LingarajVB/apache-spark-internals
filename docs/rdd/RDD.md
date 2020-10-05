= [[RDD]] RDD -- Description of Distributed Computation
:navtitle: RDD

[[T]]
RDD is a description of a fault-tolerant and resilient computation over a possibly distributed collection of records (of type `T`).

== [[contract]] RDD Contract

=== [[compute]] Computing Partition (in TaskContext)

[source, scala]
----
compute(
  split: Partition,
  context: TaskContext): Iterator[T]
----

compute computes the input `split` xref:rdd:spark-rdd-partitions.adoc[partition] in the xref:scheduler:spark-TaskContext.adoc[TaskContext] to produce a collection of values (of type `T`).

compute is implemented by any type of RDD in Spark and is called every time the records are requested unless RDD is xref:rdd:spark-rdd-caching.adoc[cached] or xref:ROOT:rdd-checkpointing.adoc[checkpointed] (and the records can be read from an external storage, but this time closer to the compute node).

When an RDD is xref:rdd:spark-rdd-caching.adoc[cached], for specified xref:storage:StorageLevel.adoc[storage levels] (i.e. all but `NONE`)...FIXME

compute runs on the xref:ROOT:spark-driver.adoc[driver].

compute is used when RDD is requested to <<computeOrReadCheckpoint, computeOrReadCheckpoint>>.

=== [[getPartitions]] Partitions

[source, scala]
----
getPartitions: Array[Partition]
----

getPartitions is used when RDD is requested for the <<partitions, partitions>> (called only once as the value is cached afterwards).

=== [[getDependencies]] Dependencies

[source, scala]
----
getDependencies: Seq[Dependency[_]]
----

getDependencies is used when RDD is requested for the <<dependencies, dependencies>> (called only once as the value is cached afterwards).

=== [[getPreferredLocations]] Preferred Locations (Placement Preferences)

[source, scala]
----
getPreferredLocations(
  split: Partition): Seq[String] = Nil
----

getPreferredLocations is used when RDD is requested for the <<preferredLocations, preferred locations>> of a given xref:rdd:spark-rdd-Partition.adoc[partition].

=== [[partitioner]] Partitioner

[source, scala]
----
partitioner: Option[Partitioner] = None
----

RDD can have a xref:rdd:Partitioner.adoc[Partitioner] defined.

== [[extensions]][[implementations]] (Subset of) Available RDDs

[cols="30,70",options="header",width="100%"]
|===
| RDD
| Description

| xref:rdd:spark-rdd-CoGroupedRDD.adoc[CoGroupedRDD]
| [[CoGroupedRDD]]

| CoalescedRDD
| [[CoalescedRDD]] Result of xref:rdd:spark-rdd-partitions.adoc#repartition[repartition] or xref:rdd:spark-rdd-partitions.adoc#coalesce[coalesce] transformations

| xref:rdd:spark-rdd-HadoopRDD.adoc[HadoopRDD]
| [[HadoopRDD]] Allows for reading data stored in HDFS using the older MapReduce API. The most notable use case is the return RDD of `SparkContext.textFile`.

| xref:rdd:spark-rdd-MapPartitionsRDD.adoc[MapPartitionsRDD]
| [[MapPartitionsRDD]] Result of calling map-like operations (e.g. `map`, `flatMap`, `filter`, xref:rdd:spark-rdd-transformations.adoc#mapPartitions[mapPartitions])

| xref:rdd:spark-rdd-ParallelCollectionRDD.adoc[ParallelCollectionRDD]
| [[ParallelCollectionRDD]]

| xref:rdd:ShuffledRDD.adoc[ShuffledRDD]
| [[ShuffledRDD]] Result of "shuffle" operators (e.g. xref:rdd:spark-rdd-partitions.adoc#repartition[repartition] or xref:rdd:spark-rdd-partitions.adoc#coalesce[coalesce])

|===

== [[creating-instance]] Creating Instance

RDD takes the following to be created:

* [[_sc]] xref:ROOT:SparkContext.adoc[]
* [[deps]] *Parent RDDs*, i.e. xref:rdd:spark-rdd-Dependency.adoc[Dependencies] (that have to be all computed successfully before this RDD)

RDD is an abstract class and cannot be created directly. It is created indirectly for the <<implementations, concrete RDDs>>.

== [[storageLevel]][[getStorageLevel]] StorageLevel

RDD can have a xref:storage:StorageLevel.adoc[StorageLevel] specified. The default StorageLevel is xref:storage:StorageLevel.adoc#NONE[NONE].

storageLevel can be specified using <<persist, persist>> method.

storageLevel becomes NONE again after <<unpersist, unpersisting>>.

The current StorageLevel is available using `getStorageLevel` method.

[source, scala]
----
getStorageLevel: StorageLevel
----

== [[id]] Unique Identifier

[source, scala]
----
id: Int
----

id is an *unique identifier* (aka *RDD ID*) in the given <<_sc, SparkContext>>.

id requests the <<sc, SparkContext>> for xref:ROOT:SparkContext.adoc#newRddId[newRddId] right when RDD is created.

== [[isBarrier_]][[isBarrier]] Barrier Stage

An RDD can be part of a xref:ROOT:spark-barrier-execution-mode.adoc#barrier-stage[barrier stage]. By default, `isBarrier` flag is enabled (`true`) when:

. There are no xref:rdd:ShuffleDependency.adoc[ShuffleDependencies] among the <<dependencies, RDD dependencies>>

. There is at least one xref:rdd:spark-rdd-Dependency.adoc#rdd[parent RDD] that has the flag enabled

xref:rdd:ShuffledRDD.adoc[ShuffledRDD] has the flag always disabled.

xref:rdd:spark-rdd-MapPartitionsRDD.adoc[MapPartitionsRDD] is the only one RDD that can have the flag enabled.

== [[getOrCompute]] Getting Or Computing RDD Partition

[source, scala]
----
getOrCompute(
  partition: Partition,
  context: TaskContext): Iterator[T]
----

`getOrCompute` creates a xref:storage:BlockId.adoc#RDDBlockId[RDDBlockId] for the <<id, RDD id>> and the link:spark-rdd-Partition.adoc#index[partition index].

`getOrCompute` requests the `BlockManager` to xref:storage:BlockManager.adoc#getOrElseUpdate[getOrElseUpdate] for the block ID (with the <<storageLevel, storage level>> and the `makeIterator` function).

NOTE: `getOrCompute` uses xref:core:SparkEnv.adoc#get[SparkEnv] to access the current xref:core:SparkEnv.adoc#blockManager[BlockManager].

[[getOrCompute-readCachedBlock]]
`getOrCompute` records whether...FIXME (readCachedBlock)

`getOrCompute` branches off per the response from the xref:storage:BlockManager.adoc#getOrElseUpdate[BlockManager] and whether the internal `readCachedBlock` flag is now on or still off. In either case, `getOrCompute` creates an link:spark-InterruptibleIterator.adoc[InterruptibleIterator].

NOTE: link:spark-InterruptibleIterator.adoc[InterruptibleIterator] simply delegates to a wrapped internal `Iterator`, but allows for link:spark-TaskContext.adoc#isInterrupted[task killing functionality].

For a `BlockResult` available and `readCachedBlock` flag on, `getOrCompute`...FIXME

For a `BlockResult` available and `readCachedBlock` flag off, `getOrCompute`...FIXME

NOTE: The `BlockResult` could be found in a local block manager or fetched from a remote block manager. It may also have been stored (persisted) just now. In either case, the `BlockResult` is available (and xref:storage:BlockManager.adoc#getOrElseUpdate[BlockManager.getOrElseUpdate] gives a `Left` value with the `BlockResult`).

For `Right(iter)` (regardless of the value of `readCachedBlock` flag since...FIXME), `getOrCompute`...FIXME

NOTE: xref:storage:BlockManager.adoc#getOrElseUpdate[BlockManager.getOrElseUpdate] gives a `Right(iter)` value to indicate an error with a block.

NOTE: `getOrCompute` is used on Spark executors.

NOTE: `getOrCompute` is used exclusively when RDD is requested for the <<iterator, iterator over values in a partition>>.

== [[dependencies]] RDD Dependencies

[source, scala]
----
dependencies: Seq[Dependency[_]]
----

`dependencies` returns the link:spark-rdd-Dependency.adoc[dependencies of a RDD].

NOTE: `dependencies` is a final method that no class in Spark can ever override.

Internally, `dependencies` checks out whether the RDD is xref:ROOT:rdd-checkpointing.adoc[checkpointed] and acts accordingly.

For a RDD being checkpointed, `dependencies` returns a single-element collection with a link:spark-rdd-NarrowDependency.adoc#OneToOneDependency[OneToOneDependency].

For a non-checkpointed RDD, `dependencies` collection is computed using <<contract, `getDependencies` method>>.

NOTE: `getDependencies` method is an abstract method that custom RDDs are required to provide.

== [[iterator]] Accessing Records For Partition Lazily

[source, scala]
----
iterator(
  split: Partition,
  context: TaskContext): Iterator[T]
----

iterator <<getOrCompute, gets or computes the `split` partition>> when xref:rdd:spark-rdd-caching.adoc[cached] or <<computeOrReadCheckpoint, computes it (possibly by reading from checkpoint)>>.

== [[checkpointRDD]] Getting CheckpointRDD

[source, scala]
----
checkpointRDD: Option[CheckpointRDD[T]]
----

checkpointRDD gives the CheckpointRDD from the <<checkpointData, checkpointData>> internal registry if available (if the RDD was checkpointed).

checkpointRDD is used when RDD is requested for the <<dependencies, dependencies>>, <<partitions, partitions>> and <<preferredLocations, preferredLocations>>.

== [[isCheckpointedAndMaterialized]] isCheckpointedAndMaterialized Method

[source, scala]
----
isCheckpointedAndMaterialized: Boolean
----

isCheckpointedAndMaterialized...FIXME

isCheckpointedAndMaterialized is used when RDD is requested to <<computeOrReadCheckpoint, computeOrReadCheckpoint>>, <<localCheckpoint, localCheckpoint>> and <<isCheckpointed, isCheckpointed>>.

== [[getNarrowAncestors]] getNarrowAncestors Method

[source, scala]
----
getNarrowAncestors: Seq[RDD[_]]
----

getNarrowAncestors...FIXME

getNarrowAncestors is used when StageInfo is requested to xref:scheduler:spark-scheduler-StageInfo.adoc#fromStage[fromStage].

== [[toLocalIterator]] toLocalIterator Method

[source, scala]
----
toLocalIterator: Iterator[T]
----

toLocalIterator...FIXME

== [[persist]] Persisting RDD

[source, scala]
----
persist(): this.type
persist(
  newLevel: StorageLevel): this.type
----

Refer to xref:rdd:spark-rdd-caching.adoc#persist[Persisting RDD].

== [[persist-internal]] persist Internal Method

[source, scala]
----
persist(
  newLevel: StorageLevel,
  allowOverride: Boolean): this.type
----

persist...FIXME

persist (private) is used when RDD is requested to <<persist, persist>> and <<localCheckpoint, localCheckpoint>>.

== [[unpersist]] unpersist Method

[source, scala]
----
unpersist(blocking: Boolean = true): this.type
----

unpersist...FIXME

== [[localCheckpoint]] localCheckpoint Method

[source, scala]
----
localCheckpoint(): this.type
----

localCheckpoint marks this RDD for xref:ROOT:rdd-checkpointing.adoc[local checkpointing] using Spark's caching layer.

== [[computeOrReadCheckpoint]] Computing Partition or Reading From Checkpoint

[source, scala]
----
computeOrReadCheckpoint(
  split: Partition,
  context: TaskContext): Iterator[T]
----

computeOrReadCheckpoint reads `split` partition from a checkpoint (<<isCheckpointedAndMaterialized, if available already>>) or <<compute, computes it>> yourself.

computeOrReadCheckpoint is used when RDD is requested to <<iterator, compute records for a partition>> or <<getOrCompute, getOrCompute>>.

== [[getNumPartitions]] Getting Number of Partitions

[source, scala]
----
getNumPartitions: Int
----

getNumPartitions gives the number of partitions of a RDD.

[source, scala]
----
scala> sc.textFile("README.md").getNumPartitions
res0: Int = 2

scala> sc.textFile("README.md", 5).getNumPartitions
res1: Int = 5
----

== [[preferredLocations]] Defining Placement Preferences of RDD Partition

[source, scala]
----
preferredLocations(
  split: Partition): Seq[String]
----

preferredLocations requests the CheckpointRDD for <<checkpointRDD, placement preferences>> (if the RDD is checkpointed) or <<getPreferredLocations, calculates them itself>>.

preferredLocations is a template method that uses  <<getPreferredLocations, getPreferredLocations>> that custom RDDs can override to specify placement preferences for a partition. getPreferredLocations defines no placement preferences by default.

preferredLocations is mainly used when DAGScheduler is requested to xref:scheduler:DAGScheduler.adoc#getPreferredLocs[compute the preferred locations for missing partitions].

== [[partitions]] Accessing RDD Partitions

[source, scala]
----
partitions: Array[Partition]
----

partitions returns the xref:rdd:spark-rdd-partitions.adoc[Partitions] of a `RDD`.

partitions requests CheckpointRDD for the <<checkpointRDD, partitions>> (if the RDD is checkpointed) or <<getPartitions, finds them itself>> and cache (in <<partitions_, partitions_>> internal registry that is used next time).

Partitions have the property that their internal index should be equal to their position in the owning RDD.

== [[markCheckpointed]] markCheckpointed Method

[source, scala]
----
markCheckpointed(): Unit
----

markCheckpointed...FIXME

markCheckpointed is used when...FIXME

== [[doCheckpoint]] doCheckpoint Method

[source, scala]
----
doCheckpoint(): Unit
----

doCheckpoint...FIXME

doCheckpoint is used when SparkContext is requested to xref:ROOT:SparkContext.adoc#runJob[run a job (synchronously)].

== [[checkpoint]] Reliable Checkpointing -- checkpoint Method

[source, scala]
----
checkpoint(): Unit
----

checkpoint...FIXME

checkpoint is used when...FIXME

== [[isReliablyCheckpointed]] isReliablyCheckpointed Method

[source, scala]
----
isReliablyCheckpointed: Boolean
----

isReliablyCheckpointed...FIXME

isReliablyCheckpointed is used when...FIXME

== [[getCheckpointFile]] getCheckpointFile Method

[source, scala]
----
getCheckpointFile: Option[String]
----

getCheckpointFile...FIXME

getCheckpointFile is used when...FIXME
