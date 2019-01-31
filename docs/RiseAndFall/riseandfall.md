# Hyperledger Composer, Some personal thoughts.
For many people, Hyperledger Composer was thought of as something to make writing for applications for Hyperledger Fabric easier but actually IMO this is far from the point of Composer. Hyperledger Fabric provides a complete Distributed Ledger Technology (DLT) capability designed to satisfy a range of solutions which require an immutable ledger shared across organisations (be it internal or external). Hyperledger Composer on the other hand was designed to solve a specific domain space around digital assets (or physical assets represented digitally) which is a common use case for DLT systems. Hyperledger Composer itself did not provide a DLT implementation but uses Hyperledger Fabric to provide that capability. In theory it could have used a different DLT implementation or even allow for the choice of DLT implementations, however even though the DLT implementation was pluggable in Hyperledger Composer, the design, capability and operational aspects of hyperledger composer are clearly driven by Hyperledger Fabric capability making it actually difficult, if not impossible, to use a different underlying DLT such as Hyperledger Sawtooth for example.
In general, composer should define it's own capabilities and not be driven by new capabilities in fabric. However, fabric operational capabilities such as service discovery make sense to be exposed as composer operational capabilities are driven by the capabilities of fabric because composer applications can be deployed to hyperledger fabric.

## What is Hyperledger Composer
Hyperledger Composer is a runtime framework, operational tools and development tools to help you create a digital asset application that can be deployed onto a Hyperledger Fabric network. It makes no requirement on the design of this network (see later however for features required to make hyperledger composer production ready) but has to have knowledge of that network, through the hyperledger fabric connection profile, in order to be able to work with that network setup. The following are a summary of the capabilities

- A modeling language to model the assets, participants, transactions and events that make up the digital asset application
- A language to describe access control to the above artifacts
- A query language to be able to perform rich queries on the artifacts
- A runtime that compiles and manages
  - model validation and conformity
  - ACL checking
  - query compilation and execution
  - inbuilt transactions
  - transaction dispatching
  - a TP function API
- Client API to interact with the runtime/business network
- Client API to perform various operational capabilities
  - Business network deployment
  - identity management
  - Network Card management and storage
- A generic REST Server providing both business network interaction and identity management
- A CLI for operational and some testing facilities
- A set of development tools
  - Playground
  - Visual Studio Code plugin
  - Yo Generators
  - Embedded connector for unit testing
  - Cucumber steps

This provided a complete eco-system in the lifecycle of a digital asset application from development to production.

## Composer applications and Production readiness
As much as it was thought that when 0.19 was released, that this was production ready, it wasn't. Some of it was due to missing capability in Composer and some was because Hyperledger Fabric V1.1 was not ready for certain types of application, for example applications that made use of rich queries. We learned that the limiting factors of composer were performance and scalabity. Composer provided a lot of capability but that also meant it would use more CPU and memory to handle, for example model validation and serialisation can be expensive especially if your model was complex. The more ACL rules and more resource artifacts were processed in the runtime, the more CPU and memory were required. Finally limitations with fabric and node.js chaincodes also play a part. So what is required to make Composer production ready bearing in mind the limitations ?

- Card update capability. Given that networks can change, connection profile changes are not managable currently, not only for the card stores but also for the rest server (as in multi-user mode it uses it's own unique store, not a card store)
- Iterator support for getAll on resources. As the more resources you have if processed all in 1 go, the more processing and more memory required
- Exploit plagination for rich queries in Hyperledger Fabric v1.3, without this again you would be loading large datasets into memory and also taking a long time to process the complete dataset
- support shrinkwrap in a bna to mitigate against undeterministic chaincode images (ie a peer could run a different version of an npm module to another) by fixing the versions of npm modules used exactly
- support require()
- support shrinkwrap. Need this to ensure we can guarantee the chaincode image built is consistent across peers as well as providing knowledge of exactly what versions have been installed for a chaincode image.
- document the simulator and add integration tests (provides a much nicer development environment, no more install/upgrade on real fabrics taking so long)
- use the simulator for local playgrounds when not connecting to a real fabric (ie disable to web profile which will only be for the online playground)
- document how to run a fabric in development mode and test a business network
- REST Server should not use connection profiles of cards, it should use the one it used for discovery.
- Use of secrets for cards and support in the runtime for this should be restricted to development time only. Production only should provide the capability for users to enroll their cards if it is a secret card file and handle secret 1-time only cards appropriately. Card files with secrets are one of the main causes of identity problems faced by composer users.
- support for Mutual TLS. Composer cannot deploy to fabric networks with mutual TLS enabled.

There are some things that still cannot be addressed and still require a solution from Hyperledger Fabric around Node.JS Chaincode support
- When installing a business network onto peers for different organisations a different npmrc file may be required unique to that organisation. This will result in a different node.js chaincode package and then this will result in fingerprint mismatches trying to validate transactions.
- Fabric manages the npm install process of business networks which can fail as it means the peers need network access and things like network issues or no access to the internet are a problem.
- Node runtimes are single threaded, they won't take advantage of multiple cores and chaincode must not have long running code that doesn't yield occasionally to allow other transactions to be processed. It's possible for a transaction if it gets into an infinite loop to lock out the business network completely or a long running transaction can delay short running transactions to complete if they don't yield.
- Node has a memory limit of 1Gb by default. This is not configurable at the moment in fabric, so you cannot tell node to use more memory
- channels with the same chaincode (ie same name and version) will use a single chaincode instance putting more through the same single threaded node instance and no way to control this scalability limiting issue. You can of course get around it by using the same chaincode name but a slightly different version for each channel to get around this limitation.

### Conclusion
At the moment, Composer is not production ready. The main issues being around Rich queries and large/infinite datasets that could exist. But you should also consider that Composer adds performance overheads which can never be removed. You use composer to provide a capability you need and with that you have to accept the overhead that comes with using these generic capabilities. It will never be possible to achieve the same performance as Go or pure node.js chaincode which can implement just what it needs in an efficient manner for the use case.
Unfortunately there wasn't enough engagement from the community to drive Composer forward. It needed far more contributions from the community which sadly never happened. 
Composer can be used with hyperledger fabric upto the final V1 release, v1.4. But Composer will not work with Hyperledger fabric V2 and it's unlikely there will ever be a contribution from the community to make it so.

Composer remains a great learning tool to get to understand the concepts of digital assets and possibly even for demos as well. If you consider it for a POC then you need to be prepared to throw away a lot of what you coded in Composer and start again as there is no equivalent capability in native fabric applications. Some of the new programming model capability in native fabric may lessen the impact somewhat however. 


