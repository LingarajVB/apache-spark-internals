= SerializationStream

*SerializationStream* is an abstraction of <<implementations, streams>> for writing serialized key-value records.

== [[implementations]] Available SerializationStreams

[cols="30,70",options="header",width="100%"]
|===
| SerializationStream
| Description

| JavaSerializationStream
| [[JavaSerializationStream]]

| KryoSerializationStream
| [[KryoSerializationStream]]

|===

== [[writeAll]] Writing All Records

[source, scala]
----
writeAll[T: ClassTag](
  iter: Iterator[T]): SerializationStream
----

writeAll <<writeObject, writes all records>> of the given iterator.

writeAll is used when:

* ReliableCheckpointRDD utility is requested to writePartitionToCheckpointFile

* SerializerManager is requested to xref:serializer:SerializerManager.adoc#dataSerializeStream[dataSerializeStream] and xref:serializer:SerializerManager.adoc#dataSerializeWithExplicitClassTag[dataSerializeWithExplicitClassTag]

== [[writeObject]] Writing Object

[source, scala]
----
writeObject[T: ClassTag](
  t: T): SerializationStream
----

writeObject is the most general-purpose method to write an object.

writeObject is used when...FIXME

== [[writeKey]] Writing Key (of Key-Value Record)

[source, scala]
----
writeKey[T: ClassTag](
  key: T): SerializationStream
----

writeKey <<writeObject, writes the object>> representing the key of a key-value record.

writeKey is used when:

* UnsafeShuffleWriter is requested to xref:shuffle:UnsafeShuffleWriter.adoc#insertRecordIntoSorter[insert a record into a ShuffleExternalSorter]

* DiskBlockObjectWriter is requested to xref:storage:DiskBlockObjectWriter.adoc#write[write the key and value of a record]

== [[writeValue]] Writing Value (of Key-Value Record)

[source, scala]
----
writeValue[T: ClassTag](
  value: T): SerializationStream
----

writeValue <<writeObject, writes the object>> representing the value of a key-value record.

writeValue is used when:

* UnsafeShuffleWriter is requested to xref:shuffle:UnsafeShuffleWriter.adoc#insertRecordIntoSorter[insert a record into a ShuffleExternalSorter]

* DiskBlockObjectWriter is requested to xref:storage:DiskBlockObjectWriter.adoc#write[write the key and value of a record]

== [[flush]] Flushing Stream

[source, scala]
----
flush(): Unit
----

flush is used when:

* UnsafeShuffleWriter is requested to xref:shuffle:UnsafeShuffleWriter.adoc#insertRecordIntoSorter[insert a record into a ShuffleExternalSorter]

* DiskBlockObjectWriter is requested to xref:storage:DiskBlockObjectWriter.adoc#commitAndGet[commitAndGet]
