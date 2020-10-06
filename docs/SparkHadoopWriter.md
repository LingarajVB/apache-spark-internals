= SparkHadoopWriter

`SparkHadoopWriter` utility is used to <<write, write a key-value RDD (as a Hadoop OutputFormat)>>.

`SparkHadoopWriter` utility is used by xref:rdd:PairRDDFunctions.adoc#saveAsNewAPIHadoopDataset[saveAsNewAPIHadoopDataset] and xref:rdd:PairRDDFunctions.adoc#saveAsHadoopDataset[saveAsHadoopDataset] transformations.

[[logging]]
[TIP]
====
Enable `ALL` logging level for `org.apache.spark.internal.io.SparkHadoopWriter` logger to see what happens inside.

Add the following line to `conf/log4j.properties`:

```
log4j.logger.org.apache.spark.internal.io.SparkHadoopWriter=ALL
```

Refer to <<spark-logging.adoc#, Logging>>.
====

== [[write]] Writing Key-Value RDD Out (As Hadoop OutputFormat) -- `write` Utility

[source, scala]
----
write[K, V: ClassTag](
  rdd: RDD[(K, V)],
  config: HadoopWriteConfigUtil[K, V]): Unit
----

[[write-commitJobId]]
`write` uses the id of the given RDD as the `commitJobId`.

[[write-jobTrackerId]]
`write` creates a `jobTrackerId` with the current date.

[[write-jobContext]]
`write` requests the given `HadoopWriteConfigUtil` to <<spark-internal-io-HadoopWriteConfigUtil.adoc#createJobContext, create a Hadoop JobContext>> (for the <<write-jobTrackerId, jobTrackerId>> and <<write-commitJobId, commitJobId>>).

`write` requests the given `HadoopWriteConfigUtil` to <<spark-internal-io-HadoopWriteConfigUtil.adoc#initOutputFormat, initOutputFormat>> with the Hadoop https://hadoop.apache.org/docs/r2.7.3/api/org/apache/hadoop/mapreduce/JobContext.html[JobContext].

`write` requests the given `HadoopWriteConfigUtil` to <<spark-internal-io-HadoopWriteConfigUtil.adoc#assertConf, assertConf>>.

`write` requests the given `HadoopWriteConfigUtil` to <<spark-internal-io-HadoopWriteConfigUtil.adoc#createCommitter, create a HadoopMapReduceCommitProtocol committer>> for the <<write-commitJobId, commitJobId>>.

`write` requests the `HadoopMapReduceCommitProtocol` to <<spark-internal-io-HadoopMapReduceCommitProtocol.adoc#setupJob, setupJob>> (with the <<write-jobContext, jobContext>>).

[[write-runJob]][[write-executeTask]]
`write` uses the `SparkContext` (of the given RDD) to xref:ROOT:SparkContext.adoc#runJob[run a Spark job asynchronously] for the given RDD with the <<executeTask, executeTask>> partition function.

[[write-commitJob]]
In the end, `write` requests the <<write-committer, HadoopMapReduceCommitProtocol>> to <<spark-internal-io-HadoopMapReduceCommitProtocol.adoc#commitJob, commit the job>> and prints out the following INFO message to the logs:

```
Job [getJobID] committed.
```

NOTE: `write` is used when `PairRDDFunctions` is requested to xref:rdd:PairRDDFunctions.adoc#saveAsNewAPIHadoopDataset[saveAsNewAPIHadoopDataset] and xref:rdd:PairRDDFunctions.adoc#saveAsHadoopDataset[saveAsHadoopDataset].

=== [[write-Throwable]] `write` Utility And Throwables

In case of any `Throwable`, `write` prints out the following ERROR message to the logs:

```
Aborting job [getJobID].
```

[[write-abortJob]]
`write` requests the <<write-committer, HadoopMapReduceCommitProtocol>> to <<spark-internal-io-HadoopMapReduceCommitProtocol.adoc#abortJob, abort the job>> and throws a `SparkException`:

```
Job aborted.
```

== [[executeTask]] Writing RDD Partition -- `executeTask` Internal Utility

[source, scala]
----
executeTask[K, V: ClassTag](
  context: TaskContext,
  config: HadoopWriteConfigUtil[K, V],
  jobTrackerId: String,
  commitJobId: Int,
  sparkPartitionId: Int,
  sparkAttemptNumber: Int,
  committer: FileCommitProtocol,
  iterator: Iterator[(K, V)]): TaskCommitMessage
----

`executeTask` requests the given `HadoopWriteConfigUtil` to <<spark-internal-io-HadoopWriteConfigUtil.adoc#createTaskAttemptContext, create a TaskAttemptContext>>.

`executeTask` requests the given `FileCommitProtocol` to <<spark-internal-io-FileCommitProtocol.adoc#setupTask, set up a task>> with the `TaskAttemptContext`.

`executeTask` requests the given `HadoopWriteConfigUtil` to <<spark-internal-io-HadoopWriteConfigUtil.adoc#initWriter, initWriter>> (with the `TaskAttemptContext` and the given `sparkPartitionId`).

`executeTask` <<initHadoopOutputMetrics, initHadoopOutputMetrics>>.

`executeTask` writes all rows of the RDD partition (from the given `Iterator[(K, V)]`). `executeTask` requests the given `HadoopWriteConfigUtil` to <<spark-internal-io-HadoopWriteConfigUtil.adoc#write, write>>. In the end, `executeTask` requests the given `HadoopWriteConfigUtil` to <<spark-internal-io-HadoopWriteConfigUtil.adoc#closeWriter, closeWriter>> and the given `FileCommitProtocol` to <<spark-internal-io-FileCommitProtocol.adoc#commitTask, commit the task>>.

`executeTask` updates metrics about writing data to external systems (*bytesWritten* and *recordsWritten*) every few records and at the end.

In case of any errors, `executeTask` requests the given `HadoopWriteConfigUtil` to <<spark-internal-io-HadoopWriteConfigUtil.adoc#closeWriter, closeWriter>> and the given `FileCommitProtocol` to <<spark-internal-io-FileCommitProtocol.adoc#abortTask, abort the task>>. In the end, `executeTask` prints out the following ERROR message to the logs:

```
Task [taskAttemptID] aborted.
```

NOTE: `executeTask` is used when `SparkHadoopWriter` utility is used to <<write, write>>.

== [[initHadoopOutputMetrics]] `initHadoopOutputMetrics` Utility

[source, scala]
----
initHadoopOutputMetrics(
  context: TaskContext): (OutputMetrics, () => Long)
----

`initHadoopOutputMetrics`...FIXME

NOTE: `initHadoopOutputMetrics` is used when `SparkHadoopWriter` utility is used to <<executeTask, executeTask>>.
