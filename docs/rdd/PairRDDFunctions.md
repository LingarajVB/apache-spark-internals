= [[PairRDDFunctions]] PairRDDFunctions
:page-toctitle: Transformations

*PairRDDFunctions* is an extension of RDD API to provide additional <<transformations, transformations>> for RDDs of key-value pairs (`RDD[(K, V)]`).

PairRDDFunctions is available in RDDs of key-value pairs via Scala implicit conversion.

[[transformations]]
.PairRDDFunctions' Transformations
[cols="30m,70",options="header",width="100%"]
|===
| Method
| Description

| aggregateByKey
a| [[aggregateByKey]]

[source, scala]
----
aggregateByKey[U: ClassTag](
  zeroValue: U)(
    seqOp: (U, V) => U,
    combOp: (U, U) => U): RDD[(K, U)]
aggregateByKey[U: ClassTag](
  zeroValue: U, numPartitions: Int)(
    seqOp: (U, V) => U,
    combOp: (U, U) => U): RDD[(K, U)]
aggregateByKey[U: ClassTag](
  zeroValue: U, partitioner: Partitioner)(
    seqOp: (U, V) => U,
    combOp: (U, U) => U): RDD[(K, U)]
----

| combineByKey
a| [[combineByKey]]

[source, scala]
----
combineByKey[C](
  createCombiner: V => C,
  mergeValue: (C, V) => C,
  mergeCombiners: (C, C) => C): RDD[(K, C)]
combineByKey[C](
  createCombiner: V => C,
  mergeValue: (C, V) => C,
  mergeCombiners: (C, C) => C,
  numPartitions: Int): RDD[(K, C)]
combineByKey[C](
  createCombiner: V => C,
  mergeValue: (C, V) => C,
  mergeCombiners: (C, C) => C,
  partitioner: Partitioner,
  mapSideCombine: Boolean = true,
  serializer: Serializer = null): RDD[(K, C)]
----

| countApproxDistinctByKey
a| [[countApproxDistinctByKey]]

[source, scala]
----
countApproxDistinctByKey(
  relativeSD: Double = 0.05): RDD[(K, Long)]
countApproxDistinctByKey(
  relativeSD: Double,
  numPartitions: Int): RDD[(K, Long)]
countApproxDistinctByKey(
  relativeSD: Double,
  partitioner: Partitioner): RDD[(K, Long)]
countApproxDistinctByKey(
  p: Int,
  sp: Int,
  partitioner: Partitioner): RDD[(K, Long)]
----

| flatMapValues
a| [[flatMapValues]]

[source, scala]
----
flatMapValues[U](
  f: V => TraversableOnce[U]): RDD[(K, U)]
----

| foldByKey
a| [[foldByKey]]

[source, scala]
----
foldByKey(
  zeroValue: V)(
    func: (V, V) => V): RDD[(K, V)]
foldByKey(
  zeroValue: V, numPartitions: Int)(
    func: (V, V) => V): RDD[(K, V)]
foldByKey(
  zeroValue: V,
  partitioner: Partitioner)(
    func: (V, V) => V): RDD[(K, V)]
----

| mapValues
a| [[mapValues]]

[source, scala]
----
mapValues[U](
  f: V => U): RDD[(K, U)]
----

| partitionBy
a| [[partitionBy]]

[source, scala]
----
partitionBy(
  partitioner: Partitioner): RDD[(K, V)]
----

| saveAsHadoopDataset
a| [[saveAsHadoopDataset]]

[source, scala]
----
saveAsHadoopDataset(
  conf: JobConf): Unit
----

`saveAsHadoopDataset` uses the `SparkHadoopWriter` utility to <<spark-internal-io-SparkHadoopWriter.md#write, write the key-value RDD out>> with a <<spark-internal-io-HadoopMapRedWriteConfigUtil.md#, HadoopMapRedWriteConfigUtil>> (for the given Hadoop https://hadoop.apache.org/docs/r2.7.3/api/org/apache/hadoop/mapred/JobConf.html[JobConf])

