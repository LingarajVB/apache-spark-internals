== [[Source]] Source -- Contract of Metrics Sources

`Source` is a <<contract, contract>> of *metrics sources*.

[[contract]]
[source, scala]
----
package org.apache.spark.metrics.source

trait Source {
  def sourceName: String
  def metricRegistry: MetricRegistry
}
----

NOTE: `Source` is a `private[spark]` contract.

.Source Contract
[cols="1,2",options="header",width="100%"]
|===
| Method
| Description

| `sourceName`
| [[sourceName]] Used when...FIXME

| `metricRegistry`
| [[metricRegistry]] Dropwizard Metrics' https://metrics.dropwizard.io/3.1.0/apidocs/com/codahale/metrics/MetricRegistry.html[MetricRegistry]

Used when...FIXME
|===

[[implementations]]
.Sources
[cols="1,2",options="header",width="100%"]
|===
| Source
| Description

| `ApplicationSource`
| [[ApplicationSource]]

| xref:storage:spark-BlockManager-BlockManagerSource.adoc[BlockManagerSource]
| [[BlockManagerSource]]

| `CacheMetrics`
| [[CacheMetrics]]

| `CodegenMetrics`
| [[CodegenMetrics]]

| xref:metrics:spark-scheduler-DAGSchedulerSource.adoc[DAGSchedulerSource]
| [[DAGSchedulerSource]]

| xref:ROOT:spark-service-ExecutorAllocationManagerSource.adoc[ExecutorAllocationManagerSource]
| [[ExecutorAllocationManagerSource]]

| xref:executor:ExecutorSource.adoc[]
| [[ExecutorSource]]

| `ExternalShuffleServiceSource`
| [[ExternalShuffleServiceSource]]

| `HiveCatalogMetrics`
| [[HiveCatalogMetrics]]

| xref:metrics:JvmSource.adoc[JvmSource]
| [[JvmSource]]

| `LiveListenerBusMetrics`
| [[LiveListenerBusMetrics]]

| `MasterSource`
| [[MasterSource]]

| `MesosClusterSchedulerSource`
| [[MesosClusterSchedulerSource]]

| xref:storage:ShuffleMetricsSource.adoc[]
| [[ShuffleMetricsSource]]

| `StreamingSource`
| [[StreamingSource]]

| `WorkerSource`
| [[WorkerSource]]
|===
