# Guidelines for writing Node Clients

There are currently 2 versions of Fabric Node SDKs, 1.4 and 2.2. The 1.4 library can still be used with Fabric 2.2 (And support is claimed but unsure about using it for future 2.x releases). The 1.4 SDK won't support new features in fabric 2.2 such as receiving blocks containing private data. 

If you are starting out and planning to use the 1.4 SDK then you need to be aware of migration concerns if you then are going to move to using the 2.2 SDK. However there are advantages to using the 1.4 SDK over the 2.2.

* Supports a more comprehensive API (known as the low level api) which allows you to develop applications which may not be possible using the fabric-network Gateway api
  * There is still a more comprehensive API in 2.2 but there are no examples and as such has not had any field testing and not sure if it's really intended for use by real world applications
  * They are not compatible so you do code to this lower level api in 1.4 be prepared to re-write it when moving to the 2.2 sdk
* Includes an operational api (but only supports the 1.4 lifecycle implementation)
  * There is a 3rd party operational api [https://www.npmjs.com/package/khala-fabric-admin]
* Provides the ability to query the ledger and decode the blocks
  * This is still possible in 2.2 but you need to work out how to perform the correct query (ie using qscc) and then you need to decode the blocks yourself. People have done this so there could be articles to find using google (other web search engines exist)
* The Gateway apis between 1.4 and 2.2 are mostly compatible but there will still be a migration exercise required see [https://hyperledger.github.io/fabric-sdk-node/release-2.2/tutorial-migration.html]
* The node sdk 2.2 uses grpc-js (a pure javascript implementation of grpc) whereas the 1.4 sdk used a version that required a native component which more often than not now requires the npm install to compile the C source code (if a pre-compiled binary could not be downloaded). This extends the NPM install time, the native version of grpc for node is not being improved and possibly not maintained either and the pure javascript version, grpc-js, appears to perform better. The 1.4 sdk is unlikely to move to this version however because grpc-js is not fully backward compatible with native grpc and this could break some existing applications coded to the 1.4 sdk.
* The node sdk 1.4 is had more field testing compared to 2.2 however and 1.4 is now a very stable release.


## Gateways
* reuse
* bound to a single identity
* can't change that identity
* do not constantly connect/disconnec it
* stale cache policy


## Use dynamic connection profiles and discovery

## Connection information
* profile, identity, chaincode id, channel

## Identity Management
* don't keep enrolling the same id
* have a certificate management policy to handle
  * certificate expiration
  * certificate revocation
* store identities securely (filesystem may not be appropriate, use in memory instead and load dynamically)

## Security
* secure connection profiles and public certs as well as private keys

## Limitations of fabric-network

## Submit vs Evaluate

## Serializers
Use a common serializer between your chaincode implementation and the client application

## Event handling
Use a suitable commit event strategy

## Query Handling
Use a suitable query handler strategy

## Other
endure localhost is set to false by default developers can choose true themselves or better still handle local environments better for name resolution





