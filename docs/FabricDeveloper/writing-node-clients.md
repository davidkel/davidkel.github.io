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
* Connection profile formats have changed between 1.4 and 2.2 of the node sdk
  * It's now unclear what format is common across all SDKs in both low level and gateway levels so you need to refer to the specific version of whichever SDK you are using (ie these connection profiles are not common and will have SDK specific as well as gateway vs lowlevel differences)
  * in node sdk 2.2 the only relevant sections appear to be (ie no client or certificate authority definitions now)
    * peers
      * can now specify request-timeout as part of grpcOptions
    * orderers
      * can now specify request-timeout as part of grpcOptions
    * channels
      * can't specify peer roles anymore
    * organizations
      * can't specify associated certificate authorities
      * can't specify admin keys anymore

## Gateways

* reuse
  * bound to a single identity
  * can't change that identity
  * do not constantly connect/disconnect it (especially across async/threads that's really bad)
  * supports concurrent requests through the same gateway instance
  * stale cache policy

## Use dynamic connection profiles and discovery

* can become stale and out of date
* will manage the live status of peers
  * Can change the refresh time
* doesn't manage the live status of orderers
* doesn't stop you from using org/peer targeting if required
  * org targeting in 1.4 doesn't work as expected

## Connection information

In order to invoke chaincode you need the following information

* a connection profile
  * For low level interation you can create the objects yourself this is not recommended
* an identity (public certificate and private key or HSM PKCS11 information (slot, pin etc))
* The chaincode id
* The channel the chaincode is deployed to

You might for example consider creating a package (zip file) that contains this information for which you can then easily establish a point to point connection between your client code and the chaincode you want to invoke.

## Identity Management

* you don't have to keep enrolling the same id, only enroll when you need to renew a certificate that is about to expire
* enrolling generates a new private key it doesn't re-use the existing one even if you re-enroll using your existing certificate
* have a certificate management policy to handle (your provider may handle the peers, orderers and tls certificate automatically for you)
  * certificate expiration
  * certificate revocation

## Security

Store identities for use by client applications securely

* storing on the filesystem may not be appropriate
  * consider storing onto a file system by injecting into a running container whose file system is destroyed
* the couchdb wallet impl doesn't use https so that may be a concern (Check this)
* retrieve the certificate and private key (if not using HSM) from some secure environment such as vault and load into an in memory wallet
* Use HSM's or other provider options such as KeyProtect to secure your identities

I would recommend that you secure not only all private keys but also public certificates and connection profiles as well.

## Limitations of fabric-network

fabric-network is more limited in what you can do compared to the original 1.4 low level api.

* It doesn't allow you to do any processing on proposal responses before they are submitted to the orderer
  * 2.2 now performs a compare before they are sent to the orderer which you may not want depending on your endorsement requirements. Unfortunately you can't disable it currently

* query handling is limited, although you can write plugins the information presented to the plugin in limited. For example if you need to query when a block event is received then you would want to query the same peer the block event came from but a plugin won't have access easily to that information
* event handling may also be limited. You can again write plugins but in 1.4 the reliability of the plugin would not be as good as the built in plugins. I don't know about the 2.2 plugins
* event handling plugin format changed from 1.4 to 2.2 so things may have been improved and be simpler to do now.

The only option if the fabric-network doesn't meet your needs is to code the the lower level apis but again in node sdk 2.2 this isn't being advertised is an approach.

## Submit vs Evaluate

The difference between evaluate and submit are as follows

* evaluate sends to peers dictated by the query plugin selected
* submit will send to peers based on peer/org targeting or endorsement policy
* submit sends proposals to the orderer
  
Take great care to make sure you don't submit a transaction that returns private data. If you do then that private data will effectively become public as it will be committed to the blockchain (even if validation fails)

If you need to retrieve private data as part of a transaction submission then you will have to split the call into a submission (wait for the commit) and then an evaluation, but that means you lose the atomicity of a submission.

## Serializers

If you can, use a common serializer implementation between your chaincode implementation and the client application. A Client application needs to serialize and deserialize payloads when communicating with the chaincode and conversely your chaincode/contract needs to do the same. This may not be so easy if your client and chaincode are written in different languages. Also the contract API has a build in JSON serializer, but it should be possible to replace that with your own implementation.

## Event handling

Use a suitable commit event strategy

## Query Handling

Use a suitable query handler strategy

## Other

ensure localhost is set to false by default developers can choose true themselves or better still handle local environments better for name resolution
