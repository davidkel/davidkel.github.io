### [TOC](./TOC.md)

# Introduction
Welcome to this operations guide for Hyperledger Composer. This book provides information about the operational aspects of composer with respect to Hyperledger Fabric. It assumes a working knowledger of Hyperledger Fabric, Hyperledger Composer, Docker and NPM (The Node Package Facility). It covers aspects such as security (identities, participants and ACLs specifically around operations), deployment aspects, using a Fabric-ca for identities and how to obtain diagnostic information.

The book is currently based on capabilities of Hyperledger Fabric 1.1 and Hyperledger Composer 0.19.x

## Playground
Playground is not really covered in this book. Currently it isn't a tool that should be used for operational aspects of running and managing a business network running on Hyperledger Fabric. Playground at this time is a development tool and should only be used in assisting the development of business networks and should only ever be used to connect to a real fabric, if that fabric is setup purely for the development of business networks. I don't recommend you use playground to connect to a production fabric to perform deploy/upgrade or identity management of a production business network

There will be some mention of playground in the context of the docker images provided by Hyperledger Composer however.

### [Next - The Hyperledger Composer security model](./idsandparts.md)
