= Web UI -- Spark Application's Web Console

*Web UI* (aka *Application UI* or *webUI* or *Spark UI*) is the web interface of a Spark application to monitor and inspect Spark jobs in a web browser.

.Welcome Page of web UI &mdash; Jobs Tab
image::spark-webui-jobs.png[align="center"]

Every Spark application (xref:ROOT:SparkContext.adoc[]) starts a xref:ROOT:spark-SparkContext-creating-instance-internals.adoc#ui[web UI] that is available at `http://[driverHostname]:4040` by default.

NOTE: The default port can be changed using xref:spark-webui-properties.adoc#spark.ui.port[spark.ui.port] configuration property. `SparkContext` will increase the port if it is already taken until a free one is found.

web UI comes with the following tabs (_pages_):

. xref:spark-webui-jobs.adoc[Jobs]
. xref:spark-webui-stages.adoc[Stages]
. xref:spark-webui-storage.adoc[Storage]
. xref:spark-webui-environment.adoc[Environment]
. xref:spark-webui-executors.adoc[Executors]

TIP: You can use the web UI after the application has finished by persisting events (using xref:spark-history-server:EventLoggingListener.adoc[EventLoggingListener]) and using xref:spark-history-server:index.adoc[Spark History Server].

NOTE: All the information that is displayed in web UI is available thanks to xref:spark-webui-JobProgressListener.adoc[JobProgressListener] and other xref:ROOT:SparkListener.adoc[]s. One could say that web UI is a web layer over Spark listeners.
