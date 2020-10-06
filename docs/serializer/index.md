# Serialization System

**Serialization System** is a core component of Apache Spark with pluggable serializers for RDD and shuffle data.

Serialization System uses xref:SerializerManager.adoc[] to select the xref:serializer:Serializer.adoc[Serializer] to use based on xref:ROOT:configuration-properties.adoc#spark.serializer[spark.serializer] configuration property.
