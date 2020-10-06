= RpcEnv

RpcEnv is an <<contract, abstraction>> of <<implementations, RPC systems>>.

== [[implementations]] Available RpcEnvs

xref:rpc:NettyRpcEnv.adoc[] is the default and only known RpcEnv in Apache Spark.

== [[contract]] Contract

=== [[address]] address Method

[source,scala]
----
address: RpcAddress
----

xref:rpc:RpcAddress.adoc[] of the RPC system

=== [[asyncSetupEndpointRefByURI]] asyncSetupEndpointRefByURI Method

[source,scala]
----
asyncSetupEndpointRefByURI(
  uri: String): Future[RpcEndpointRef]
----

Sets up an RPC endpoing by URI (asynchronously) and returns xref:rpc:RpcEndpointRef.adoc[]

Used when:

* WorkerWatcher is created

* CoarseGrainedExecutorBackend is requested to xref:executor:CoarseGrainedExecutorBackend.adoc#onStart[onStart]

* RpcEnv is requested to <<setupEndpointRefByURI, setupEndpointRefByURI>>

=== [[awaitTermination]] awaitTermination Method

[source,scala]
----
awaitTermination(): Unit
----

Waits till the RPC system terminates

Used when:

* SparkEnv is requested to xref:core:SparkEnv.adoc#stop[stop]

* ClientApp is requested to start

* LocalSparkCluster is requested to stop

* (Spark Standalone) Master and Worker are launched

* CoarseGrainedExecutorBackend standalone application is launched

=== [[deserialize]] deserialize Method

[source,scala]
----
deserialize[T](
  deserializationAction: () => T): T
----

Used when:

* PersistenceEngine is requested to readPersistedData

* NettyRpcEnv is requested to xref:rpc:NettyRpcEnv.adoc#deserialize[deserialize]

=== [[endpointRef]] endpointRef Method

[source,scala]
----
endpointRef(
  endpoint: RpcEndpoint): RpcEndpointRef
----

Used when RpcEndpoint is requested for the xref:rpc:RpcEndpoint.adoc#self[RpcEndpointRef to itself]

=== [[fileServer]] RpcEnvFileServer

[source,scala]
----
fileServer: RpcEnvFileServer
----

xref:rpc:RpcEnvFileServer.adoc[] of the RPC system

