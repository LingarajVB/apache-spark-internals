== [[LiveRDD]] LiveRDD

`LiveRDD` is a link:spark-core-LiveEntity.adoc[LiveEntity] that...FIXME

`LiveRDD` is <<creating-instance, created>> exclusively when `AppStatusListener` is requested to xref:core:AppStatusListener.adoc#onStageSubmitted[handle onStageSubmitted event]

[[creating-instance]]
[[info]]
`LiveRDD` takes a xref:storage:RDDInfo.adoc[RDDInfo] when created.

=== [[doUpdate]] `doUpdate` Method

[source, scala]
----
doUpdate(): Any
----

NOTE: `doUpdate` is part of link:spark-core-LiveEntity.adoc#doUpdate[LiveEntity Contract] to...FIXME.

`doUpdate`...FIXME
