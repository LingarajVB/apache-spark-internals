= Configuration Properties

The following contains the configuration properties of <<EventLoggingListener, EventLoggingListener>> and <<HistoryServer, Spark History Server>>.

NOTE: xref:EventLoggingListener.adoc[EventLoggingListener] is responsible for writing out JSON-encoded events of a Spark application to an event log file that xref:HistoryServer.adoc[HistoryServer] can display in a web UI-based interface.

[[EventLoggingListener]]
.EventLoggingListener's Spark Properties
[cols="30m,70",options="header",width="100%"]
|===
| Name
| Description

| spark.eventLog.buffer.kb
a| [[spark.eventLog.buffer.kb]] Size of the buffer to use when writing to output streams.

Default: `100`

| spark.eventLog.compress
a| [[spark.eventLog.compress]] Whether to enable (`true`) or disable (`false`) event compression (using a xref:io:CompressionCodec.adoc[CompressionCodec])

Default: `false`

| spark.eventLog.dir
a| [[spark.eventLog.dir]] Directory where Spark events are logged to (e.g. `hdfs://namenode:8021/directory`)

Default: `/tmp/spark-events`

The directory must exist before xref:ROOT:spark-SparkContext-creating-instance-internals.adoc#_eventLogger[Spark starts up].

| spark.eventLog.enabled
a| [[spark.eventLog.enabled]] Whether to enable (`true`) or disable (`false`) persisting Spark events.

Default: `false`

| spark.eventLog.logBlockUpdates.enabled
a| [[spark.eventLog.logBlockUpdates.enabled]][[EVENT_LOG_BLOCK_UPDATES]] Whether xref:EventLoggingListener.adoc[EventLoggingListener] should log RDD block updates (`true`) or not (`false`)

Default: `false`

| spark.eventLog.overwrite
a| [[spark.eventLog.overwrite]] Whether to enable (`true`) or disable (`false`) deleting (or at least overwriting) an existing xref:EventLoggingListener.adoc#inprogress[.inprogress] event log files

Default: `false`

| spark.eventLog.testing
a| [[spark.eventLog.testing]] *(internal)* Whether to enable (`true`) or disable (`false`) adding JSON-encoded events to the internal `loggedEvents` array for testing

Default: `false`

|===

[[HistoryServer]]
.HistoryServer's Spark Properties
[cols="30m,70",options="header",width="100%"]
|===
| Name
| Description

| spark.history.fs.logDirectory
| [[spark.history.fs.logDirectory]] The directory for event log files. The directory has to exist before starting History Server.

Default: `file:/tmp/spark-events`

| spark.history.kerberos.enabled
| [[spark.history.kerberos.enabled]] Whether to enable (`true`) or disable (`false`) security when working with HDFS with security enabled (Kerberos).

Default: `false`

| spark.history.kerberos.keytab
| [[spark.history.kerberos.keytab]] Keytab to use for login to Kerberos. Required when `spark.history.kerberos.enabled` is enabled.

Default: (empty)

| spark.history.kerberos.principal
| [[spark.history.kerberos.principal]] Kerberos principal. Required when `spark.history.kerberos.enabled` is enabled.

Default: (empty)

| spark.history.provider
| [[spark.history.provider]] Fully-qualified class name of the xref:ApplicationHistoryProvider.adoc[ApplicationHistoryProvider]

Default: xref:FsHistoryProvider.adoc[org.apache.spark.deploy.history.FsHistoryProvider]

| spark.history.retainedApplications
| [[spark.history.retainedApplications]] How many Spark applications xref:HistoryServer.adoc#retainedApplications[HistoryServer] should retain

Default: `50`

| spark.history.ui.maxApplications
| [[spark.history.ui.maxApplications]][[HISTORY_UI_MAX_APPS]] How many Spark applications xref:HistoryServer.adoc#maxApplications[HistoryServer] should show in the UI

Default: (unbounded)

| spark.history.ui.port
| [[spark.history.ui.port]][[HISTORY_SERVER_UI_PORT]] The port of History Server's web UI.

Default: `18080`

|===