| saveAsHadoopFile
a| [[saveAsHadoopFile]]

[source, scala]
----
saveAsHadoopFile(
  path: String,
  keyClass: Class[_],
  valueClass: Class[_],
  outputFormatClass: Class[_ <: OutputFormat[_, _]],
  codec: Class[_ <: CompressionCodec]): Unit
saveAsHadoopFile(
  path: String,
  keyClass: Class[_],
  valueClass: Class[_],
  outputFormatClass: Class[_ <: OutputFormat[_, _]],
  conf: JobConf = new JobConf(self.context.hadoopConfiguration),
  codec: Option[Class[_ <: CompressionCodec]] = None): Unit
saveAsHadoopFile[F <: OutputFormat[K, V]](
  path: String)(implicit fm: ClassTag[F]): Unit
saveAsHadoopFile[F <: OutputFormat[K, V]](
  path: String,
  codec: Class[_ <: CompressionCodec])(implicit fm: ClassTag[F]): Unit
----

| saveAsNewAPIHadoopDataset
a| [[saveAsNewAPIHadoopDataset]]

[source, scala]
----
saveAsNewAPIHadoopDataset(
  conf: Configuration): Unit
----

Saves this RDD of key-value pairs (`RDD[K,V]`) to any Hadoop-supported storage system with new Hadoop API (using a Hadoop https://hadoop.apache.org/docs/r2.7.3/api/org/apache/hadoop/conf/Configuration.html[Configuration] object for that storage system).

The configuration should set relevant output params (an https://hadoop.apache.org/docs/r2.7.3/api/org/apache/hadoop/mapreduce/OutputFormat.html[output format], output paths, e.g. a table name to write to) in the same way as it would be configured for a Hadoop MapReduce job.

`saveAsNewAPIHadoopDataset` uses the `SparkHadoopWriter` utility to <<spark-internal-io-SparkHadoopWriter.md#write, write the key-value RDD out>> with a <<spark-internal-io-HadoopMapReduceWriteConfigUtil.md#, HadoopMapReduceWriteConfigUtil>> (for the given Hadoop https://hadoop.apache.org/docs/r2.7.3/api/org/apache/hadoop/conf/Configuration.html[Configuration])

| saveAsNewAPIHadoopFile
a| [[saveAsNewAPIHadoopFile]]

[source, scala]
----
saveAsNewAPIHadoopFile(
  path: String,
  keyClass: Class[_],
  valueClass: Class[_],
  outputFormatClass: Class[_ <: NewOutputFormat[_, _]],
  conf: Configuration = self.context.hadoopConfiguration): Unit
saveAsNewAPIHadoopFile[F <: NewOutputFormat[K, V]](
  path: String)(implicit fm: ClassTag[F]): Unit
----

|===

== [[reduceByKey]][[groupByKey]] groupByKey and reduceByKey

`reduceByKey` is sort of a particular case of <<aggregateByKey, aggregateByKey>>.

You may want to look at the number of partitions from another angle.

It may often not be important to have a given number of partitions upfront (at RDD creation time upon spark-data-sources.md[loading data from data sources]), so only "regrouping" the data by key after it is an RDD might be...the key (_pun not intended_).

You can use `groupByKey` or another PairRDDFunctions method to have a key in one processing flow.

You could use `partitionBy` that is available for RDDs to be RDDs of tuples, i.e. `PairRDD`:

```
rdd.keyBy(_.kind)
  .partitionBy(new HashPartitioner(PARTITIONS))
  .foreachPartition(...)
```

Think of situations where `kind` has low cardinality or highly skewed distribution and using the technique for partitioning might be not an optimal solution.

You could do as follows:

```
rdd.keyBy(_.kind).reduceByKey(....)
```

or `mapValues` or plenty of other solutions. _FIXME, man_.

== [[combineByKeyWithClassTag]] combineByKeyWithClassTag

[source, scala]
----
combineByKeyWithClassTag[C](
  createCombiner: V => C,
  mergeValue: (C, V) => C,
  mergeCombiners: (C, C) => C)(implicit ct: ClassTag[C]): RDD[(K, C)] // <1>
combineByKeyWithClassTag[C](
  createCombiner: V => C,
  mergeValue: (C, V) => C,
  mergeCombiners: (C, C) => C,
  numPartitions: Int)(implicit ct: ClassTag[C]): RDD[(K, C)] // <2>
combineByKeyWithClassTag[C](
  createCombiner: V => C,
  mergeValue: (C, V) => C,
  mergeCombiners: (C, C) => C,
  partitioner: Partitioner,
  mapSideCombine: Boolean = true,
  serializer: Serializer = null)(implicit ct: ClassTag[C]): RDD[(K, C)]
----
<1> Uses the rdd:Partitioner.md#defaultPartitioner[default partitioner]
<2> Uses a rdd:HashPartitioner.md[HashPartitioner] with the given number of partitions

combineByKeyWithClassTag creates an rdd:Aggregator.md[Aggregator] for the given aggregation functions.

combineByKeyWithClassTag branches off per the given rdd:Partitioner.md[Partitioner].

If the input partitioner and the RDD's are the same, combineByKeyWithClassTag simply rdd:spark-rdd-transformations.md#mapPartitions[mapPartitions] on the RDD with the following arguments:

* Iterator of the rdd:Aggregator.md#combineValuesByKey[Aggregator]

* preservesPartitioning flag turned on

If the input partitioner is different than the RDD's, combineByKeyWithClassTag creates a rdd:ShuffledRDD.md[ShuffledRDD] (with the Serializer, the Aggregator, and the mapSideCombine flag).

=== [[combineByKeyWithClassTag-usage]] Usage

combineByKeyWithClassTag lays the foundation for the following transformations:

* <<aggregateByKey, aggregateByKey>>
* <<combineByKey, combineByKey>>
* <<countApproxDistinctByKey, countApproxDistinctByKey>>
* <<foldByKey, foldByKey>>
* <<groupByKey, groupByKey>>
* <<reduceByKey, reduceByKey>>

=== [[combineByKeyWithClassTag-requirements]] Requirements

combineByKeyWithClassTag requires that the mergeCombiners is defined (not-``null``) or throws an IllegalArgumentException:

[source,plaintext]
----
mergeCombiners must be defined
----

combineByKeyWithClassTag throws a SparkException for the keys being of type array with the mapSideCombine flag enabled:

[source,plaintext]
----
Cannot use map-side combining with array keys.
----

combineByKeyWithClassTag throws a SparkException for the keys being of type array with the partitioner being a rdd:HashPartitioner.md[HashPartitioner]:

[source,plaintext]
----
HashPartitioner cannot partition array keys.
----

=== [[combineByKeyWithClassTag-example]] Example

[source,scala]
----
val nums = sc.parallelize(0 to 9, numSlices = 4)
val groups = nums.keyBy(_ % 2)
def createCombiner(n: Int) = {
  println(s"createCombiner($n)")
  n
}
def mergeValue(n1: Int, n2: Int) = {
  println(s"mergeValue($n1, $n2)")
  n1 + n2
}
def mergeCombiners(c1: Int, c2: Int) = {
  println(s"mergeCombiners($c1, $c2)")
  c1 + c2
}
val countByGroup = groups.combineByKeyWithClassTag(
  createCombiner,
  mergeValue,
  mergeCombiners)
println(countByGroup.toDebugString)
/*
(4) ShuffledRDD[3] at combineByKeyWithClassTag at <console>:31 []
 +-(4) MapPartitionsRDD[1] at keyBy at <console>:25 []
    |  ParallelCollectionRDD[0] at parallelize at <console>:24 []
*/
----
