### [TOC](./TOC.md)

# Introduction
This document will provide some guidance to help you in your journey from porting a composer based application to one based on Hyperledger Fabric 1.4.

One thing that needs to be made very clear from the beginning; Composer provided a vast amount of inbuilt capability and even with the new programming models in both the node chaincode and the node client sdks, there will be a large amount of capability that will require you to provide your own capability. All this guide can do is point you to some of the capabilities in fabric that may be able to help. Of course you could look for external libraries to help. Also you don't have to continue with node.js chaincode. Hyperledger fabric supports Java and Go as well, but be aware that capabilities in Chaincode implementations and SDKs are not always in sync, some languages get newer capabilities before other languages.

Depending on this you may decide to export your existing data using the Composer client SDK and start again with a completely new application and re-import your data into a new blockchain. Alternatively this guide will provide information on how composer stored your data in hyperledger fabric as well as the metadata it maintained for it's own use in order to provide you with a complete picture.

### [Next - Modelling Language](./modelling.md)
