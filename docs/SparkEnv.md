= SparkEnv

*SparkEnv* is the Spark Execution Environment with the <<services, core services>> of Apache Spark (that interact with each other to establish a distributed computing platform for a Spark application).

SparkEnv are two separate execution environments for the <<createDriverEnv, driver>> and <<createExecutorEnv, executors>>.

== [[services]] Core Services

[cols="40m,60",options="header",width="100%"]
|===
| Property
| Service

| [[blockManager]] blockManager
| xref:storage:BlockManager.adoc[BlockManager]

| [[broadcastManager]] broadcastManager
| xref:core:BroadcastManager.adoc[]

| [[closureSerializer]] closureSerializer
| xref:serializer:Serializer.adoc[Serializer]

| [[conf]] conf
| xref:ROOT:SparkConf.adoc[SparkConf]

| [[mapOutputTracker]] mapOutputTracker
| xref:scheduler:MapOutputTracker.adoc[MapOutputTracker]

| [[memoryManager]] memoryManager
| xref:memory:MemoryManager.adoc[MemoryManager]

| [[metricsSystem]] metricsSystem
| xref:metrics:spark-metrics-MetricsSystem.adoc[MetricsSystem]

| [[outputCommitCoordinator]] outputCommitCoordinator
| xref:scheduler:OutputCommitCoordinator.adoc[OutputCommitCoordinator]

| [[rpcEnv]] rpcEnv
| xref:rpc:RpcEnv.adoc[]

| [[securityManager]] securityManager
| SecurityManager

| [[serializer]] serializer
| xref:serializer:Serializer.adoc[Serializer]

| [[serializerManager]] serializerManager
| xref:serializer:SerializerManager.adoc[SerializerManager]

| [[shuffleManager]] shuffleManager
| xref:shuffle:ShuffleManager.adoc[ShuffleManager]

|===

== [[creating-instance]] Creating Instance

SparkEnv takes the following to be created:

* [[executorId]] String
* <<rpcEnv, RpcEnv>>
* <<serializer, Serializer>>
* <<closureSerializer, Serializer>>
* <<serializerManager, SerializerManager>>
* <<mapOutputTracker, MapOutputTracker>>
* <<shuffleManager, ShuffleManager>>
* <<broadcastManager, BroadcastManager>>
* <<blockManager, BlockManager>>
* <<securityManager, SecurityManager>>
* <<metricsSystem, MetricsSystem>>
* <<memoryManager, MemoryManager>>
* <<outputCommitCoordinator, OutputCommitCoordinator>>
* <<conf, SparkConf>>

SparkEnv is created when:

* xref:ROOT:spark-SparkContext-creating-instance-internals.adoc[SparkContext] is created (for the <<createDriverEnv, driver>>)

* CoarseGrainedExecutorBackend is requested to xref:executor:CoarseGrainedExecutorBackend.adoc#run[run] (for <<createExecutorEnv, executors>>)

* MesosExecutorBackend is requested to xref:spark-on-mesos:spark-executor-backends-MesosExecutorBackend.adoc#registered[registered] (for <<createExecutorEnv, executors>>)

== [[get]] Accessing SparkEnv

[source, scala]
----
get: SparkEnv
----

get returns the SparkEnv on the driver and executors.

[source, scala]
----
import org.apache.spark.SparkEnv
assert(SparkEnv.get.isInstanceOf[SparkEnv])
----

== [[create]] Creating "Base" SparkEnv (for Driver and Executors)

[source, scala]
----
create(
  conf: SparkConf,
  executorId: String,
  bindAddress: String,
  advertiseAddress: String,
  port: Option[Int],
  isLocal: Boolean,
  numUsableCores: Int,
  ioEncryptionKey: Option[Array[Byte]],
  listenerBus: LiveListenerBus = null,
  mockOutputCommitCoordinator: Option[OutputCommitCoordinator] = None): SparkEnv
----

create is an utility to create the "base" SparkEnv (that is "enhanced" for the driver and executors later on).

.create's Input Arguments and Their Usage
[cols="1,2",options="header",width="100%"]
|===
| Input Argument
| Usage

| `bindAddress`
| Used to create xref:rpc:index.adoc[RpcEnv] and xref:storage:NettyBlockTransferService.adoc#creating-instance[NettyBlockTransferService].

| `advertiseAddress`
| Used to create xref:rpc:index.adoc[RpcEnv] and xref:storage:NettyBlockTransferService.adoc#creating-instance[NettyBlockTransferService].

| `numUsableCores`
| Used to create xref:memory:MemoryManager.adoc[MemoryManager],
 xref:storage:NettyBlockTransferService.adoc#creating-instance[NettyBlockTransferService] and xref:storage:BlockManager.adoc#creating-instance[BlockManager].
|===

[[create-Serializer]]
create creates a `Serializer` (based on <<spark_serializer, spark.serializer>> setting). You should see the following `DEBUG` message in the logs:

```
DEBUG SparkEnv: Using serializer: [serializer]
```

