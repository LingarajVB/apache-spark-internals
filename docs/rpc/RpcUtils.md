= RpcUtils

RpcUtils is an utility for...FIXME

== [[makeDriverRef]] makeDriverRef Method

[source,scala]
----
makeDriverRef(
  name: String,
  conf: SparkConf,
  rpcEnv: RpcEnv): RpcEndpointRef
----

makeDriverRef...FIXME

makeDriverRef is used when:

* xref:scheduler:spark-BarrierTaskContext.adoc#barrierCoordinator[BarrierTaskContext] is created

* SparkEnv utility is used to xref:core:SparkEnv.adoc#create[create a SparkEnv] (on executors)

* xref:executor:Executor.adoc#heartbeatReceiverRef[Executor] is created
