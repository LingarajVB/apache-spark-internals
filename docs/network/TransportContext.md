= TransportContext

*TransportContext* is used to create a <<createServer, server>> or <<createClientFactory, ClientFactory>>.

== [[creating-instance]] Creating Instance

TransportContext takes the following to be created:

* [[conf]] xref:network:TransportConf.adoc[]
* [[rpcHandler]] xref:network:RpcHandler.adoc[]
* [[closeIdleConnections]] closeIdleConnections flag (default: `false`)

TransportContext is created when:

* ExternalShuffleClient is requested to xref:storage:ExternalShuffleClient.adoc#init[initialize]

* YarnShuffleService is requested to xref:spark-on-yarn:spark-yarn-YarnShuffleService.adoc#serviceInit[serviceInit]

== [[createClientFactory]] createClientFactory Method

[source,java]
----
TransportClientFactory createClientFactory(
  List<TransportClientBootstrap> bootstraps)
----

createClientFactory...FIXME

createClientFactory is used when TransportContext is requested to <<initializePipeline, initializePipeline>>.

== [[createChannelHandler]] createChannelHandler Method

[source, java]
----
TransportChannelHandler createChannelHandler(
  Channel channel,
  RpcHandler rpcHandler)
----

createChannelHandler...FIXME

createChannelHandler is used when TransportContext is requested to <<initializePipeline, initializePipeline>>.

== [[initializePipeline]] initializePipeline Method

[source, java]
----
TransportChannelHandler initializePipeline(
  SocketChannel channel) // <1>
TransportChannelHandler initializePipeline(
  SocketChannel channel,
  RpcHandler channelRpcHandler)
----
<1> Uses the <<rpcHandler, RpcHandler>>

initializePipeline...FIXME

initializePipeline is used when:

* `TransportServer` is requested to xref:network:TransportServer.adoc#init[init]

* `TransportClientFactory` is requested to xref:network:TransportClientFactory.adoc#createClient[createClient]

== [[createServer]] Creating Server

[source, java]
----
TransportServer createServer()
TransportServer createServer(
  int port,
  List<TransportServerBootstrap> bootstraps)
TransportServer createServer(
  List<TransportServerBootstrap> bootstraps)
TransportServer createServer(
  String host,
  int port,
  List<TransportServerBootstrap> bootstraps)
----

createServer simply creates a TransportServer (with the current TransportContext, the host, the port, the <<rpcHandler, RpcHandler>> and the bootstraps).

createServer is used when:

* `NettyBlockTransferService` is requested to xref:storage:NettyBlockTransferService.adoc#createServer[createServer]

* `NettyRpcEnv` is requested to `startServer`

* `ExternalShuffleService` is requested to xref:deploy:ExternalShuffleService.adoc#start[start]

* Spark on YARN's `YarnShuffleService` is requested to `serviceInit`
