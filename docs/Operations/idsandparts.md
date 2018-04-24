### [TOC](./TOC.md)
### [Back - Introduction](./introduction.md)

# The Hyperledger Composer Security Model
In Hyperledger Composer there are 2 important distinctions that need to be made, the first is what is an identity and the second is how does this relate to a participant

## Participants
In Hyperledger Composer all interactions that occur will be in the context of some defined participant. That participant must exist and thus must exist in a Participant registry. There is 1 exception to this rule and that is the start transaction which get's run when a business network is started for the first time. From then on all interactions must be in the context of a defined participant.

Because of this it's possible to track all activity down to the participant the performed the action. It's also possible to be able to restrict what a participant can do through ACL definitions.

Before you can perform any action within a business network however you have to have appropriate authority from Hyperledger Fabric itself. If you don't have the correct authority then Hyperledger Fabric won't let you be able to do anything.

## Identities
For the purposes of this book an _identity_ is going to be a public certificate and private key that represents the entity that wishes to interact on a Hyperledger Fabric network. Hyperledger Fabric supports the concepts of MSPs (Membership Service Provider) and is actually a pluggable implementation. You could provided your own implementation. The default implementation provided by Hyperledger Fabric is based on PKI (Public Key Infrastructure) and Hyperledger Composer can only work with this default implementation.

**IMPORTANT: You may have seen something that looks like a userid and password type of mechanism and what you see will be covered later but let's be clear. There is no userid/password mechanism for accessing a business network, the only way to access a business network is to present a valid identity. You won't even get past the Hyperledger Fabric security unless the identity is valid.**

How Hyperledger Fabric manages these identities is beyond the scope of this document, you will need to understand the operational aspects of Hyperledger Fabric. This book will assume that from a Hyperledger Fabric point of view, the identity presented is valid and Hyperledger Fabric permits access to the business network (of course Hyperledger Composer can then decide whether to permit any access to the business network).

### Public certificates
Public certificates contain something called a `Common Name`. Because of the examples/tutorials on the Hyperledger Composer documentation, this field tended to contain simple names such as 'admin', 'alice' or 'bob'. It's important to note that 2 certificates with the same common name do not represent the same identity. For example I can have 2 certificates with the Common name set to `admin`, but these are in fact 2 different identities. They just have the same name. We will discuss this more when using the composer identity commands for identity management later.

### Private key
The entity wishing to access the hyperledger fabric network must have access to their associated identity's private key whenever they want to interact with a fabric network. This private key is used to sign all interactions.

## How Hyperledger Composer maps from Identities to Participants
(Insert Diagram here)

The Hyperledger Composer runtime that forms part of the business network deployment creates and maintains an `identity registry`. It uses this to map the _identity's public certificate_ that has been presented to the Hyperledger Fabric by the Hyperledger Composer client application, Playground or Rest Server (we will talk about how these components do that later in the book) to an associated Participant.
Given that statement the following can be inferred

1. An _identity_ can **only** be associated with a single participant
2. A participant can be associated with more than 1 _identity_

We will discuss how this identity registry is managed (ie how the mapping is created and subsequently managed using the composer identity commands later in this book).

The identity registry holds 1 or more identity assets. The identity asset structure is described in the system model file `org.hyperledger.composer.system.cto` which can be found in the code base. It contains a number of properties for example

```
asset Identity identified by identityId {
    o String identityId
    o String name
...
    o IdentityState state
    --> Participant participant
}
```
The important point to note here is that an Identity is identifier by the identityId **NOT** the name (the name being the Common Name of the certificate). This means that you have many identities, all with the same `name` but they are not the same identity.

When a request from a client comes into the business network the composer runtime will first obtain the public certificate of the identity making the request from fabric. It then extracts the `public key` from the certificate and creates a unique identitifer from that public key. This identitier is used to find the Identity asset in the registry and must match an entry with the same `identityId` value. If it doesn't find one you get a message similar to this one.
```
Error: The current identity, with the name 'admin' and the identifier 'ea173e6aa087cc9b0bfe433ed2f01c8ff939ac100a2b6d67b57cf3fe750cb83c', has not been registered
```
Even if the certificate has the same `name`, the public key of that certificate did not match the one that has been registered and so they are not the same identity.

## Identity State and Activation
TBD

### [Next - Connection Profiles](./connectionprofiles.md)
