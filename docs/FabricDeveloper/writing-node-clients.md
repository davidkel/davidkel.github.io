# Guidelines for writing Node Clients

There are currently 2 versions of Fabric Node SDKs, 1.4 and 2.2. The 1.4 library can still be used with Fabric 2.2 (And support is claimed but unsure about using it for future 2.x releases). The 1.4 SDK won't support new features in fabric 2.2 such as receiving blocks containing private data.

## Deciding between 1.4 SDK and 2.2 SDK

If you are starting out and planning to use the 1.4 SDK then you need to be aware of migration concerns if you then are going to move to using the 2.2 SDK. However there are advantages to using the 1.4 SDK over the 2.2.

* Supports and documents a more comprehensive API (known as the low level api) which allows you to develop applications which may not be possible using the fabric-network Gateway api
  * There is still a more comprehensive API in 2.2 but there are no examples and as such has not had any field testing and not sure if it's really intended for use by real world applications so may not be supported (although to write plugins you may have to use this low level api)
  * The 1.4 low level api is quite different to the 2.2 low level api so you do code to this lower level api in 1.4 be prepared to re-write it when moving to the 2.2 sdk
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
  
## Deciding between fabric-client and fabric-network in 1.4

You don't have the same choice in sdk 2.2. Fabric-network (gateway) is the preferred api to use to write business applications, but the fabric-client (low-level) is more powerful. Unfortunately because fabric-client has gone from the 2.2 sdk (there is a low level api but it's not documented and there is no encouragement to use it), it's hard to recommend using this api now. It's unfortunate that the fabric-network api can be too restrictive making it impossible to do certain things.

## what needs to be considered when writing a client application

Client applications need to primarily consider the following

* executing transactions that will be added to the blockchain (submit)
* retrieving information from the world state or private data (evaluate)
* handling events

Other things the application needs to consider are

* High availability (this will be covered in a separate page)
* Scalability and performance (eg multiple instances)
* Security

## How to correctly using fabric-network Gateways

(NEEDS WORK)

* reuse
  * bound to a single identity
  * can't change that identity
  * do not constantly connect/disconnect it (especially across async/threads that's really bad)
  * supports concurrent requests through the same gateway instance
  * stale cache policy

## Use dynamic connection profiles and discovery

(NEEDS WORK)

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

## Submit vs Evaluate in fabric-network

The difference between evaluate and submit are as follows

* evaluate sends to peers dictated by the query plugin selected
* submit will send to peers based on peer/org targeting or endorsement policy
* submit sends proposals to the orderer
  
Take great care to make sure you don't submit a transaction that returns private data. If you do then that private data will effectively become public as it will be committed to the blockchain (even if validation fails)

If you need to retrieve private data as part of a transaction submission then you will have to split the call into a submission (wait for the commit) and then an evaluation, but that means you lose the atomicity of a submission.

## Serializers

If you can, use a common serializer implementation between your chaincode implementation and the client application. A Client application needs to serialize and deserialize payloads when communicating with the chaincode and conversely your chaincode/contract needs to do the same. This may not be so easy if your client and chaincode are written in different languages. Also the contract API (if you are using the contract API) has a build in JSON serializer, but it should be possible to replace that with your own implementation.

## Event handling

Fabric Network tries to simplify the complexity around event handling as well as trying to add High Availability features.

One thing to remember is that if you plan to listen for chaincode events you must ensure you are listening for full blocks not filtered blocks (This I would hope only applies to the low level 1.4 api and is all handled for you by fabric network when registering to listen for chaincode events)

### Why events are important

* Transaction Committed events
* Chaincode Events
* Block Events (useful for off-chain storage for querying)

(WORK NEEDED)

### Checkpointing

In order to ensure you don't miss events you need to implement a checkpointing mechanism to track which blocks and transactions within blocks you have processed. fabric-network does provide the concept of checkpointing as discussed later.

This is a large topic so won't be covered outside of what fabric-network provides

### Event Handling Strategies in fabric-network

There are a set of built in event handling strategies you can choose from, so try to choose one that best suites your needs. The Event Handling Strategy refers to how many transaction committed events and from where those events come from should be received before the submit call unblocks.

If you plan to add your own listeners then you can't set the event handling strategy to null, you have to have selected one in order to be able to add further listeners, which personally seems very odd to me as an event handling strategy is applicable only to transaction commit events

It is possible to plug in your own event handling strategies but I cannot say whether it's possible to develop a reliable (ie ensure that you don't create any sort of memory or resource leak) or a suitable strategy that can meet your needs.

