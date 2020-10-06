= [[ShuffleMapStage]] ShuffleMapStage

*ShuffleMapStage* (_shuffle map stage_ or simply _map stage_) is one of the two types of xref:scheduler:Stage.adoc[stages] in a physical execution DAG (beside a xref:scheduler:ResultStage.adoc[ResultStage]).

NOTE: The *logical DAG* or *logical execution plan* is the xref:rdd:spark-rdd-lineage.adoc[RDD lineage].

ShuffleMapStage corresponds to (and is associated with) a <<shuffleDep, ShuffleDependency>>.

ShuffleMapStage is created when DAGScheduler is requested to xref:scheduler:DAGScheduler.adoc#createShuffleMapStage[plan a ShuffleDependency for execution].

ShuffleMapStage can also be xref:scheduler:DAGScheduler.adoc#submitMapStage[submitted independently as a Spark job] for xref:scheduler:DAGScheduler.adoc#adaptive-query-planning[Adaptive Query Planning / Adaptive Scheduling].

ShuffleMapStage is an input for the other following stages in the DAG of stages and is also called a *shuffle dependency's map side*.

== [[creating-instance]] Creating Instance

ShuffleMapStage takes the following to be created:

* [[id]] Stage ID
* [[rdd]] xref:rdd:ShuffleDependency.adoc#rdd[RDD] of the <<shuffleDep, ShuffleDependency>>
* [[numTasks]] Number of tasks
* [[parents]] Parent xref:scheduler:Stage.adoc[stages]
* [[firstJobId]] ID of the xref:scheduler:spark-scheduler-ActiveJob.adoc[ActiveJob] that created it
* [[callSite]] CallSite
* [[shuffleDep]] xref:rdd:ShuffleDependency.adoc[ShuffleDependency]
* [[mapOutputTrackerMaster]] xref:scheduler:MapOutputTrackerMaster.adoc[MapOutputTrackerMaster]

ShuffleMapStage initializes the <<internal-registries, internal registries and counters>>.

== [[_mapStageJobs]][[mapStageJobs]][[addActiveJob]][[removeActiveJob]] Jobs Registry

ShuffleMapStage keeps track of xref:scheduler:spark-scheduler-ActiveJob.adoc[jobs] that were submitted to execute it independently (if any).

The registry is used when DAGScheduler is requested to xref:scheduler:DAGScheduler.adoc#markMapStageJobsAsFinished[markMapStageJobsAsFinished] (FIXME: when xref:scheduler:DAGSchedulerEventProcessLoop.adoc#handleTaskCompletion[`DAGScheduler` is notified that a `ShuffleMapTask` has finished successfully] and the task made ShuffleMapStage completed and so marks any map-stage jobs waiting on this stage as finished).

A new job is registered (_added_) when DAGScheduler is xref:scheduler:DAGScheduler.adoc#handleMapStageSubmitted[notified that a ShuffleDependency was submitted for execution (as a MapStageSubmitted event)].

An active job is deregistered (_removed_) when DAGScheduler is requested to xref:scheduler:DAGScheduler.adoc#cleanupStateForJobAndIndependentStages[clean up after a job and independent stages].

== [[isAvailable]][[numAvailableOutputs]] ShuffleMapStage is Available (Fully Computed)

When executed, a ShuffleMapStage saves *map output files* (for reduce tasks).

When all <<numPartitions, partitions>> have shuffle map outputs available, ShuffleMapStage is considered *available* (_done_ or _ready_).

ShuffleMapStage is asked about its availability when DAGScheduler is requested for xref:scheduler:DAGScheduler.adoc#getMissingParentStages[missing parent map stages for a stage], xref:scheduler:DAGScheduler.adoc#handleMapStageSubmitted[handleMapStageSubmitted], xref:scheduler:DAGScheduler.adoc#submitMissingTasks[submitMissingTasks], xref:scheduler:DAGScheduler.adoc#handleTaskCompletion[handleTaskCompletion], xref:scheduler:DAGScheduler.adoc#markMapStageJobsAsFinished[markMapStageJobsAsFinished], xref:scheduler:DAGScheduler.adoc#stageDependsOn[stageDependsOn].

ShuffleMapStage uses the <<mapOutputTrackerMaster, MapOutputTrackerMaster>> for the xref:scheduler:MapOutputTrackerMaster.adoc#getNumAvailableOutputs[number of partitions with shuffle map outputs available] (of the <<shuffleDep, ShuffleDependency>> by the shuffle ID).

== [[findMissingPartitions]] Finding Missing Partitions

[source, scala]
----
findMissingPartitions(): Seq[Int]
----

findMissingPartitions requests the <<mapOutputTrackerMaster, MapOutputTrackerMaster>> for the xref:scheduler:MapOutputTrackerMaster.adoc#findMissingPartitions[missing partitions] (of the <<shuffleDep, ShuffleDependency>> by the shuffle ID) and returns them.

If MapOutputTrackerMaster does not track the ShuffleDependency yet, findMissingPartitions simply returns all the xref:scheduler:Stage.adoc#numPartitions[partitions] as missing.

findMissingPartitions is part of the xref:scheduler:Stage.adoc#findMissingPartitions[Stage] abstraction.

== [[stage-sharing]] ShuffleMapStage Sharing

A ShuffleMapStage can be shared across multiple jobs, if these jobs reuse the same RDDs.

.Skipped Stages are already-computed ShuffleMapStages
image::dagscheduler-webui-skipped-stages.png[align="center"]

[source, scala]
----
val rdd = sc.parallelize(0 to 5).map((_,1)).sortByKey()  // <1>
rdd.count  // <2>
rdd.count  // <3>
----
<1> Shuffle at `sortByKey()`
<2> Submits a job with two stages with two being executed
<3> Intentionally repeat the last action that submits a new job with two stages with one being shared as already-being-computed
