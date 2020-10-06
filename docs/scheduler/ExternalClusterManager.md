= ExternalClusterManager

ExternalClusterManager is a <<contract, contract for pluggable cluster managers>>. It returns a xref:scheduler:TaskScheduler.adoc[task scheduler] and a xref:scheduler:SchedulerBackend.adoc[backend scheduler] that will be used by xref:ROOT:SparkContext.adoc[SparkContext] to schedule tasks.

NOTE: The support for pluggable cluster managers was introduced in https://issues.apache.org/jira/browse/SPARK-13904[SPARK-13904 Add support for pluggable cluster manager].

External cluster managers are link:spark-SparkContext-creating-instance-internals.adoc#getClusterManager[registered using the `java.util.ServiceLoader` mechanism] (with service markers under `META-INF/services` directory). This allows auto-loading implementations of ExternalClusterManager interface.

NOTE: ExternalClusterManager is a `private[spark]` trait in `org.apache.spark.scheduler` package.

NOTE: The two implementations of the <<contract, ExternalClusterManager contract>> in Spark 2.0 are link:yarn/spark-yarn-YarnClusterManager.adoc[YarnClusterManager] and `MesosClusterManager`.

== [[contract]] ExternalClusterManager Contract

=== [[canCreate]] `canCreate` Method

[source, scala]
----
canCreate(masterURL: String): Boolean
----

`canCreate` is a mechanism to match a ExternalClusterManager implementation to a given master URL.

NOTE: `canCreate` is used when link:spark-SparkContext-creating-instance-internals.adoc#getClusterManager[`SparkContext` loads the external cluster manager for a master URL].

=== [[createTaskScheduler]] `createTaskScheduler` Method

[source, scala]
----
createTaskScheduler(sc: SparkContext, masterURL: String): TaskScheduler
----

`createTaskScheduler` creates a xref:scheduler:TaskScheduler.adoc[TaskScheduler] given a xref:ROOT:SparkContext.adoc[SparkContext] and the input `masterURL`.

=== [[createSchedulerBackend]] `createSchedulerBackend` Method

[source, scala]
----
createSchedulerBackend(sc: SparkContext,
  masterURL: String,
  scheduler: TaskScheduler): SchedulerBackend
----

`createSchedulerBackend` creates a xref:scheduler:SchedulerBackend.adoc[SchedulerBackend] given a xref:ROOT:SparkContext.adoc[SparkContext], the input `masterURL`, and xref:scheduler:TaskScheduler.adoc[TaskScheduler].

=== [[initialize]] Initializing Scheduling Components -- `initialize` Method

[source, scala]
----
initialize(scheduler: TaskScheduler, backend: SchedulerBackend): Unit
----

`initialize` is called after the xref:scheduler:TaskScheduler.adoc[task scheduler] and the xref:scheduler:SchedulerBackend.adoc[backend scheduler] were created and initialized separately.

NOTE: There is a cyclic dependency between a task scheduler and a backend scheduler that begs for this additional initialization step.

NOTE: xref:scheduler:TaskScheduler.adoc[TaskScheduler] and xref:scheduler:SchedulerBackend.adoc[SchedulerBackend] (with xref:scheduler:DAGScheduler.adoc[DAGScheduler]) are commonly referred to as *scheduling components*.