If you don't care when a submit completes, then you can either just not await on the submit but be aware that events will still be listened for or you can set the event listening strategy to null, but if you do then you won't actually know if your transaction was successful in any way for example you won't know if an MVCC_READ_CONFLICT occurs.

In 2.2 the ability to write your own custom strategy exists and there is some information in a tutorial on the SDK doc website, but it isn't comprehensive enough and you also need to understand the low level api which has no real documentation now.

### Checkpointing in fabric-network

If you want to ensure you never miss a chaincode or block event then you must enable checkpointing. Checkpointing ensures that if connection to an eventhub is lost (either through network connectivity or your application is stopped and restarted) then it will request events starting from an appropriate block number. The checkpointing implementation in 1.4 works at a transaction level (as a block can contain multiple transactions) to ensure that if the processing was in the middle of a block when handling events, it will request that block and process the remaining transactions. This handles a common problem with home grown checkpointing where it checkpoints at a block level and so the possibility of either missing events or having duplicate events can arise.

With 1.4 the checkpoint persistence is to a file and I believe that this file although not big is not pruned so constantly grows. But as it is persisted to a file you need to make sure this file doesn't get lost so persisted on a non volatile file system. You will also have to give careful consideration to running multiple instance of apps that do event listening as the file checkpointer implementation may not work on a single file shared across these multiple instances.

checkpointing isn't used for handling transaction committed events and thus what might unblock a submit request

You can write your own custom checkpointing implementation however I would recommend looking at the official implemented source code for the inbuilt checkpointing as it will demonstrate some of the idosynchrasies which you need to be aware of in order to get it right.

Also in 1.4 you can provide your own custom plugin to handle how event hubs a selected and reconnected, but I've not looked at this. This unfortunately shows how messy the whole 1.4 event handling system is for users to try to implement event handling that suites their needs.

I cannot comment on the 2.2 implementation at this time as I have not looked at it, but there doesn't appear to be any tutorials about it nor any information about whether the implementation is customisable in any way.

## Query Handling

Some requests to chaincode are about retrieving information and not updating private data or the world state. In most cases you don't want to submit these requests to the orderer in order for it to appear on the blockchain.

Fabric network tries to make things easier here by first providing the api `evaluateTransaction` and then proving the ability to select query strategies to handle where you go to get the information from

### Query Strategies

Fabric-network provides 2 query strategies in 1.4

* Single
* Round Robin

Both of these strategies will only query peers in your organisation (so long as you have peers otherwise it won't be able to query at all). The reasons are

* You should only trust the data stored and returned by your peers
* Other organisations won't want constant query type traffic on their peers

The single implementation works by "locking" onto the first responding peer. It then will continue to use that peer. If that peer cannot be contacted then it will find another responding peer and "lock" onto that one.
Round Robin will loop through each of the peers for each request, if a peer doesn't respond it will try the next in the list

In 2.2 2 new strategies appear which add means it will prefer to go to your org but will try other orgs if it can't. I'm not sure these are sensible options as it can contrevene both the above reasons so I don't actually understand the problem they would solve. orgs that don't have peers will have agreements with orgs that the can trust and should use so it would not solve that either.

Both 1.4 and 2.2 provide the ability to develop your own strategy plugins, however as discussed in the High Availability page on writing node clients, this plugin capability at the moment may be too limited to address all the considerations needed.

## Other

ensure localhost is set explicitly in the discovery options and ensure that the default in your application is `false` (as it's set to true by default which I consider a crazy default). Developers and their development environments set say an environment variable to change the default value to true themselves and will give them a better understanding of what is going on. It also means you don't inadvertently deploy to a real environment such as production with a flag that isn't going to work. The ideal situation is you set up a development environment that is more like the real world, ie use real name resolution rather than using localhost and this value will always be false for these environments.

## servicability

- always have log points
- have the ability to turn on node sdk logging
