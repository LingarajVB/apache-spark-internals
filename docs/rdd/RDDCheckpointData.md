= RDDCheckpointData

*RDDCheckpointData* is an abstraction of information related to RDD checkpointing.

== [[implementations]] Available RDDCheckpointDatas

[cols="30,70",options="header",width="100%"]
|===
| RDDCheckpointData
| Description

| xref:rdd:LocalRDDCheckpointData.adoc[LocalRDDCheckpointData]
| [[LocalRDDCheckpointData]]

| xref:rdd:ReliableRDDCheckpointData.adoc[ReliableRDDCheckpointData]
| [[ReliableRDDCheckpointData]] xref:ROOT:rdd-checkpointing.adoc#reliable-checkpointing[Reliable Checkpointing]

|===

== [[creating-instance]] Creating Instance

RDDCheckpointData takes the following to be created:

* [[rdd]] xref:rdd:RDD.adoc[RDD]

== [[Serializable]] RDDCheckpointData as Serializable

RDDCheckpointData is java.io.Serializable.

== [[cpState]] States

* [[Initialized]] Initialized

* [[CheckpointingInProgress]] CheckpointingInProgress

* [[Checkpointed]] Checkpointed

== [[checkpoint]] Checkpointing RDD

[source, scala]
----
checkpoint(): CheckpointRDD[T]
----

checkpoint changes the <<cpState, state>> to <<CheckpointingInProgress, CheckpointingInProgress>> only when in <<Initialized, Initialized>> state. Otherwise, checkpoint does nothing and returns.

checkpoint <<doCheckpoint, doCheckpoint>> that gives an CheckpointRDD (that is the <<cpRDD, cpRDD>> internal registry).

checkpoint changes the <<cpState, state>> to <<Checkpointed, Checkpointed>>.

In the end, checkpoint requests the given <<rdd, RDD>> to xref:rdd:RDD.adoc#markCheckpointed[markCheckpointed].

checkpoint is used when RDD is requested to xref:rdd:RDD.adoc#doCheckpoint[doCheckpoint].

== [[doCheckpoint]] doCheckpoint Method

[source, scala]
----
doCheckpoint(): CheckpointRDD[T]
----

doCheckpoint is used when RDDCheckpointData is requested to <<checkpoint, checkpoint the RDD>>.
