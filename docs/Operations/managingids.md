### [TOC](./TOC.md)

### [Back - ACLs for Operations](./acls.md)

# Managing Id's and Participants
This section is about how to manage your identities, participants and the association between them, but let's recap first on some things.
First, you will have a business network deployed and you will have at least 1 business network card that has an identity which is bound to some sort of participant that has authority to perform activities such as Participant Creation, identity binding etc (as per the previous section on ACLs). 
Secondly, you will recall that 1 or more identities can be bound to a participant, but you can't have an identity bound to more than 1 participant, whenever an identity is presented to the Hyperledger Composer runtime, it looks up the participant associated with that identity.
Thirdly, an identity is represented by a public certificate and associated private key, it is not represented by a username and secret. Usernames and secrets are specific to the fabric ca to request a **NEW** public certificate and this represents a new identity (ie if you are able to request the identity multiple times you get different identities with the same name).

There are 2 scenarios here to consider
1. You want to create a new identity and associate that identity with an existing participant
2. You want to create a new identity and a new participant to associate it with

You cannot create a new participant and try to associate it with an identity that is already associated with another participant.


## creating Participants
There are many ways you can create a participant. You could use playground (not recommended for production), or you could use the rest server explorer UI (although in production I would be looking to remove this UI myself). The alternatives are to write a client application using either the composer-sdk or by utilising the REST api of the business network or to use the `composer participant add` command. 
An example of this command is as follows

```
composer participant add -c admin@digitalproperty-network -d '{"$class":"net.biz.digitalPropertyNetwork.Person","personId":"P1", "firstName":"J", "lastName":"Daniels"}'
```
As described earlier you need to use a business network card that has the appropriate authority to add a participant. The JSON string must be enclosed in single quotes and the format must match the model definition of the participant you have declared in the `$class` property.

## Creating identities
To create an identity you can host and run a fabric ca server in your network or you could make use of a third party certificate authority to provide the service for you. You need to ensure that either of these services can create identities that will be recognised as being in the correct MSP for your organisation.

Some of the following information is about using an appropriately configured Fabric CA Server for obtaining identities, you can always obtain an identity (public certificate and private key) from another source and most of the following is still applicable.

This is not a guide on how to interact with the fabric ca server, I would recommend reading http://hyperledger-fabric-ca.readthedocs.io/en/latest/

and specifically about the fabric-ca-client CLI tool here
http://hyperledger-fabric-ca.readthedocs.io/en/latest/users-guide.html#fabric-ca-client

This section is about how to use composer rather than the fabric-ca-client but provides some basic information as it will be necessary to make use of it, so for example it doesn't discuss revoking identities on the fabric-ca-server or how that should be managed with the CRL list in a fabric network.

### Registering identities in the fabric ca server
```
fabric-ca-client register --id.affiliation org1 --id.type user --id.name <USER_NAME> --tls.certfiles <FABRIC_CA_TLS_CA_CERT> -u https://admin:adminpw@ca.example.com
```
TBD

### enrolling identities from the fabric ca server
```
fabric-ca-client enroll --tls.certfiles <FABRIC_CA_TLS_CA_CERT> -u https://<USER_NAME>:<USER_SECRET>@ca.example.com
```
TBD

#### composer identity request
This command isn't documented.... Reason for this was that it was a quick temporary solution to a specific problem at the time where it was required to compile the fabric-ca-client yourself. Now that pre-built binaries of fabric-ca-client are available the plan was to get rid of this command. It still exists for now because the BYFN tutorial on the Hyperledger Composer web site has to use it because it can get over problems using TLS certs as the fabric ca server in that tutorial is TLS enabled. 

`composer identity request` can use TLS but can told in the connection profile of the business network card used to not verify the TLS certificate. The TLS certificate used by the CA's have names in their certs such as ca.org1.example.com yet to make the request you have to call `localhost` (as your machine is unlikely to point ca.org1.example.com to your machine). fabric-ca-client doesn't allow for a hostname override or don't verify the certificate so in the tutorial you cannot use fabric-ca-client as it won't allow the connection due to TLS verification failure, but this would only be a problem in a development environment and you should never not verify the TLS certificate of perform a hostname override in a production environment. 

`composer identity request` should only be used for development purposes only and expect it to disappear as soon as fabric-ca-client can be substituted into the BYFN tutorial. 

Always use fabric-ca-client for production and non local development testing.

## Creating your business network card for new identities
TBD

## associating an identity with a participant
Once you have the identity, you can then bind (associate) it to a participant using the command `composer identity bind`. For example
```
composer identity bind -c admin@digitalproperty-network -a net.biz.digitalPropertyNetwork.Person#P1 -e dave@org1.com.pem
```
You specify the participant using it's fully qualified name (-a) and the file containing the public certificate for the identity (-e).

This registers the identity in the Composer identity registry and is marked as `BOUND`. It still needs to be activated and that will happen on the first interaction where that identity is used and is done automatically.

## Composer identity issue
TBD
- Identity issue and the 2 priviledges it requires
- single enrollment only, can never be enrolled again.

## listing existing identities and participants
- composer identity list


## Revoking identities
- composer identity revoke (needs composer identity list)
Stops access to the business network, not the fabric.
- what revoke actually does

## updating and removing participants
- cannot update or remove a participant from the CLI

## Common problems
- authorization failure
- identity not registered

## Recommendations
- recommend always using certs, not secrets



### [Next - Cloud Wallets](./cloud-wallets.md)