[[create-closure-Serializer]]
create creates a closure `Serializer` (based on <<spark_closure_serializer, spark.closure.serializer>>).

[[ShuffleManager]][[create-ShuffleManager]]
create creates a xref:shuffle:ShuffleManager.adoc[ShuffleManager] given the value of xref:ROOT:configuration-properties.adoc#spark.shuffle.manager[spark.shuffle.manager] configuration property.

[[MemoryManager]][[create-MemoryManager]]
create creates a xref:memory:MemoryManager.adoc[MemoryManager] based on xref:ROOT:configuration-properties.adoc#spark.memory.useLegacyMode[spark.memory.useLegacyMode] setting (with xref:memory:UnifiedMemoryManager.adoc[UnifiedMemoryManager] being the default and `numCores` the input `numUsableCores`).

[[NettyBlockTransferService]][[create-NettyBlockTransferService]]
create creates a xref:storage:NettyBlockTransferService.adoc#creating-instance[NettyBlockTransferService] with the following ports:

* link:spark-driver.adoc#spark_driver_blockManager_port[spark.driver.blockManager.port] for the driver (default: `0`)

* xref:storage:BlockManager.adoc#spark_blockManager_port[spark.blockManager.port] for an executor (default: `0`)

NOTE: create uses the `NettyBlockTransferService` to <<create-BlockManager, create a BlockManager>>.

CAUTION: FIXME A picture with SparkEnv, `NettyBlockTransferService` and the ports "armed".

[[BlockManagerMaster]][[create-BlockManagerMaster]]
create creates a xref:storage:BlockManagerMaster.adoc#creating-instance[BlockManagerMaster] object with the `BlockManagerMaster` RPC endpoint reference (by <<registerOrLookupEndpoint, registering or looking it up by name>> and xref:storage:BlockManagerMasterEndpoint.adoc[]), the input xref:ROOT:SparkConf.adoc[SparkConf], and the input `isDriver` flag.

.Creating BlockManager for the Driver
image::sparkenv-driver-blockmanager.png[align="center"]

NOTE: create registers the *BlockManagerMaster* RPC endpoint for the driver and looks it up for executors.

.Creating BlockManager for Executor
image::sparkenv-executor-blockmanager.png[align="center"]

[[BlockManager]][[create-BlockManager]]
create creates a xref:storage:BlockManager.adoc#creating-instance[BlockManager] (using the above <<BlockManagerMaster, BlockManagerMaster>>, <<create-NettyBlockTransferService, NettyBlockTransferService>> and other services).

create creates a xref:core:BroadcastManager.adoc[].

[[MapOutputTracker]][[create-MapOutputTracker]]
create creates a xref:scheduler:MapOutputTrackerMaster.adoc[MapOutputTrackerMaster] or xref:scheduler:MapOutputTrackerWorker.adoc[MapOutputTrackerWorker] for the driver and executors, respectively.

NOTE: The choice of the real implementation of xref:scheduler:MapOutputTracker.adoc[MapOutputTracker] is based on whether the input `executorId` is *driver* or not.

[[MapOutputTrackerMasterEndpoint]][[create-MapOutputTrackerMasterEndpoint]]
create <<registerOrLookupEndpoint, registers or looks up `RpcEndpoint`>> as *MapOutputTracker*. It registers xref:scheduler:MapOutputTrackerMasterEndpoint.adoc[MapOutputTrackerMasterEndpoint] on the driver and creates a RPC endpoint reference on executors. The RPC endpoint reference gets assigned as the xref:scheduler:MapOutputTracker.adoc#trackerEndpoint[MapOutputTracker RPC endpoint].

CAUTION: FIXME

[[create-CacheManager]]
It creates a CacheManager.

[[create-MetricsSystem]]
It creates a MetricsSystem for a driver and a worker separately.

It initializes `userFiles` temporary directory used for downloading dependencies for a driver while this is the executor's current working directory for an executor.

[[create-OutputCommitCoordinator]]
An OutputCommitCoordinator is created.

create is used when SparkEnv is requested for the SparkEnv for the <<createDriverEnv, driver>> and <<createExecutorEnv, executors>>.

== [[registerOrLookupEndpoint]] Registering or Looking up RPC Endpoint by Name

[source, scala]
----
registerOrLookupEndpoint(
  name: String,
  endpointCreator: => RpcEndpoint)
----

`registerOrLookupEndpoint` registers or looks up a RPC endpoint by `name`.

If called from the driver, you should see the following INFO message in the logs:

```
Registering [name]
```

And the RPC endpoint is registered in the RPC environment.

Otherwise, it obtains a RPC endpoint reference by `name`.

== [[createDriverEnv]] Creating SparkEnv for Driver

[source, scala]
----
createDriverEnv(
  conf: SparkConf,
  isLocal: Boolean,
  listenerBus: LiveListenerBus,
  numCores: Int,
  mockOutputCommitCoordinator: Option[OutputCommitCoordinator] = None): SparkEnv
----

