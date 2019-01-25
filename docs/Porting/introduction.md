### [TOC](./TOC.md)

# Introduction
This document will provide some guidance to help you in your journey from porting a composer based application to one based on Hyperledger Fabric 1.4.

One thing that needs to be made very clear from the beginning; Composer provided a vast amount of inbuilt capability and even with the new programming models in both the node chaincode and the node client sdks, there will be a large amount of capability that will require a complete re-engineering of the application.

Depending on this you may decide to export your existing data using the Composer client SDK and start again with a completely new application and re-import your data into a new blockchain. Alternatively this guide will provide information on how composer stored your data in hyperledger fabric as well as the metadata it maintained for it's own use in order to provide you with a complete picture.

### [Next - Modelling Language](./modelling.md)
