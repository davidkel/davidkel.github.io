### [TOC](./TOC.md)

# Introduction
This document will provide some guidance to help you in your journey from porting a composer based application to one based on Hyperledger Fabric 1.4.

One thing that needs to be made very clear from the beginning; Composer provided a vast amount of inbuilt capability and even with the new programming models in both the node chaincode and the node client sdks, there will be a large amount of capability that will require you to provide your own capability. All this guide can do is point you to some of the capabilities in fabric that may be able to help. Of course you could look for external libraries to help. Also you don't have to continue with node.js chaincode. Hyperledger fabric supports Java and Go as well, but be aware that capabilities in Chaincode implementations and SDKs are not always in sync, some languages get newer capabilities before other languages.

Depending on this you may decide to export your existing data using the Composer client SDK and start again with a completely new application and re-import your data into a new blockchain. Alternatively this guide will provide information on how composer stored your data in hyperledger fabric as well as the metadata it maintained for it's own use in order to provide you with a complete picture.

A quick summary of the capabilities are given here. The sections will provide more information

| Composer Capability | comments |
| ------------------- | -------- |
| modeling language | No equivalent capability in fabric |
| resource creation and data validation | No equivalent capability in fabric |
| Query Language | use mango queries |
| ACLs | no equilavent capability in fabric, Attribute based access control may be of use to create your own |
| Historian | no equivalent capability in fabric, getHistoryForKey may be of use or you can provide your own equivalent |
| Events | need to use fabric-client api as nothing available in fabric-network |
| Identity Management | no equivalent capability in fabric as no concept of participants |
| BNA packaging | Peer chaincode package command exists |
| CLI | Cli was specific to composer capabilities, look at the Peer CLI for equivalent fabric capabilities |
| Developer tools | No equivalent capability in fabric. But there are others providing development tools |
| Loopback connector | No equivalent capability in fabric, was never documented in Composer either |
| Rest Server | No equivalent capability in fabric, consider node.js express app |

| Request and Post TP API | use standard node.js npm modules such as request |
| Registry API | No fabric equivalent will require own implementation in chaincode |
| Transaction invocation | Some capability is available in fabric-network, others will require own solution in both client code and chaincode |

### [Next - Modeling Language](./modeling.md)
