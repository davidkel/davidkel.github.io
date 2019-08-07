# Developing clients using the Java SDK
TO BE DONE

## How to turn on SDK logging

```
Logging for the SDK can be enabled with setting environment variables:
ORG_HYPERLEDGER_FABRIC_SDK_LOGLEVEL=TRACE (or DEBUG)

ORG_HYPERLEDGER_FABRIC_CA_SDK_LOGLEVEL=TRACE

ORG_HYPERLEDGER_FABRIC_SDK_DIAGNOSTICFILEDIR=<full path to directory> # dumps protobuf and diagnostic data. Can be produce large amounts of data!
```

## How to turn on GRPC logging

To enable fine grained debug running the following logging configuration can be used.

Put this in a file grpc-debug-logging.properties:
```
handlers=java.util.logging.ConsoleHandler
io.grpc.netty.level=FINE
java.util.logging.ConsoleHandler.level=FINE
java.util.logging.ConsoleHandler.formatter=java.util.logging.SimpleFormatter
Run with -Djava.util.logging.config.file=/path/to/grpc-debug-logging.properties.
```
Also try the node way just in case

## Important connectivity notes
Firewalls, load balancers, network proxies can sometimes silently kill a network connections and prevent them from auto reconnecting. To fix this look at adding to Peers and Orderer's connection properties: 
```
grpc.NettyChannelBuilderOption.keepAliveTime, grpc.NettyChannelBuilderOption.keepAliveTimeout, grpc.NettyChannelBuilderOption.keepAliveWithoutCalls
```
Examples of this are in End2endIT.java
