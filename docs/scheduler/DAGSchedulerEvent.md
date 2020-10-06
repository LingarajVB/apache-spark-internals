= [[DAGSchedulerEvent]] DAGScheduler Events

== [[AllJobsCancelled]] AllJobsCancelled

AllJobsCancelled event carries no extra information.

Posted when DAGScheduler is requested to xref:scheduler:DAGScheduler.adoc#cancelAllJobs[cancelAllJobs]

Event handler: xref:scheduler:DAGScheduler.adoc#doCancelAllJobs[doCancelAllJobs]

== [[BeginEvent]] BeginEvent

BeginEvent event carries the following:

* xref:scheduler:Task.adoc[Task]
* xref:scheduler:spark-scheduler-TaskInfo.adoc[TaskInfo]

Posted when DAGScheduler is requested to xref:scheduler:DAGScheduler.adoc#taskStarted[taskStarted]

Event handler: xref:scheduler:DAGScheduler.adoc#handleBeginEvent[handleBeginEvent]

== [[CompletionEvent]] CompletionEvent

CompletionEvent event carries the following:

* xref:scheduler:Task.adoc[Task]
* Reason
* Result (value computed)
* Accumulator updates
* xref:scheduler:spark-scheduler-TaskInfo.adoc[TaskInfo]

Posted when DAGScheduler is requested to xref:scheduler:DAGScheduler.adoc#taskEnded[taskEnded]

Event handler: xref:scheduler:DAGScheduler.adoc#handleTaskCompletion[handleTaskCompletion]

== [[ExecutorAdded]] ExecutorAdded

ExecutorAdded event carries the following:

* Executor ID
* Host name

Posted when DAGScheduler is requested to xref:scheduler:DAGScheduler.adoc#executorAdded[executorAdded]

Event handler: xref:scheduler:DAGScheduler.adoc#handleExecutorAdded[handleExecutorAdded]

== [[ExecutorLost]] ExecutorLost

ExecutorLost event carries the following:

* Executor ID
* Reason

Posted when DAGScheduler is requested to xref:scheduler:DAGScheduler.adoc#executorLost[executorLost]

Event handler: xref:scheduler:DAGScheduler.adoc#handleExecutorLost[handleExecutorLost]

== [[GettingResultEvent]] GettingResultEvent

GettingResultEvent event carries the following:

* xref:scheduler:spark-scheduler-TaskInfo.adoc[TaskInfo]

Posted when DAGScheduler is requested to xref:scheduler:DAGScheduler.adoc#taskGettingResult[taskGettingResult]

Event handler: xref:scheduler:DAGScheduler.adoc#handleGetTaskResult[handleGetTaskResult]

== [[JobCancelled]] JobCancelled

JobCancelled event carries the following:

* Job ID
* Reason (optional)

Posted when DAGScheduler is requested to xref:scheduler:DAGScheduler.adoc#cancelJob[cancelJob]

Event handler: xref:scheduler:DAGScheduler.adoc#handleJobCancellation[handleJobCancellation]

== [[JobGroupCancelled]] JobGroupCancelled

JobGroupCancelled event carries the following:

* Group ID

Posted when DAGScheduler is requested to xref:scheduler:DAGScheduler.adoc#cancelJobGroup[cancelJobGroup]

Event handler: xref:scheduler:DAGScheduler.adoc#handleJobGroupCancelled[handleJobGroupCancelled]

== [[JobSubmitted]] JobSubmitted

JobSubmitted event carries the following:

* Job ID
* xref:rdd:RDD.adoc[RDD]
* Partition function (`(TaskContext, Iterator[_]) => _`)
* Partitions to compute
* CallSite
* xref:scheduler:spark-scheduler-JobListener.adoc[JobListener] to keep updated about the status of the stage execution
* Execution properties

Posted when DAGScheduler is requested to xref:scheduler:DAGScheduler.adoc#submitJob[submit a job], xref:scheduler:DAGScheduler.adoc#runApproximateJob[run an approximate job] and xref:scheduler:DAGScheduler.adoc#handleJobSubmitted[handleJobSubmitted]

Event handler: xref:scheduler:DAGScheduler.adoc#handleJobSubmitted[handleJobSubmitted]

== [[MapStageSubmitted]] MapStageSubmitted

MapStageSubmitted event carries the following:

* Job ID
* xref:rdd:ShuffleDependency.adoc[ShuffleDependency]
* CallSite
* xref:scheduler:spark-scheduler-JobListener.adoc[JobListener]
* Execution properties

Posted when DAGScheduler is requested to xref:scheduler:DAGScheduler.adoc#submitMapStage[submitMapStage]

Event handler: xref:scheduler:DAGScheduler.adoc#handleMapStageSubmitted[handleMapStageSubmitted]

== [[ResubmitFailedStages]] ResubmitFailedStages

ResubmitFailedStages event carries no extra information.

Posted when DAGScheduler is requested to xref:scheduler:DAGScheduler.adoc#handleTaskCompletion[handleTaskCompletion]

Event handler: xref:scheduler:DAGScheduler.adoc#resubmitFailedStages[resubmitFailedStages]

== [[SpeculativeTaskSubmitted]] SpeculativeTaskSubmitted

SpeculativeTaskSubmitted event carries the following:

* xref:scheduler:Task.adoc[Task]

Posted when DAGScheduler is requested to xref:scheduler:DAGScheduler.adoc#speculativeTaskSubmitted[speculativeTaskSubmitted]

Event handler: xref:scheduler:DAGScheduler.adoc#handleSpeculativeTaskSubmitted[handleSpeculativeTaskSubmitted]

== [[StageCancelled]] StageCancelled

StageCancelled event carries the following:

* Stage ID
* Reason (optional)

Posted when DAGScheduler is requested to xref:scheduler:DAGScheduler.adoc#cancelStage[cancelStage]

Event handler: xref:scheduler:DAGScheduler.adoc#handleStageCancellation[handleStageCancellation]

== [[TaskSetFailed]] TaskSetFailed

TaskSetFailed event carries the following:

* xref:scheduler:TaskSet.adoc[TaskSet]
* Reason
* Exception (optional)

Posted when DAGScheduler is requested to xref:scheduler:DAGScheduler.adoc#taskSetFailed[taskSetFailed]

Event handler: xref:scheduler:DAGScheduler.adoc#handleTaskSetFailed[handleTaskSetFailed]

== [[WorkerRemoved]] WorkerRemoved

WorkerRemoved event carries the following:

* Worked ID
* Host name
* Reason

Posted when DAGScheduler is requested to xref:scheduler:DAGScheduler.adoc#workerRemoved[workerRemoved]

Event handler: xref:scheduler:DAGScheduler.adoc#handleWorkerRemoved[handleWorkerRemoved]
