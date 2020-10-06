= TorrentBroadcastFactory

*TorrentBroadcastFactory* is a xref:core:BroadcastFactory.adoc[BroadcastFactory] of xref:core:TorrentBroadcast.adoc[TorrentBroadcast]s (for BitTorrent-like xref:ROOT:Broadcast.adoc[]s).

NOTE: As of https://issues.apache.org/jira/browse/SPARK-12588[Spark 2.0] TorrentBroadcastFactory is is the one and only known xref:core:BroadcastFactory.adoc[BroadcastFactory].

== [[creating-instance]] Creating Instance

TorrentBroadcastFactory takes no arguments to be created.

TorrentBroadcastFactory is created for xref:BroadcastManager.adoc#broadcastFactory[BroadcastManager].

== [[newBroadcast]] Creating Broadcast Variable (TorrentBroadcast)

[source,scala]
----
newBroadcast[T: ClassTag](
  value_ : T,
  isLocal: Boolean,
  id: Long): Broadcast[T]
----

newBroadcast creates a xref:core:TorrentBroadcast.adoc[] (for the given `value_` and `id` and ignoring the `isLocal` flag).

newBroadcast is part of the xref:BroadcastFactory.adoc#newBroadcast[BroadcastFactory] abstraction.

== [[unbroadcast]] Unbroadcasting Broadcast Variable

[source,scala]
----
unbroadcast(
  id: Long,
  removeFromDriver: Boolean,
  blocking: Boolean): Unit
----

unbroadcast xref:core:TorrentBroadcast.adoc#unpersist[removes all persisted state associated with the TorrentBroadcast] (by the given id).

unbroadcast is part of the xref:BroadcastFactory.adoc#unbroadcast[BroadcastFactory] abstraction.

== [[initialize]] Initializing TorrentBroadcastFactory

[source,scala]
----
initialize(
  isDriver: Boolean,
  conf: SparkConf,
  securityMgr: SecurityManager): Unit
----

initialize does nothing.

initialize is part of the xref:BroadcastFactory.adoc#initialize[BroadcastFactory] abstraction.

== [[stop]] Stopping TorrentBroadcastFactory

[source,scala]
----
stop(): Unit
----

stop does nothing.

stop is part of the xref:BroadcastFactory.adoc#stop[BroadcastFactory] abstraction.