Used when xref:ROOT:SparkContext.adoc[] is created (and registers the REPL's output directory) and requested to xref:ROOT:SparkContext.adoc#addFile[addFile] or xref:ROOT:SparkContext.adoc#addJar[addJar]

=== [[openChannel]] openChannel Method

[source,scala]
----
openChannel(
  uri: String): ReadableByteChannel
----

Opens a channel to download a file from the given URI

Used when:

* Utils utility is used to doFetchFile

* ExecutorClassLoader is requested to getClassFileInputStreamFromSparkRPC

=== [[setupEndpoint]] setupEndpoint Method

[source,scala]
----
setupEndpoint(
  name: String,
  endpoint: RpcEndpoint): RpcEndpointRef
----

Used when:

* xref:ROOT:SparkContext.adoc[] is created (and registers the xref:ROOT:spark-SparkContext-creating-instance-internals.adoc#_heartbeatReceiver[HeartbeatReceiver])

* SparkEnv utility is used to xref:core:SparkEnv.adoc#create[create a SparkEnv] (and register the BlockManagerMaster, MapOutputTracker and OutputCommitCoordinator RPC endpoints on the driver)

* ClientApp is requested to start (and register the client RPC endpoint)

* StandaloneAppClient is requested to start (and register the AppClient RPC endpoint)

* (Spark Standalone) Master is requested to startRpcEnvAndEndpoint (and register the Master RPC endpoint)

* (Spark Standalone) Worker is requested to startRpcEnvAndEndpoint (and register the Worker RPC endpoint)

* DriverWrapper standalone application is launched (and registers the workerWatcher RPC endpoint)

* CoarseGrainedExecutorBackend standalone application is launched (and registers the Executor and WorkerWatcher RPC endpoints)

* TaskSchedulerImpl is requested to xref:scheduler:TaskSchedulerImpl.adoc#maybeInitBarrierCoordinator[maybeInitBarrierCoordinator]

* CoarseGrainedSchedulerBackend is requested to xref:scheduler:CoarseGrainedSchedulerBackend.adoc#createDriverEndpointRef[createDriverEndpointRef] (and registers the CoarseGrainedScheduler RPC endpoint)

* LocalSchedulerBackend is requested to xref:spark-local:spark-LocalSchedulerBackend.adoc#start[start] (and registers the LocalSchedulerBackendEndpoint RPC endpoint)

* xref:storage:BlockManager.adoc#slaveEndpoint[BlockManager] is created (and registers the BlockManagerEndpoint RPC endpoint)

* (Spark on YARN) ApplicationMaster is requested to xref:spark-on-yarn:spark-yarn-applicationmaster.adoc#createAllocator[createAllocator] (and registers the YarnAM RPC endpoint)

* (Spark on YARN) xref:spark-on-yarn:spark-yarn-yarnschedulerbackend.adoc#yarnSchedulerEndpointRef[YarnSchedulerBackend] is created (and registers the YarnScheduler RPC endpoint)

=== [[setupEndpointRef]] setupEndpointRef Method

[source,scala]
----
setupEndpointRef(
  address: RpcAddress,
  endpointName: String): RpcEndpointRef
----

setupEndpointRef creates an RpcEndpointAddress (for the given xref:rpc:RpcAddress.adoc[] and endpoint name) and <<setupEndpointRefByURI, setupEndpointRefByURI>>.

setupEndpointRef is used when:

* ClientApp is requested to start

* ClientEndpoint is requested to tryRegisterAllMasters

* Worker is requested to tryRegisterAllMasters and reregisterWithMaster

* RpcUtils utility is used to xref:rpc:RpcUtils.adoc#makeDriverRef[makeDriverRef]

* (Spark on YARN) ApplicationMaster is requested to xref:spark-on-yarn:spark-yarn-applicationmaster.adoc#runDriver[runDriver] and xref:spark-on-yarn:spark-yarn-applicationmaster.adoc#runExecutorLauncher[runExecutorLauncher]

=== [[setupEndpointRefByURI]] setupEndpointRefByURI Method

[source,scala]
----
setupEndpointRefByURI(
  uri: String): RpcEndpointRef
----

setupEndpointRefByURI <<asyncSetupEndpointRefByURI, asyncSetupEndpointRefByURI>> by the given URI and waits for the result or <<defaultLookupTimeout, defaultLookupTimeout>>.

setupEndpointRefByURI is used when:

* CoarseGrainedExecutorBackend standalone application is xref:executor:CoarseGrainedExecutorBackend.adoc#run[launched]

* RpcEnv is requested to <<setupEndpointRef, setupEndpointRef>>

=== [[shutdown]] shutdown Method

[source,scala]
----
shutdown(): Unit
----

Shuts down the RPC system

Used when:

* SparkEnv is requested to xref:core:SparkEnv.adoc#stop[stop]

* LocalSparkCluster is requested to xref:spark-standalone:spark-standalone-LocalSparkCluster.adoc#stop[stop]

* DriverWrapper is launched

* CoarseGrainedExecutorBackend is xref:executor:CoarseGrainedExecutorBackend.adoc#run[launched]

* NettyRpcEnvFactory is requested to xref:rpc:NettyRpcEnvFactory.adoc#create[create an RpcEnv] (in server mode and failed to assign a port)

=== [[stop]] Stopping RpcEndpointRef

[source,scala]
----
stop(
  endpoint: RpcEndpointRef): Unit
----

Used when:

* SparkContext is requested to xref:ROOT:SparkContext.adoc#stop[stop]

* RpcEndpoint is requested to xref:rpc:RpcEndpoint.adoc#stop[stop]

* BlockManager is requested to xref:storage:BlockManager.adoc#stop[stop]

== [[defaultLookupTimeout]] Default Endpoint Lookup Timeout

RpcEnv uses the default lookup timeout for...FIXME

When a remote endpoint is resolved, a local RPC environment connects to the remote one. It is called *endpoint lookup*. To configure the time needed for the endpoint lookup you can use the following settings.

It is a prioritized list of *lookup timeout* properties (the higher on the list, the more important):

* xref:ROOT:configuration-properties.adoc#spark.rpc.lookupTimeout[spark.rpc.lookupTimeout]
* <<spark.network.timeout, spark.network.timeout>>

Their value can be a number alone (seconds) or any number with time suffix, e.g. `50s`, `100ms`, or `250us`. See <<settings, Settings>>.

== [[creating-instance]] Creating Instance

RpcEnv takes the following to be created:

* [[conf]] xref:ROOT:SparkConf.adoc[]

RpcEnv is created using <<create, RpcEnv.create>> utility.

RpcEnv is an abstract class and cannot be created directly. It is created indirectly for the <<implementations, concrete RpcEnvs>>.

== [[create]] Creating RpcEnv

[source,scala]
----
create(
  name: String,
  host: String,
  port: Int,
  conf: SparkConf,
  securityManager: SecurityManager,
  clientMode: Boolean = false): RpcEnv // <1>
create(
  name: String,
  bindAddress: String,
  advertiseAddress: String,
  port: Int,
  conf: SparkConf,
  securityManager: SecurityManager,
  numUsableCores: Int,
  clientMode: Boolean): RpcEnv
----
<1> Uses 0 for numUsableCores

create creates a xref:rpc:NettyRpcEnvFactory.adoc[] and requests to xref:rpc:NettyRpcEnvFactory.adoc#create[create an RpcEnv] (with an xref:rpc:RpcEnvConfig.adoc[] with all the given arguments).

create is used when:

* SparkEnv utility is requested to xref:core:SparkEnv.adoc#create[create a SparkEnv] (clientMode flag is turned on for executors and off for the driver)

* With clientMode flag turned on:

** (Spark on YARN) ApplicationMaster is requested to xref:spark-on-yarn:spark-yarn-applicationmaster.adoc#runExecutorLauncher[runExecutorLauncher] (in client deploy mode with clientMode flag is turned on)

** ClientApp is requested to start

** (Spark Standalone) Master is requested to startRpcEnvAndEndpoint

** DriverWrapper standalone application is launched

** (Spark Standalone) Worker is requested to startRpcEnvAndEndpoint

** CoarseGrainedExecutorBackend is requested to xref:executor:CoarseGrainedExecutorBackend.adoc#run[run]
