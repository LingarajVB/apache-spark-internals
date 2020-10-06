= [[ShuffleWriter]] ShuffleWriter

*ShuffleWriter* of K keys and V values (`ShuffleWriter[K, V]`) is an abstraction of <<implementations, shuffle writers>> that can <<write, write key-value records>> (of a RDD partition) to a shuffle system.

ShuffleWriter is used when xref:scheduler:ShuffleMapTask.adoc[ShuffleMapTask] is requested to xref:scheduler:ShuffleMapTask.adoc#runTask[run].

== [[implementations]] ShuffleWriters

.ShuffleWriters
[cols="40m,60",options="header",width="100%"]
|===
| ShuffleWriter
| Description

| xref:shuffle:BypassMergeSortShuffleWriter.adoc[BypassMergeSortShuffleWriter]
| [[BypassMergeSortShuffleWriter]] ShuffleWriter for a xref:shuffle:BypassMergeSortShuffleHandle.adoc[BypassMergeSortShuffleHandle]

| xref:shuffle:SortShuffleWriter.adoc[SortShuffleWriter]
| [[SortShuffleWriter]] Fallback ShuffleWriter (when neither <<BypassMergeSortShuffleWriter, BypassMergeSortShuffleWriter>> nor <<UnsafeShuffleWriter, UnsafeShuffleWriter>> could be used)

| xref:shuffle:UnsafeShuffleWriter.adoc[UnsafeShuffleWriter]
| [[UnsafeShuffleWriter]] ShuffleWriter for xref:shuffle:SerializedShuffleHandle.adoc[SerializedShuffleHandles]

|===

== [[stop]] Stopping ShuffleWriter

[source, scala]
----
stop(
  success: Boolean): Option[MapStatus]
----

Stops (_closes_) the ShuffleWriter and returns a xref:scheduler:MapStatus.adoc[MapStatus] if the writing completed successfully. The `success` flag is the status of the task execution.

stop is used when ShuffleMapTask is requested to xref:scheduler:ShuffleMapTask.adoc#runTask[run].

== [[write]] Writing Partition Records Out to Shuffle System

[source, scala]
----
write(
  records: Iterator[Product2[K, V]]): Unit
----

Writes key-value records out to a shuffle system.

write is used when `ShuffleMapTask` is requested to xref:scheduler:ShuffleMapTask.adoc#runTask[run].
