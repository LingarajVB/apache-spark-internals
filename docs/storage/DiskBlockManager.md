= [[DiskBlockManager]] DiskBlockManager

*DiskBlockManager* creates and maintains the logical mapping between logical blocks and physical on-disk locations for a storage:BlockManager.md#diskBlockManager[BlockManager].

.DiskBlockManager and BlockManager
image::DiskBlockManager-BlockManager.png[align="center"]

By default, one block is mapped to one file with a name given by its `BlockId`. It is however possible to have a block map to only a segment of a file.

Block files are hashed among the <<getConfiguredLocalDirs, local directories>>.

DiskBlockManager is used to create a DiskStore.md[DiskStore].

TIP: Consult demo-diskblockmanager-and-block-data.md[Demo: DiskBlockManager and Block Data].

== [[creating-instance]] Creating Instance

DiskBlockManager takes the following to be created:

* [[conf]] ROOT:SparkConf.md[SparkConf]
* [[deleteFilesOnStop]] `deleteFilesOnStop` flag

When created, DiskBlockManager <<localDirs, creates one or many local directories to store block data>> and initializes the internal <<subDirs, subDirs>> collection of locks for every local directory.

In the end, DiskBlockManager <<addShutdownHook, registers a shutdown hook>> to clean up the local directories for blocks.

== [[localDirs]] Local Directories for Blocks

[source,scala]
----
localDirs: Array[File]
----

While being created, DiskBlockManager <<createLocalDirs, creates local directories>> for block data. DiskBlockManager expects at least one local directory or prints out the following ERROR message to the logs and exits the JVM (with exit code 53).

```
Failed to create any local dir.
```

localDirs is used when:

* DiskBlockManager is requested to <<getFile, getFile>>, initialize the <<subDirs, subDirs>> internal registry, and to <<doStop, doStop>>

* BlockManager is requested to storage:BlockManager.md#registerWithExternalShuffleServer[register with an external shuffle server]

== [[subDirsPerLocalDir]][[subDirs]] File Locks for Local Block Store Directories

[source, scala]
----
subDirs: Array[Array[File]]
----

`subDirs` is a lookup table for file locks of every <<localDirs, local block directory>> (with the first dimension for local directories and the second for locks).

The number of block subdirectories is controlled by ROOT:configuration-properties.md#spark.diskStore.subDirectories[spark.diskStore.subDirectories] configuration property (default: `64`).

`subDirs(dirId)(subDirId)` is used to access `subDirId` subdirectory in `dirId` local directory.

subDirs is used when DiskBlockManager is requested for a <<getFile, block file>> or <<getAllFiles, all block files>>.

== [[createLocalDirs]] Creating Local Directories for Block Data

[source, scala]
----
createLocalDirs(
  conf: SparkConf): Array[File]
----

createLocalDirs creates `blockmgr-[random UUID]` directory under local directories to store block data.

Internally, createLocalDirs <<getConfiguredLocalDirs, finds the configured local directories where Spark can write files>> and creates a subdirectory `blockmgr-[UUID]` under every configured parent directory.

For every local directory, createLocalDirs prints out the following INFO message to the logs:

```
Created local directory at [localDir]
```

In case of an exception, createLocalDirs prints out the following ERROR message to the logs and skips the directory.

```
Failed to create local dir in [rootDir]. Ignoring this directory.
```

createLocalDirs is used when the <<localDirs, localDirs>> internal registry is initialized.

== [[getFile]] Finding Block File (and Creating Parent Directories)

[source, scala]
----
getFile(
  blockId: BlockId): File // <1>
getFile(
  filename: String): File
----
<1> Uses the name of the given `BlockId`

getFile computes a hash of the file name of the input storage:BlockId.md[] that is used for the name of the parent directory and subdirectory.

getFile creates the subdirectory unless it already exists.

getFile is used when:

* DiskBlockManager is requested to <<containsBlock, containsBlock>>, <<createTempLocalBlock, createTempLocalBlock>>, <<createTempShuffleBlock, createTempShuffleBlock>>

* DiskStore is requested to DiskStore.md#getBytes[getBytes], DiskStore.md#remove[remove], DiskStore.md#contains[contains], and DiskStore.md#put[put]

* IndexShuffleBlockResolver is requested to shuffle:IndexShuffleBlockResolver.md#getDataFile[getDataFile] and shuffle:IndexShuffleBlockResolver.md#getIndexFile[getIndexFile]

== [[createTempShuffleBlock]] createTempShuffleBlock Method

[source, scala]
----
createTempShuffleBlock(): (TempShuffleBlockId, File)
----

`createTempShuffleBlock` creates a temporary `TempShuffleBlockId` block.

CAUTION: FIXME

== [[getAllFiles]] All Block Files

[source, scala]
----
getAllFiles(): Seq[File]
----

`getAllFiles`...FIXME

NOTE: `getAllFiles` is used exclusively when DiskBlockManager is requested to <<getAllBlocks, getAllBlocks>>.

== [[addShutdownHook]] Registering Shutdown Hook -- `addShutdownHook` Internal Method

