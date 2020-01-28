### [TOC](./TOC.md)
### [Back - Query Language](./querylang.md)

# ACLs

There is no equivalent to ACLs in fabric. Acls controlled what participants could do what with various resources, which included Composer system resources as well as the applications defined data resources. You will need to create your own concept on how to perform access control. When an identity was used to access fabric, this was mapped to a composer participant. You can get the identity used to invoke the transaction using the `getCreator` api or more likely you should using the `client identity` api which extracts the public certificate and provides easy access to information about this certificate. 
If you are using the contract api then the identity api can be obtained from the context
```
public async someTransaction(ctx, ...) {
   // ctx.clientIdentity contains the identity apis
}
```

If you are using the old fabric-shim api then you would do this to get the identity api
```
const ClientIdentity = require('../chaincode').ClientIdentity;
...
public async someTransaction(stub, ...) {
   const identity = new ClientIdentity(stub);
}
```

## Fabric Attribute based access control

You could make use of attribute based access control by putting attributes into the certificate you issue to users and then using the client identity package to access those attributes to make access control based decisions. Careful consideration is required for this because it is a major task to address changes to using this implementation. For example if you decide to add or remove or change the meaning of the attribute then all the currently issued certificates will need to be revoked and new ones issued. 

More information can be found around attributes in certificates 
https://hyperledger-fabric-ca.readthedocs.io/en/latest/users-guide.html#registering-a-new-identity
and the `Client Identity` APIs
https://hyperledger-fabric.readthedocs.io/en/release-1.4/chaincode4ade.html?highlight=client%20identity#chaincode-access-control

## Fabric policy based access control

Even with Composer you should be familiar with the fine-grained access control for resources in fabric such as access to the ledger itself and event handling as these could still be used to bypass the composer ACL capabilities of controlling the visibility of data. Information on this can be found https://hyperledger-fabric.readthedocs.io/en/release-1.4/access_control.html


### [Next - Historian](./historian.md)