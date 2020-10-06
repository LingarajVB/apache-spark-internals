= Demo: DiskBlockManager and Block Data

The demo shows how Spark stores data blocks on local disk (using storage:DiskBlockManager.md[DiskBlockManager] and storage:DiskStore.md[DiskStore] among the services).

== Configure Local Directories

Spark uses `spark.local.dir` configuration property for one or more local directories to store data blocks.

Start `spark-shell` with the property set to a directory of your choice (say `local-dirs`). Use one directory for easier monitoring.

```
$SPARK_HOME/bin/spark-shell --conf spark.local.dir=local-dirs
```

When started, Spark will create a proper directory layout. You are interested in `blockmgr-[uuid]` directory.

== "Create" Data Blocks

Execute the following Spark application that forces persisting (_caching_) data to disk.

```
import org.apache.spark.storage.StorageLevel
spark.range(2).persist(StorageLevel.DISK_ONLY).count
```

== Observe Block Files

Go to the `blockmgr-[uuid]` directory and observe the block files. There should be a few. Do you know how many and why?

```
$ tree local-dirs/blockmgr-b7167b5a-ae8d-404b-8de2-1a0fb101fe00/
local-dirs/blockmgr-b7167b5a-ae8d-404b-8de2-1a0fb101fe00/
├── 00
├── 04
│   └── shuffle_0_8_0.data
├── 06
├── 08
│   └── shuffle_0_8_0.index
...
├── 37
│   └── shuffle_0_7_0.index
├── 38
│   └── shuffle_0_4_0.data
├── 39
│   └── shuffle_0_9_0.index
└── 3a
    └── shuffle_0_6_0.data

47 directories, 48 files
```

== Use web UI

Open http://localhost:4040 and switch to Storage tab (at http://localhost:4040/storage/). You should see one RDD cached.

.Storage tab in web UI
image::demo-DiskBlockManager-and-Block-Data-webui-storage.png[align="center"]

Click the link in RDD Name column and review the information.

== Enable Logging

Enable ALL logging level for storage:DiskStore.md#logging[org.apache.spark.storage.DiskStore] and storage:DiskBlockManager.md#logging[org.apache.spark.storage.DiskBlockManager] loggers to have an even deeper insight on the block storage internals.

```
log4j.logger.org.apache.spark.storage.DiskBlockManager=ALL
log4j.logger.org.apache.spark.storage.DiskStore=ALL
```
