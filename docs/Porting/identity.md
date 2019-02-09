### [TOC](./TOC.md)
### [Back - Events](./events.md)

# Identity Management
Composer never worked with identities (ie the private key/public certificate combination that is used by hyperledger fabric) directly. Instead it would map an identity to a participant. The Composer runtime maintained a mapping between identities and participants in the Identity Registry. This mapping was managed through apis and made available to the end user via a CLI. CLI commands that managed this mapping were
- composer network start
- composer identity issue
- composer identity bind
- composer identity revoke 

The api's available were similar.
There is no equivalent in hyperledger fabric to this concept as it really is part of the composer programming model itself.

The `identity issue` capability also performed some extra steps using the fabric-ca. It would register a user to the fabric-ca-server and collect the secret that is returned.
Composer would, if only gven a card with a secret, would `enroll` that user automatically. 

You can perform manual `register` and `enrollment` against a fabric-ca-server using the `fabric-ca-client` CLI.
TODO: Provide a reference to the fabric-ca CLI

Identity records which provide the mapping between identities and participants are stored with a base key of `Asset:org.hyperledger.composer.system.Identity` and the extension key will be the value of the `identityId` field.

```
enum IdentityState {
    o ISSUED
    o BOUND
    o ACTIVATED
    o REVOKED
}

asset Identity identified by identityId {
    o String identityId
    o String name
    o String issuer
    o String certificate
    o IdentityState state
    --> Participant participant
}
```

On the client side you could get access to this registry by using the `getIdentityRegistry` on a business network connection that then provided a `registry` object to be able to work with the Identity Asset.

In Composer Transaction Processor functions you could use `getCurrentParticipant` to get the current participant executing the transaction. As there is no concept of a participant in chaincode/smart contracts there is no equivalent.
You could also use the `getCurrentIdentity` to get the current identity, the `client identity` api will allow you to get the same information.

If you are using the contract api then the identity api can be obtained from the context
```
public async someTransaction(ctx, ...) {
   // ctx.identity contains the identity apis
}
```

If you are using the old fabric-shim api then you would do this to get the identity api
```
const ClientIdentity = require('../chaincode').ClientIdentity;
...
public async someTransaction(stub, ...) {
   const identity = new ClientIdentity(stub);
}

### [Next - BNA](./packaging.md)