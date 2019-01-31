### [TOC](./TOC.md)
### [Back - BNA](./packaging.md)

# Business Network Cards and Card Stores
Business network cards in composer were just a packaging of information to make it easier to reference and transport access to a specific hyperledger fabric network channel with a specific identity and optionally a specific business network on the defined fabric network channel. A card file was the packaged format of this information (using zip) and a card store was where these cards could be imported for use by applications, the composer cli, composer playground and composer rest server. A Business network card was therefore comprised of

- A hyperledger fabric connection profile (contrary to popular believe, the connection profile format is owned by hyperledger fabric and not hyperledger composer)
- An identity in the form of a public certificate file and private key file
- Metadata (which might include the name of the business network) of the form
```
{"version":1,"userName":"admin","businessNetwork":"empty-business-network","enrollmentSecret":"adminpw"}
```

A Business network card could also optionally not include a specific identity (represented by the public certificate and private key) but just a userName and enrollmentSecret in the metadata. This would drive composer to enroll that user to obtain real cryptographic credentials to be stored so that the caller could interact with Hyperledger Fabric and thus the business network.A Cloud wallet allowed for the storage of imported cards (and another other special stores required) in various different locations. Cloud Wallet was probably the wrong definition here as they are really just different types of persistence for a card store. The following persistent types were made available
- In memory
- Local File System
- Redis
- IBM Cloudant
- IBM Cloud Object Storage

There is no equivalent in the new fabric-network package of the node-sdk. The fabric-network api now expects the following
- The connection profile
- A Wallet that contains the identities which have already been imported into that wallet from a private key and public certificate.
- The name of the chaincode ID you will be interacting with (chaincodeID was equivalent to business network name in composer)

A Wallet is a persistent store for identities and fabric-network provides more than 1 but doesn’t match the stores available in composer. Currently it provides:
- In Memory
- Local File System
- CouchDB

Composer did provide support for HSM credentials too and the‘fabric-network’ module also provides an equivalent to this.

You will need to refer to the fabric documentation on using the fabric-network package to see how you now `connect` to a fabric network to interact with chaincode/smart contract.

### [Next - CLI](./cli.md)