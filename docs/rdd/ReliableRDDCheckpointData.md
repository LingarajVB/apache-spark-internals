= ReliableRDDCheckpointData

*ReliableRDDCheckpointData* is a xref:rdd:RDDCheckpointData.adoc[RDDCheckpointData] for xref:ROOT:rdd-checkpointing.adoc#reliable-checkpointing[Reliable Checkpointing].

== [[creating-instance]] Creating Instance

ReliableRDDCheckpointData takes the following to be created:

* [[rdd]] xref:rdd:RDD.adoc[++RDD[T]++]

ReliableRDDCheckpointData is created for xref:rdd:RDD.adoc#checkpoint[RDD.checkpoint] operator.

== [[cpDir]][[checkpointPath]] Checkpoint Directory

ReliableRDDCheckpointData creates a subdirectory of the xref:ROOT:SparkContext.adoc#checkpointDir[application-wide checkpoint directory] for <<doCheckpoint, checkpointing>> the given <<rdd, RDD>>.

The name of the subdirectory uses the xref:rdd:RDD.adoc#id[unique identifier] of the <<rdd, RDD>>:

[source,plaintext]
----
rdd-[id]
----

== [[doCheckpoint]] Checkpointing RDD

[source, scala]
----
doCheckpoint(): CheckpointRDD[T]
----

doCheckpoint xref:rdd:ReliableCheckpointRDD.adoc#writeRDDToCheckpointDirectory[writes] the <<rdd, RDD>> to the <<cpDir, checkpoint directory>> (that creates a new RDD).

With xref:ROOT:configuration-properties.adoc#spark.cleaner.referenceTracking.cleanCheckpoints[spark.cleaner.referenceTracking.cleanCheckpoints] configuration property enabled, doCheckpoint requests the xref:ROOT:SparkContext.adoc#cleaner[ContextCleaner] to xref:core:ContextCleaner.adoc#registerRDDCheckpointDataForCleanup[registerRDDCheckpointDataForCleanup] for the new RDD.

In the end, doCheckpoint prints out the following INFO message to the logs and returns the new RDD.

[source,plaintext]
----
Done checkpointing RDD [id] to [cpDir], new parent is RDD [id]
----

doCheckpoint is part of the xref:rdd:RDDCheckpointData.adoc#doCheckpoint[RDDCheckpointData] abstraction.
