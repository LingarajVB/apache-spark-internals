== [[SchedulingMode]] Scheduling Mode -- `spark.scheduler.mode` Spark Property

*Scheduling Mode* (aka _order task policy_ or _scheduling policy_ or _scheduling order_) defines a policy to sort tasks in order for execution.

The scheduling mode `schedulingMode` attribute is part of the xref:scheduler:TaskScheduler.adoc#schedulingMode[TaskScheduler Contract].

The only implementation of the `TaskScheduler` contract in Spark -- xref:scheduler:TaskSchedulerImpl.adoc[TaskSchedulerImpl] -- uses xref:ROOT:configuration-properties.adoc#spark.scheduler.mode[spark.scheduler.mode] setting to configure `schedulingMode` that is _merely_ used to set up the xref:scheduler:TaskScheduler.adoc#rootPool[rootPool] attribute (with `FIFO` being the default). It happens when xref:scheduler:TaskSchedulerImpl.adoc#initialize[`TaskSchedulerImpl` is initialized].

There are three acceptable scheduling modes:

* [[FIFO]] `FIFO` with no pools but a single top-level unnamed pool with elements being xref:scheduler:TaskSetManager.adoc[TaskSetManager] objects; lower priority gets xref:scheduler:spark-scheduler-Schedulable.adoc[Schedulable] sooner or earlier stage wins.
* [[FAIR]] `FAIR` with a xref:scheduler:spark-scheduler-FairSchedulableBuilder.adoc#buildPools[hierarchy of `Schedulable` (sub)pools] with the xref:scheduler:TaskScheduler.adoc#rootPool[rootPool] at the top.
* [[NONE]] *NONE* (not used)

NOTE: Out of three possible `SchedulingMode` policies only `FIFO` and `FAIR` modes are supported by xref:scheduler:TaskSchedulerImpl.adoc[TaskSchedulerImpl].

[NOTE]
====
After the root pool is initialized, the scheduling mode is no longer relevant (since the link:spark-scheduler-Schedulable.adoc[Schedulable] that represents the root pool is fully set up).

The root pool is later used when xref:scheduler:TaskSchedulerImpl.adoc#submitTasks[`TaskSchedulerImpl` submits tasks (as `TaskSets`) for execution].
====

NOTE: The xref:scheduler:TaskScheduler.adoc#rootPool[root pool] is a `Schedulable`. Refer to link:spark-scheduler-Schedulable.adoc[Schedulable].

=== [[fair-scheduling-sparkui]] Monitoring FAIR Scheduling Mode using Spark UI

CAUTION: FIXME Describe me...
