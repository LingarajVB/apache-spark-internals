= HadoopWriteConfigUtil

`HadoopWriteConfigUtil[K, V]` is an <<contract, abstraction>> of <<implementations, writer configurers>>.

`HadoopWriteConfigUtil` is used for <<spark-internal-io-SparkHadoopWriter.adoc#, SparkHadoopWriter>> utility when requested to <<spark-internal-io-SparkHadoopWriter.adoc#write, write an RDD of key-value pairs>> (for xref:rdd:PairRDDFunctions.adoc#saveAsNewAPIHadoopDataset[saveAsNewAPIHadoopDataset] and xref:rdd:PairRDDFunctions.adoc#saveAsHadoopDataset[saveAsHadoopDataset] transformations).

[[contract]]
.HadoopWriteConfigUtil Contract
[cols="30m,70",options="header",width="100%"]
|===
| Method
| Description

| assertConf
a| [[assertConf]]

[source, scala]
----
assertConf(
  jobContext: JobContext,
  conf: SparkConf): Unit
----

| closeWriter
a| [[closeWriter]]

[source, scala]
----
closeWriter(
  taskContext: TaskAttemptContext): Unit
----

| createCommitter
a| [[createCommitter]]

[source, scala]
----
createCommitter(
  jobId: Int): HadoopMapReduceCommitProtocol
----

| createJobContext
a| [[createJobContext]]

[source, scala]
----
createJobContext(
  jobTrackerId: String,
  jobId: Int): JobContext
----

| createTaskAttemptContext
a| [[createTaskAttemptContext]]

[source, scala]
----
createTaskAttemptContext(
  jobTrackerId: String,
  jobId: Int,
  splitId: Int,
  taskAttemptId: Int): TaskAttemptContext
----

Creates a Hadoop https://hadoop.apache.org/docs/r2.7.3/api/org/apache/hadoop/mapreduce/TaskAttemptContext.html[TaskAttemptContext]

| initOutputFormat
a| [[initOutputFormat]]

[source, scala]
----
initOutputFormat(
  jobContext: JobContext): Unit
----

| initWriter
a| [[initWriter]]

[source, scala]
----
initWriter(
  taskContext: TaskAttemptContext,
  splitId: Int): Unit
----

| write
a| [[write]]

[source, scala]
----
write(
  pair: (K, V)): Unit
----

Writes out the key-value pair

Used when `SparkHadoopWriter` is requested to <<spark-internal-io-SparkHadoopWriter.adoc#executeTask, executeTask>> (while <<spark-internal-io-SparkHadoopWriter.adoc#write, writing out key-value pairs of a partition>>)

|===

[[implementations]]
.HadoopWriteConfigUtils
[cols="30,70",options="header",width="100%"]
|===
| HadoopWriteConfigUtil
| Description

| <<spark-internal-io-HadoopMapReduceWriteConfigUtil.adoc#, HadoopMapReduceWriteConfigUtil>>
| [[HadoopMapReduceWriteConfigUtil]]

| <<spark-internal-io-HadoopMapRedWriteConfigUtil.adoc#, HadoopMapRedWriteConfigUtil>>
| [[HadoopMapRedWriteConfigUtil]]

|===
