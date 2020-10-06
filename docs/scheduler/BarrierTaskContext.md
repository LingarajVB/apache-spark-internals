== [[BarrierTaskContext]] BarrierTaskContext -- TaskContext for Barrier Tasks

BarrierTaskContext is a concrete <<spark-TaskContext.adoc#, TaskContext>> that is <<creating-instance, created>> exclusively when `Task` is requested to xref:scheduler:Task.adoc#run[run] and the task is xref:scheduler:Task.adoc#isBarrier[isBarrier] (when `Executor` is requested to xref:executor:Executor.adoc#launchTask[launch a task (on "Executor task launch worker" thread pool) sometime in the future]).

[[creating-instance]]
[[taskContext]]
BarrierTaskContext takes a single <<spark-TaskContext.adoc#, TaskContext>> to be created.

=== [[barrierCoordinator]] RpcEndpointRef

BarrierTaskContext creates an RpcEndpointRef for...FIXME
