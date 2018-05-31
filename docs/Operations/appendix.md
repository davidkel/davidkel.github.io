### [TOC](./TOC.md)

# List of thoughts I still need to include somewhere
- Things that need to be included
  - Misc
     - version migration ????
     - commands that create cards for you and the issues there
     - reliability handling
     - detail how to fix composer runtime version for a bna and control it
     - how to turn event source off it you don't want a submit/notify mechanism
  - Docker Images
     - No included diagnostic tools
     - removed ability to install anything else for security reasons
     - based on alpine
     - issues if you copy binaries to it
     - CLI, Playground, Rest Server
     - why you might not want to use the images as is
     - PM2 is used to try to ensure reliability
  - Rest Server
     - discovery id used for system activities and bn parsing
     - it only uses the cloud wallet
     - explorer and how to disable
     - examples of identity management using it
     - Multi-user and authentication go hand in hand
     - uploaded cards are not stored in the cloud wallet
     - in-memory and mongodb
     - The REST server can be configured to authenticate clients. When this option is enabled, clients must authenticate to the REST server before they are permitted to call the REST API.
     - Mult-user: By default, the Hyperledger Composer REST server services all requests by using the Blockchain identity specified on the command line at startup. requires authentication setup.
     - APIKey
  - Diagnostics
     - CORE_VM_DOCKER_ATTACHSTDOUT
     - CORE_CHAINCODE_LOGGING
     - CORE_PEER_LOGGING
     - CORE_CHAINCODE_EXECUTETIMEOUT + client side timeout
     - CORE_CHAINCODE_STARTUPTIMEOUT + client side timeout
     - orderer logging
     - ca server logging
     - client version mismatch with runtime version
     - chaincode image fails to build, takes to long to build (REQUEST_TIMEOUT)
     - chaincode fails to register with peer.
     - Discuss this message in the peer logs

```
error: transaction returned with failure: Error: The current identity, with the name 'admin' and the identifier 'aa216b3767bf4e9a1e2e29ee43fe36a7fe188c0182ae501ddc8976a06c7765e1', must be activated (ACTIVATION_REQUIRED)
```    

     - fingerprint mismatch deploy/upgrade as well as diagnostics
     - Discuss this scenario

```
Exception: Error: Error trying to ping. Error: Composer runtime (0.19.8) is not compatible with client (0.19.5) Error: Error trying to ping. Error: Composer runtime (0.19.8) is not compatible with client (0.19.5) at _checkRuntimeVersions.then.catch (/home/composer/.npm-global/lib/node_modules/composer-rest-server/node_modules/composer-connector-hlfv1/lib/hlfconnection.js:790:34) at <anonymous> 
```

  - cloud wallets
    - wallet to hold sensitive info such as cards, connector info eg fabric keystore for keys & certs
    - can share a wallet through filesystem sharing, remote server
    - danger with fabric-ca admin if you store card with a secret, if 2 people use it at the same time, 2 enrollments can occur
    - cloud wallet backed card store

