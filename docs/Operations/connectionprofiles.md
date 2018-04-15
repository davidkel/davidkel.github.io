### [TOC](./TOC.md)
### [Back - The Hyperledger Composer security model](./idsandparts.md)

# Connection Profiles
Connection profiles are a fundamental part of a business network card (see later). The purpose of a connection profile is to describe the components that make up a Hyperledger Fabric Network. They describe

1. The connectivity information of each node (peer, orderer, fabric-ca)
2. A channel and the nodes that are participating in that channel
3. The role of the peer in that channel
4. The Organisations that own the various nodes (peers and fabric-ca servers, but not orderers)
5. A client section that describes information specific to a single organisation.

The Hyperledger Composer documentation provides quite a lot of detail about connection profiles and how to create a connection profile to define your specific network 

- [Connection Profile Reference](https://hyperledger.github.io/composer/latest/reference/connectionprofile)
- [Single Org Tutorial](https://hyperledger.github.io/composer/latest/tutorials/deploy-to-fabric-single-org)
- [Multi Org Tutorial](https://hyperledger.github.io/composer/latest/tutorials/deploy-to-fabric-multi-org)

What will be covered here are some more important aspects of the operational side not covered by the documentation.

## Who owns the specification and implementation of the connection profile ?
It's NOT Hyperledger Composer. In fact it is a specification that has come from Hyperledger Fabric (with input from the Composer team) and the implementation for parsing it is available in the fabric node sdk.

## Portability of business network cards
The philisophy behind business network cards is that they can be shared. However unless you get the connection profile right, this will not be possible. For example there is no point giving a business network card which contains a connection profile with URLs that contain things like `localhost`, `127.0.0.1` or docker names that can only be resolved by your local docker fabric setup. In the real world, a Hyperledger Fabric network is not going to be running all the nodes on your local development machine, it is going to be running each node on a different machine (be it a set of VMs, Docker Containers, Physical Machines) in some remote location. So the first thing to do when building your connection profile for a Hyperledger Fabric network is to ensure that you use real ip addresses or real resolvable hostnames. You would only use `localhost` when you are running a development fabric for your own use on your own machine. There are known issues with using `localhost` on remote VMs such as AWS even if you are using it as a single development server.

Secondly, if you are using TLS to communicate with your fabric nodes, you should use the format

```
"tlsCACerts": {
    "pem": "-----BEGIN CERTIFICATE ... -----END CERTIFICATE-----"
}
```  

to specifiy the tls certificate, and not the `path` option. Paths are going to be specific to your machine and not someone elses unless they set up the exact same directory structure and are passed the certificate files to store there. 

Currently using the `pem` option is the only safe way to ensure your connection profile is portable. Getting a pem file into the a JSON document does have it's issues. An easy command to convert a pem file to the format you can just paste into a JSON document is

```
awk 'NF {sub(/\r/, ""); printf "%s\\n",$0;}' YOUR_PEM_FILE > text_to_paste.txt
```

will write a file called text_to_paste.txt which will contain the correct format for a JSON document.

## Connection Profiles can be in JSON or YAML format.
Not documented on the Composer website, but the `composer card create` will allow for a connection profile to be specified in `json` or `yaml` format. Hyperledger Fabric document the YAML format whereas the Composer documentation document the JSON format. 

YAML makes this easier to handle the contents of a PEM file so you may prefer to use the YAML format instead.

## Peer roles
Here is an example of defining the roles a peer has in a channel

```
"channels": {
    "mychannel": {
        "orderers": [
            "orderer.example.com"
        ],
        "peers": {
            "peer0.org1.example.com": {
                "endorsingPeer": true,
                "chaincodeQuery": true,
                "eventSource": true
            },
            ...
```

By default if you don't specify a role then the role is assumed to be true. In the above example the role `ledgerQuery` is not defined so this is assumed to be true.

For a peer to have the role of `endorsingPeer` or `chaincodeQuery`, a business network must have been installed on that peer.

What makes this feature useful is the ability for Organisations to decide which peers they send endorsements to, qhich peers they want to query and which peers to use as an event source. A Peer may be capable of all those roles, but from a client interaction perspective you may wish to be selective about which peers you interact with (more details later about how clients connect and interact with nodes)

## Channels
The connection profile specification allows for the definition of multiple channels in the connection profile. Composer does not support multiple channels. You should only ever define a single channel in your connection profile.

## Other connection profile properties

A bit more detail about some of the other properties in the connection profile. 

### x-type
This doesn't refer to any specific fabric version. This is a property used exclusively by Hyperledger Composer to determine the node module to load in order to interact with Hyperledger Fabric. You should always follow the guidance of the documentation based on the version of composer you are using. For example the `composer-connector-hlfv1` node module handles connectivity to hyperledger fabric 1.1 (it could in the future handle connectivity to 1.1 and 1.2) but it's referenced by the `hlfv1` x-type.

### version
This is the version of the connection profile document. Currently the only supported version is `1.0`. You can also specify `1.0.0` or even `1.0.0-something`. Only the Major and Minor numbers a looked at.

### x-committimeout
This is another property used exclusively by Hyperledger Composer. Any property starting with `x-` is seen as a application specific extension. This is the timeout in seconds that Composer will wait for a transaction to commit before returning control back to the calling client program by throwing an error.

### grpcOptions

### httpOptions

### x-hsm
This is another Hyperledger Composer specific extension but will be discussed in the [HSM Chapter](./hsm.md)

### [Next - Business Network Cards](./busnetcards.md)
