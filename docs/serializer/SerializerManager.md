= SerializerManager

*SerializerManager* is used to <<getSerializer, select a serializer>> for shuffle blocks (either the default <<defaultSerializer, JavaSerializer>> or <<kryoSerializer, KryoSerializer>> based on the key and value of a record).

== [[creating-instance]] Creating Instance

SerializerManager takes the following to be created:

* [[defaultSerializer]] xref:serializer:Serializer.adoc[Serializer]
* [[conf]] xref:ROOT:SparkConf.adoc[SparkConf]
* [[encryptionKey]] Optional encryption key (`[Array[Byte]]`)

SerializerManager is created when SparkEnv utility is used to xref:core:SparkEnv.adoc#create[create a SparkEnv for the driver and executors].

== [[SparkEnv]] Accessing SerializerManager Using SparkEnv

SerializerManager is available using xref:core:SparkEnv.adoc#serializerManager[SparkEnv] on the driver and executors.

[source, scala]
----
import org.apache.spark.SparkEnv
SparkEnv.get.serializerManager
----

== [[kryoSerializer]] KryoSerializer

SerializerManager creates a KryoSerializer when created.

KryoSerializer is used as a <<getSerializer, serializer>> when the type of a given key and value is <<canUseKryo, compatible with Kryo>>.

== [[wrapForCompression]] Wrapping Input or Output Stream of Block for Compression

[source, scala]
----
wrapForCompression(
  blockId: BlockId,
  s: OutputStream): OutputStream
wrapForCompression(
  blockId: BlockId,
  s: InputStream): InputStream
----

wrapForCompression...FIXME

wrapForCompression is used when:

* SerializerManager is requested to <<wrapStream, wrapStream>>, <<dataSerializeStream, dataSerializeStream>>, <<dataDeserializeStream, dataDeserializeStream>> and <<dataSerializeWithExplicitClassTag, dataSerializeWithExplicitClassTag>>

* SerializedValuesHolder (of xref:storage:MemoryStore.adoc[MemoryStore]) is requested for a SerializationStream

== [[wrapStream]] Wrapping Input or Output Stream for Block

[source, scala]
----
wrapStream(
  blockId: BlockId,
  s: InputStream): InputStream
wrapStream(
  blockId: BlockId,
  s: OutputStream): OutputStream
----

wrapStream...FIXME

wrapStream is used when:

* BlockStoreShuffleReader is requested to xref:shuffle:BlockStoreShuffleReader.adoc#read[read combined records for a reduce task]

* DiskMapIterator (of xref:shuffle:ExternalAppendOnlyMap.adoc[ExternalAppendOnlyMap]) is requested for nextBatchStream

* SpillReader (of xref:shuffle:ExternalSorter.adoc[ExternalSorter]) is requested for nextBatchStream

* xref:memory:UnsafeSorterSpillReader.adoc[UnsafeSorterSpillReader] is created

* DiskBlockObjectWriter is requested to xref:storage:DiskBlockObjectWriter.adoc#open[open]

== [[dataSerializeStream]] dataSerializeStream Method

[source, scala]
----
dataSerializeStream[T: ClassTag](
  blockId: BlockId,
  outputStream: OutputStream,
  values: Iterator[T]): Unit
----

dataSerializeStream...FIXME

dataSerializeStream is used when BlockManager is requested to xref:storage:BlockManager.adoc#doPutIterator[doPutIterator] and xref:storage:BlockManager.adoc#dropFromMemory[dropFromMemory].

== [[dataSerializeWithExplicitClassTag]] dataSerializeWithExplicitClassTag Method

[source, scala]
----
dataSerializeWithExplicitClassTag(
  blockId: BlockId,
  values: Iterator[_],
  classTag: ClassTag[_]): ChunkedByteBuffer
----

dataSerializeWithExplicitClassTag...FIXME

dataSerializeWithExplicitClassTag is used when BlockManager is requested to xref:storage:BlockManager.adoc#doGetLocalBytes[doGetLocalBytes].

== [[dataDeserializeStream]] dataDeserializeStream Method

[source, scala]
----
dataDeserializeStream[T](
  blockId: BlockId,
  inputStream: InputStream)
  (classTag: ClassTag[T]): Iterator[T]
----

dataDeserializeStream...FIXME

dataDeserializeStream is used when:

* BlockManager is requested to xref:storage:BlockManager.adoc#getLocalValues[getLocalValues], xref:storage:BlockManager.adoc#getRemoteValues[getRemoteValues] and xref:storage:BlockManager.adoc#doPutBytes[doPutBytes]

* MemoryStore is requested to xref:storage:MemoryStore.adoc#putIteratorAsBytes[putIteratorAsBytes] (when PartiallySerializedBlock is requested for a PartiallyUnrolledIterator)

== [[getSerializer]] Selecting Serializer

[source, scala]
----
getSerializer(
  ct: ClassTag[_],
  autoPick: Boolean): Serializer
getSerializer(
  keyClassTag: ClassTag[_],
  valueClassTag: ClassTag[_]): Serializer
----

getSerializer returns the <<kryoSerializer, KryoSerializer>> when the given arguments are <<canUseKryo, compatible with Kryo>>. Otherwise, getSerializer returns the <<defaultSerializer, Serializer>>.

getSerializer is used when:

* ShuffledRDD is requested for xref:rdd:ShuffledRDD.adoc#getDependencies[dependencies].

* SerializerManager is requested to <<dataSerializeStream, dataSerializeStream>>, <<dataSerializeWithExplicitClassTag, dataSerializeWithExplicitClassTag>> and <<dataDeserializeStream, dataDeserializeStream>>

* SerializedValuesHolder (of xref:storage:MemoryStore.adoc[MemoryStore]) is requested for a SerializationStream

== [[canUseKryo]] Checking Whether Kryo Serializer Could Be Used

[source, scala]
----
canUseKryo(
  ct: ClassTag[_]): Boolean
----

canUseKryo is `true` when the given ClassTag is a primitive, an array of primitives or a String. Otherwise, canUseKryo is `false`.

canUseKryo is used when SerializerManager is requested for a <<getSerializer, Serializer>>.

== [[shouldCompress]] shouldCompress Method

[source, scala]
----
shouldCompress(
  blockId: BlockId): Boolean
----

shouldCompress...FIXME

shouldCompress is used when SerializerManager is requested to <<wrapForCompression, wrapForCompression>>.
