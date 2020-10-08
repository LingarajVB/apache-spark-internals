# Dynamic Allocation of Executors

*Dynamic Allocation (of Executors)* (aka _Elastic Scaling_) is a Spark feature that allows for adding or removing executor:Executor.md[Spark executors] dynamically to match the workload.

Unlike the "traditional" static allocation where a Spark application reserves CPU and memory resources upfront (irrespective of how much it may eventually use), in dynamic allocation you get as much as needed and no more. It scales the number of executors up and down based on workload, i.e. idle executors are removed, and when there are pending tasks waiting for executors to be launched on, dynamic allocation requests them.

Dynamic allocation is enabled using <<spark.dynamicAllocation.enabled, spark.dynamicAllocation.enabled>> setting. When enabled, it is assumed that the deploy:ExternalShuffleService.md[External Shuffle Service] is also used (it is not by default as controlled by ROOT:configuration-properties.md#spark.shuffle.service.enabled[spark.shuffle.service.enabled] property).

[ExecutorAllocationManager](ExecutorAllocationManager.md) is responsible for dynamic allocation of executors.

Dynamic allocation reports the current state using spark-service-ExecutorAllocationManagerSource.md[`ExecutorAllocationManager` metric source].

Dynamic Allocation comes with the policy of scaling executors up and down as follows:

1. *Scale Up Policy* requests new executors when there are pending tasks and increases the number of executors exponentially since executors start slow and Spark application may need slightly more.
2. *Scale Down Policy* removes executors that have been idle for <<spark.dynamicAllocation.executorIdleTimeout, spark.dynamicAllocation.executorIdleTimeout>> seconds.

Dynamic allocation is available for all the currently-supported spark-cluster.md[cluster managers], i.e. Spark Standalone, Hadoop YARN and Apache Mesos.

TIP: Read about deploy:ExternalShuffleService.md[Dynamic Allocation on Hadoop YARN].

TIP: Review the excellent slide deck http://www.slideshare.net/databricks/dynamic-allocation-in-spark[Dynamic Allocation in Spark] from Databricks.

== [[isDynamicAllocationEnabled]] Is Dynamic Allocation Enabled? -- `Utils.isDynamicAllocationEnabled` Method

[source, scala]
----
isDynamicAllocationEnabled(conf: SparkConf): Boolean
----

`isDynamicAllocationEnabled` returns `true` if all the following conditions hold:

. <<spark.dynamicAllocation.enabled, spark.dynamicAllocation.enabled>> is enabled (i.e. `true`)
. spark-cluster.md[Spark on cluster] is used (i.e. ROOT:configuration-properties.md#spark.master[spark.master] is non-`local`)
. <<spark.dynamicAllocation.testing, spark.dynamicAllocation.testing>> is enabled (i.e. `true`)

Otherwise, `isDynamicAllocationEnabled` returns `false`.

NOTE: `isDynamicAllocationEnabled` returns `true`, i.e. dynamic allocation is enabled, in local/spark-local.md[Spark local (pseudo-cluster)] for testing only (with <<spark.dynamicAllocation.testing, spark.dynamicAllocation.testing>> enabled).

NOTE: `isDynamicAllocationEnabled` is used when Spark calculates the initial number of executors for scheduler:CoarseGrainedSchedulerBackend.md[coarse-grained scheduler backends] for  yarn/README.md#getInitialTargetExecutorNumber[YARN], spark-standalone-StandaloneSchedulerBackend.md#start[Spark Standalone], and spark-mesos/spark-mesos-MesosCoarseGrainedSchedulerBackend.md#executorLimitOption[Mesos]. It is also used for spark-streaming/spark-streaming-streamingcontext.md#validate[Spark Streaming].

[TIP]
====
Enable `WARN` logging level for `org.apache.spark.util.Utils` logger to see what happens inside.

Add the following line to `conf/log4j.properties`:

```
log4j.logger.org.apache.spark.util.Utils=WARN
```

Refer to spark-logging.md[Logging].
====

== [[programmable-dynamic-allocation]] Programmable Dynamic Allocation

`SparkContext` offers a ROOT:SparkContext.md#dynamic-allocation[developer API to scale executors up or down].

== [[getDynamicAllocationInitialExecutors]] Getting Initial Number of Executors for Dynamic Allocation -- `Utils.getDynamicAllocationInitialExecutors` Method

[source, scala]
----
getDynamicAllocationInitialExecutors(conf: SparkConf): Int
----

`getDynamicAllocationInitialExecutors` first makes sure that <<spark.dynamicAllocation.initialExecutors, spark.dynamicAllocation.initialExecutors>> is equal or greater than <<spark.dynamicAllocation.minExecutors, spark.dynamicAllocation.minExecutors>>.

NOTE: <<spark.dynamicAllocation.initialExecutors, spark.dynamicAllocation.initialExecutors>> falls back to <<spark.dynamicAllocation.minExecutors, spark.dynamicAllocation.minExecutors>> if not set. Why to print the WARN message to the logs?

If not, you should see the following WARN message in the logs:

[options="wrap"]
----
spark.dynamicAllocation.initialExecutors less than spark.dynamicAllocation.minExecutors is invalid, ignoring its setting, please update your configs.
----

`getDynamicAllocationInitialExecutors` makes sure that executor:Executor.md#spark.executor.instances[spark.executor.instances] is greater than <<spark.dynamicAllocation.minExecutors, spark.dynamicAllocation.minExecutors>>.

NOTE: Both executor:Executor.md#spark.executor.instances[spark.executor.instances] and <<spark.dynamicAllocation.minExecutors, spark.dynamicAllocation.minExecutors>> fall back to `0` when no defined explicitly.

If not, you should see the following WARN message in the logs:

[options="wrap"]
----
spark.executor.instances less than spark.dynamicAllocation.minExecutors is invalid, ignoring its setting, please update your configs.
----

`getDynamicAllocationInitialExecutors` sets the initial number of executors to be the maximum of:

* <<spark.dynamicAllocation.minExecutors, spark.dynamicAllocation.minExecutors>>
* <<spark.dynamicAllocation.initialExecutors, spark.dynamicAllocation.initialExecutors>>
* executor:Executor.md#spark.executor.instances[spark.executor.instances]
* `0`

You should see the following INFO message in the logs:

[options="wrap"]
----
Using initial executors = [initialExecutors], max of spark.dynamicAllocation.initialExecutors, spark.dynamicAllocation.minExecutors and spark.executor.instances
----

NOTE: `getDynamicAllocationInitialExecutors` is used when spark-ExecutorAllocationManager.md#initialNumExecutors[`ExecutorAllocationManager` sets the initial number of executors] and yarn/spark-yarn-YarnSparkHadoopUtil.md#getInitialTargetExecutorNumber[in YARN to set initial target number of executors].

== [[settings]] Settings

.Spark Properties
[cols="1,1,2",options="header",width="100%"]
|===
| Spark Property
| Default Value
| Description

| [[spark.dynamicAllocation.enabled]] spark.dynamicAllocation.enabled
| `false`
| Flag to enable (`true`) or disable (`false`) dynamic allocation.

NOTE: executor:Executor.md#spark.executor.instances[spark.executor.instances] setting can be set using spark-submit.md#command-line-options[`--num-executors` command-line option] of spark-submit.md[spark-submit].

| [[spark.dynamicAllocation.initialExecutors]] `spark.dynamicAllocation.initialExecutors`
| <<spark.dynamicAllocation.minExecutors, spark.dynamicAllocation.minExecutors>>
| Initial number of executors for dynamic allocation.

NOTE: <<getDynamicAllocationInitialExecutors, getDynamicAllocationInitialExecutors>> warns when  `spark.dynamicAllocation.initialExecutors` is less than <<spark.dynamicAllocation.minExecutors, spark.dynamicAllocation.minExecutors>>.

| [[spark.dynamicAllocation.minExecutors]] `spark.dynamicAllocation.minExecutors`
| `0`
| Minimum number of executors for dynamic allocation.

spark-ExecutorAllocationManager.md#validateSettings[Must be positive and less than or equal to `spark.dynamicAllocation.maxExecutors`].

| [[spark.dynamicAllocation.maxExecutors]] spark.dynamicAllocation.maxExecutors
| `Integer.MAX_VALUE`
| Maximum number of executors for dynamic allocation.

spark-ExecutorAllocationManager.md#validateSettings[Must be greater than `0` and greater than or equal to `spark.dynamicAllocation.minExecutors`].

| [[spark.dynamicAllocation.schedulerBacklogTimeout]] `spark.dynamicAllocation.schedulerBacklogTimeout`
| `1s`
|

spark-ExecutorAllocationManager.md#validateSettings[Must be greater than `0`].

| [[spark.dynamicAllocation.sustainedSchedulerBacklogTimeout]] `spark.dynamicAllocation.sustainedSchedulerBacklogTimeout`
| <<spark.dynamicAllocation.schedulerBacklogTimeout, spark.dynamicAllocation.schedulerBacklogTimeout>>)
|

spark-ExecutorAllocationManager.md#validateSettings[Must be greater than `0`].

| [[spark.dynamicAllocation.executorIdleTimeout]] `spark.dynamicAllocation.executorIdleTimeout`
| `60s`
| Time for how long an executor can be idle before it gets removed.

spark-ExecutorAllocationManager.md#validateSettings[Must be greater than `0`].

| [[spark.dynamicAllocation.cachedExecutorIdleTimeout]] `spark.dynamicAllocation.cachedExecutorIdleTimeout`
| `Integer.MAX_VALUE`
|

| [[spark.dynamicAllocation.testing]] `spark.dynamicAllocation.testing`
|
|

|===

=== Future

* SPARK-4922
* SPARK-4751
* SPARK-7955
