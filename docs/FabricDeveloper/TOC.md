# Fabric Development notes
Hints and tips for the various developer areas.

My current recommendation is to avoid java chaincode, it's the least performant of the chaincode languages and being a multi-threaded language requires all sorts of options to tweak it's performance such as the size of the thread pool and it's not easy to manage the JVM environment (you would have to produce your own JavaEnv as currently chaincode as a server for java is not possible). Node (especially fabric 2.2 which uses node 12) generally wouldn't require any tuning of the environment, and Go will be the most performant chaincode language.


- [Guidelines for writing node chaincode](./writing-node-chaincode.md)
- [Guidelines for writing node clients](./writing-node-clients.md)
- [Guidelines for writing Highly Available node clients](./writing-HA-node-clients.md)
- [Go Chaincode](./gochaincode.md)
- [Go SDK](./gosdk.md)
- [Node Chaincode](./nodechaincode.md)
- [Node SDK](./nodesdk.md)
- [Java Chaincode](./javachaincode.md)
- [Java SDK](./javasdk.md)