[source, scala]
----
addShutdownHook(): AnyRef
----

`addShutdownHook` registers a shutdown hook to execute <<doStop, doStop>> at shutdown.

When executed, you should see the following DEBUG message in the logs:

```
DEBUG DiskBlockManager: Adding shutdown hook
```

`addShutdownHook` adds the shutdown hook so it prints the following INFO message and executes <<doStop, doStop>>.

```
INFO DiskBlockManager: Shutdown hook called
```

== [[doStop]] Stopping DiskBlockManager (Removing Local Directories for Blocks) -- `doStop` Internal Method

[source, scala]
----
doStop(): Unit
----

`doStop` deletes the local directories recursively (only when the constructor's `deleteFilesOnStop` is enabled and the parent directories are not registered to be removed at shutdown).

NOTE: `doStop` is used when DiskBlockManager is requested to <<addShutdownHook, shut down>> or <<stop, stop>>.

== [[getConfiguredLocalDirs]] Getting Local Directories for Spark to Write Files -- `Utils.getConfiguredLocalDirs` Internal Method

[source, scala]
----
getConfiguredLocalDirs(conf: SparkConf): Array[String]
----

`getConfiguredLocalDirs` returns the local directories where Spark can write files.

Internally, `getConfiguredLocalDirs` uses `conf` ROOT:SparkConf.md[SparkConf] to know if deploy:ExternalShuffleService.md[External Shuffle Service] is enabled (based on ROOT:configuration-properties.md#spark.shuffle.service.enabled[spark.shuffle.service.enabled] configuration property).

`getConfiguredLocalDirs` checks if <<isRunningInYarnContainer, Spark runs on YARN>> and if so, returns <<getYarnLocalDirs, ``LOCAL_DIRS``-controlled local directories>>.

In non-YARN mode (or for the driver in yarn-client mode), `getConfiguredLocalDirs` checks the following environment variables (in the order) and returns the value of the first met:

1. `SPARK_EXECUTOR_DIRS` environment variable
2. `SPARK_LOCAL_DIRS` environment variable
3. `MESOS_DIRECTORY` environment variable (only when External Shuffle Service is not used)

In the end, when no earlier environment variables were found, `getConfiguredLocalDirs` uses spark-properties.md#spark.local.dir[spark.local.dir] Spark property or falls back on `java.io.tmpdir` System property.

[NOTE]
====
`getConfiguredLocalDirs` is used when:

* DiskBlockManager is requested to <<createLocalDirs, createLocalDirs>>

* `Utils` helper is requested to spark-Utils.md#getLocalDir[getLocalDir] and spark-Utils.md#getOrCreateLocalRootDirsImpl[getOrCreateLocalRootDirsImpl]
====

== [[getYarnLocalDirs]] Getting Writable Directories in YARN -- `getYarnLocalDirs` Internal Method

[source, scala]
----
getYarnLocalDirs(conf: SparkConf): String
----

`getYarnLocalDirs` uses `conf` ROOT:SparkConf.md[SparkConf] to read `LOCAL_DIRS` environment variable with comma-separated local directories (that have already been created and secured so that only the user has access to them).

`getYarnLocalDirs` throws an `Exception` with the message `Yarn Local dirs can't be empty` if `LOCAL_DIRS` environment variable was not set.

== [[isRunningInYarnContainer]] Checking If Spark Runs on YARN -- `isRunningInYarnContainer` Internal Method

[source, scala]
----
isRunningInYarnContainer(conf: SparkConf): Boolean
----

`isRunningInYarnContainer` uses `conf` ROOT:SparkConf.md[SparkConf] to read Hadoop YARN's http://hadoop.apache.org/docs/current/hadoop-yarn/hadoop-yarn-api/apidocs/org/apache/hadoop/yarn/api/ApplicationConstants.Environment.html#CONTAINER_ID[`CONTAINER_ID` environment variable] to find out if Spark runs in a YARN container.

NOTE: `CONTAINER_ID` environment variable is exported by YARN NodeManager.

== [[getAllBlocks]] Getting All Blocks (From Files Stored On Disk)

[source, scala]
----
getAllBlocks(): Seq[BlockId]
----

getAllBlocks gets all the blocks stored on disk.

Internally, getAllBlocks takes the <<getAllFiles, block files>> and returns their names (as `BlockId`).

getAllBlocks is used when BlockManager is requested to storage:BlockManager.md#getMatchingBlockIds[find IDs of existing blocks for a given filter].

== [[stop]] `stop` Internal Method

[source, scala]
----
stop(): Unit
----

`stop`...FIXME

NOTE: `stop` is used exclusively when `BlockManager` is requested to storage:BlockManager.md#stop[stop].

== [[logging]] Logging

Enable `ALL` logging level for `org.apache.spark.storage.DiskBlockManager` logger to see what happens inside.

Add the following line to `conf/log4j.properties`:

[source]
----
log4j.logger.org.apache.spark.storage.DiskBlockManager=ALL
----

Refer to ROOT:spark-logging.md[Logging].
