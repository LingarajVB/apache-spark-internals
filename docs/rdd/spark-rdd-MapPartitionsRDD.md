== [[MapPartitionsRDD]] MapPartitionsRDD

`MapPartitionsRDD` is an xref:rdd:RDD.adoc[RDD] that has exactly xref:rdd:spark-rdd-NarrowDependency.adoc#OneToOneDependency[one-to-one narrow dependency] on the <<prev, parent RDD>> and "describes" a distributed computation of the given <<f, function>> to every RDD partition.

`MapPartitionsRDD` is <<creating-instance, created>> when:

* `PairRDDFunctions` (`RDD[(K, V)]`) is requested to xref:rdd:PairRDDFunctions.adoc#mapValues[mapValues] and xref:rdd:PairRDDFunctions.adoc#flatMapValues[flatMapValues] (with the <<preservesPartitioning, preservesPartitioning>> flag enabled)

* `RDD[T]` is requested to <<spark-rdd-transformations.adoc#map, map>>, <<spark-rdd-transformations.adoc#flatMap, flatMap>>, <<spark-rdd-transformations.adoc#filter, filter>>, <<spark-rdd-transformations.adoc#glom, glom>>, <<spark-rdd-transformations.adoc#mapPartitions, mapPartitions>>, <<spark-rdd-transformations.adoc#mapPartitionsWithIndexInternal, mapPartitionsWithIndexInternal>>, <<spark-rdd-transformations.adoc#mapPartitionsInternal, mapPartitionsInternal>>, and <<spark-rdd-transformations.adoc#mapPartitionsWithIndex, mapPartitionsWithIndex>>

* `RDDBarrier[T]` is requested to <<spark-RDDBarrier.adoc#mapPartitions, mapPartitions>> (with the <<isFromBarrier, isFromBarrier>> flag enabled)

By default, it does not preserve partitioning -- the last input parameter `preservesPartitioning` is `false`. If it is `true`, it retains the original RDD's partitioning.

`MapPartitionsRDD` is the result of the following transformations:

* `filter`
* `glom`
* link:spark-rdd-transformations.adoc#mapPartitions[mapPartitions]
* `mapPartitionsWithIndex`
* xref:rdd:PairRDDFunctions.adoc#mapValues[PairRDDFunctions.mapValues]
* xref:rdd:PairRDDFunctions.adoc#flatMapValues[PairRDDFunctions.flatMapValues]

[[isBarrier_]]
When requested for the xref:rdd:RDD.adoc#isBarrier_[isBarrier_] flag, `MapPartitionsRDD` gives the <<isFromBarrier, isFromBarrier>> flag or check whether any of the RDDs of the xref:rdd:RDD.adoc#dependencies[RDD dependencies] are xref:rdd:RDD.adoc#isBarrier[barrier-enabled].

=== [[creating-instance]] Creating MapPartitionsRDD Instance

`MapPartitionsRDD` takes the following to be created:

* [[prev]] Parent xref:rdd:RDD.adoc[RDD] (`RDD[T]`)
* [[f]] Function to execute on partitions
+
```
(TaskContext, partitionID, Iterator[T]) => Iterator[U]
```
* [[preservesPartitioning]] `preservesPartitioning` flag (default: `false`)
* [[isFromBarrier]] `isFromBarrier` flag for <<spark-barrier-execution-mode.adoc#, Barrier Execution Mode>> (default: `false`)
* [[isOrderSensitive]] `isOrderSensitive` flag (default: `false`)
