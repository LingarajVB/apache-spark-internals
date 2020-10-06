= BroadcastFactory

*BroadcastFactory* is an <<contract, abstraction>> for <<implementations, factories>> that xref:core:BroadcastManager.adoc[BroadcastManager] uses for xref:ROOT:Broadcast.adoc[].

NOTE: As of https://issues.apache.org/jira/browse/SPARK-12588[Spark 2.0], it is no longer possible to plug a custom BroadcastFactory in, and xref:core:TorrentBroadcastFactory.adoc[TorrentBroadcastFactory] is the one and only known implementation.

== [[contract]] Contract

=== [[initialize]] initialize Method

[source,scala]
----
initialize(
  isDriver: Boolean,
  conf: SparkConf,
  securityMgr: SecurityManager): Unit
----

Used when BroadcastManager is xref:BroadcastManager.adoc#creating-instance[created].

=== [[newBroadcast]] newBroadcast Method

[source,scala]
----
newBroadcast[T: ClassTag](
  value: T,
  isLocal: Boolean,
  id: Long): Broadcast[T]
----

Used when BroadcastManager is requested for a xref:BroadcastManager.adoc#newBroadcast[new broadcast variable].

=== [[stop]] stop Method

[source,scala]
----
stop(): Unit
----

Used when BroadcastManager is requested to xref:BroadcastManager.adoc#stop[stop].

=== [[unbroadcast]] unbroadcast Method

[source,scala]
----
unbroadcast(
  id: Long,
  removeFromDriver: Boolean,
  blocking: Boolean): Unit
----

Used when BroadcastManager is requested to xref:BroadcastManager.adoc#unbroadcast[unbroadcast a broadcast variable].

== [[implementations]] Available BroadcastFactories

xref:core:TorrentBroadcastFactory.adoc[TorrentBroadcastFactory] is the default and only known BroadcastFactory in Apache Spark.