`createDriverEnv` creates a SparkEnv execution environment for the driver.

.Spark Environment for driver
image::sparkenv-driver.png[align="center"]

`createDriverEnv` accepts an instance of xref:ROOT:SparkConf.adoc[SparkConf], link:spark-deployment-environments.adoc[whether it runs in local mode or not], xref:scheduler:LiveListenerBus.adoc[], the number of cores to use for execution in local mode or `0` otherwise, and a xref:scheduler:OutputCommitCoordinator.adoc[OutputCommitCoordinator] (default: none).

`createDriverEnv` ensures that link:spark-driver.adoc#spark_driver_host[spark.driver.host] and link:spark-driver.adoc#spark_driver_port[spark.driver.port] settings are defined.

It then passes the call straight on to the <<create, create helper method>> (with `driver` executor id, `isDriver` enabled, and the input parameters).

NOTE: `createDriverEnv` is exclusively used by link:spark-SparkContext-creating-instance-internals.adoc#createSparkEnv[SparkContext to create a SparkEnv] (while a xref:ROOT:SparkContext.adoc#creating-instance[SparkContext is being created for the driver]).

== [[createExecutorEnv]] Creating SparkEnv for Executor

[source, scala]
----
createExecutorEnv(
  conf: SparkConf,
  executorId: String,
  hostname: String,
  numCores: Int,
  ioEncryptionKey: Option[Array[Byte]],
  isLocal: Boolean): SparkEnv
----

`createExecutorEnv` creates an *executor's (execution) environment* that is the Spark execution environment for an executor.

.Spark Environment for executor
image::sparkenv-executor.png[align="center"]

NOTE: `createExecutorEnv` is a `private[spark]` method.

`createExecutorEnv` simply <<create, creates the base SparkEnv>> (passing in all the input parameters) and <<set, sets it as the current SparkEnv>>.

NOTE: The number of cores `numCores` is configured using `--cores` command-line option of `CoarseGrainedExecutorBackend` and is specific to a cluster manager.

NOTE: `createExecutorEnv` is used when xref:executor:CoarseGrainedExecutorBackend.adoc#run[`CoarseGrainedExecutorBackend` runs] and link:spark-executor-backends-MesosExecutorBackend.adoc#registered[`MesosExecutorBackend` registers a Spark executor].

== [[stop]] Stopping SparkEnv

[source, scala]
----
stop(): Unit
----

stop checks <<isStopped, isStopped>> internal flag and does nothing when enabled already.

Otherwise, stop turns `isStopped` flag on, stops all `pythonWorkers` and requests the following services to stop:

1. xref:scheduler:MapOutputTracker.adoc#stop[MapOutputTracker]
2. xref:shuffle:ShuffleManager.adoc#stop[ShuffleManager]
3. xref:core:BroadcastManager.adoc#stop[BroadcastManager]
4. xref:storage:BlockManager.adoc#stop[BlockManager]
5. xref:storage:BlockManagerMaster.adoc#stop[BlockManagerMaster]
6. link:spark-metrics-MetricsSystem.adoc#stop[MetricsSystem]
7. xref:scheduler:OutputCommitCoordinator.adoc#stop[OutputCommitCoordinator]

stop xref:rpc:index.adoc#shutdown[requests `RpcEnv` to shut down] and xref:rpc:index.adoc#awaitTermination[waits till it terminates].

Only on the driver, stop deletes the <<driverTmpDir, temporary directory>>. You can see the following WARN message in the logs if the deletion fails.

```
Exception while deleting Spark temp dir: [path]
```

NOTE: stop is used when xref:ROOT:SparkContext.adoc#stop[`SparkContext` stops] (on the driver) and xref:executor:Executor.adoc#stop[`Executor` stops].

== [[set]] `set` Method

[source, scala]
----
set(e: SparkEnv): Unit
----

`set` saves the input SparkEnv to <<env, env>> internal registry (as the default SparkEnv).

NOTE: `set` is used when...FIXME

== [[environmentDetails]] environmentDetails Utility

[source, scala]
----
environmentDetails(
  conf: SparkConf,
  schedulingMode: String,
  addedJars: Seq[String],
  addedFiles: Seq[String]): Map[String, Seq[(String, String)]]
----

environmentDetails...FIXME

environmentDetails is used when SparkContext is requested to xref:ROOT:SparkContext.adoc#postEnvironmentUpdate[post a SparkListenerEnvironmentUpdate event].

== [[logging]] Logging

Enable `ALL` logging level for `org.apache.spark.SparkEnv` logger to see what happens inside.

Add the following line to `conf/log4j.properties`:

[source]
----
log4j.logger.org.apache.spark.SparkEnv=ALL
----

Refer to xref:ROOT:spark-logging.adoc[Logging].

== [[internal-properties]] Internal Properties

[cols="30m,70",options="header",width="100%"]
|===
| Name
| Description

| isStopped
| [[isStopped]] Used to mark SparkEnv stopped

Default: `false`

| driverTmpDir
| [[driverTmpDir]]

|===
