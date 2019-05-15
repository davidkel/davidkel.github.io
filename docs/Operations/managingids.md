### [TOC](./TOC.md)

### [Back - ACLs for Operations](./acls.md)

# Managing Id's and Participants
This section is about how to manage your identities, participants and the association between them, but let's recap first on some things.
First, you will have a business network deployed and you will have at least 1 business network card that has an identity which is bound to some sort of participant that has authority to perform activities such as Participant Creation, identity binding etc (as per the previous section on ACLs). 
Secondly, you will recall that 1 or more identities can be bound to a participant, but you can't have an identity bound to more than 1 participant, whenever an identity is presented to the Hyperledger Composer runtime, it looks up the participant associated with that identity.
Thirdly, an identity is represented by a public certificate and associated private key, it is not represented by a username and secret. Usernames and secrets are specific to the fabric ca to request a **NEW** public certificate and this represents a new identity (ie if you are able to request the identity multiple times you get different identities with the same name).

There are some scenarios here to consider
1. You want to create a new identity and associate that identity with an existing participant
2. You want to create a new identity and a new participant to associate it with
3. You have an existing identity (not associated with a participant) and want to create a new participant to associate it with.

You cannot create a new participant and try to associate it with an identity that is already associated with another participant.


## Creating participants
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
Before you can register a new identity you will have had to enroll an issuer (such as the fabric ca bootstrap identity). If you haven't already done so, then follow the next section about enrolling an identity (and use the -H option to specify an appropriate place to store the issuer enrollment information and make sure you backup this enrollment directory).

```
fabric-ca-client register --id.affiliation org1 --id.type user --id.name <USER_NAME> --tls.certfiles <FABRIC_CA_TLS_CA_CERT> -u https://admin:adminpw@ca.example.com -H <ENROLLED_ISSUER_DIR>
```
If your fabric ca server is using tls then you need to specify `https` and include the `--tls.certfiles` option specifing the absolute path (anything else forces it to start in the $HOME/.fabric-ca-client directory as the base directory) to the `pem` file of the tls certificate

for example if you have enrolled an issuer called admin and stored results of that enrollment in the directory `~/admin`, then to register a new user might look like

```
./fabric-ca-client register --id.name dave -u http://admin:adminpw@localhost:7054 -H ~/admin
2018/05/04 15:12:48 [INFO] Configuration file location: /home/vagrant/admin/fabric-ca-client-config.yaml
Password: LMnhcqziWBdz
```

### enrolling identities from the fabric ca server
The basic format of this command is as follows.

```
fabric-ca-client enroll --tls.certfiles <FABRIC_CA_TLS_CA_CERT> -u https://<USER_NAME>:<USER_SECRET>@ca.example.com
```

If your fabric ca server is using tls then you need to specify `https` and include the `--tls.certfiles` option specifing the absolute path (anything else forces it to start in the $HOME/.fabric-ca-client directory as the base directory) to the `pem` file of the tls certificate. For example
```
fabric-ca-client enroll --tls.certfiles ~/ca/ca-cert.pem -u https://dave:LMnhcqziWBdz@ca.example.com
```

Non TLS comms would just be
```
fabric-ca-client enroll -u http://dave:LMnhcqziWBdzpw@ca.example.com
```

Both the above enroll the user `admin` who has a secret of `adminpw`.

The output is written to the $FABRIC_CA_CLIENT_HOME directory (`.fabric-ca-client` in your HOME directory if not set). The msp directory contains the certificates and private key.

```
ls ~/.fabric-ca-client/msp
cacerts  intermediatecerts  keystore  signcerts
```
you will find the public certificate file `cert.pem` in the `signcerts` directory. You will find the private key which is a file ending in `_sk` in the `keystore directory`

You can either set the `FABRIC_CA_CLIENT_HOME` environment variable or use the `-H` option (which is a better solution)

```
fabric-ca-client enroll -u http://dave:LMnhcqziWBdz@ca.example.com -H ~/dave
```
#### composer identity request
This command isn't documented.... Reason for this was that it was a quick temporary solution to a specific problem at the time where it was required to compile the fabric-ca-client yourself. Now that pre-built binaries of fabric-ca-client are available the plan was to get rid of this command. It still exists for now because the BYFN tutorial on the Hyperledger Composer web site has to use it because it can get over problems using TLS certs as the fabric ca server in that tutorial is TLS enabled. 

`composer identity request` can use TLS but can told in the connection profile of the business network card used to not verify the TLS certificate. The TLS certificate used by the CA's have names in their certs such as ca.org1.example.com yet to make the request you have to call `localhost` (as your machine is unlikely to point ca.org1.example.com to your machine). fabric-ca-client doesn't allow for a hostname override or don't verify the certificate so in the tutorial you cannot use fabric-ca-client as it won't allow the connection due to TLS verification failure, but this would only be a problem in a development environment and you should never not verify the TLS certificate of perform a hostname override in a production environment.

`composer identity request` should only be used for development purposes only and expect it to disappear as soon as fabric-ca-client can be substituted into the BYFN tutorial. 

Always use fabric-ca-client for production and non local development testing.

## Creating your business network card for new identities
Please refer back to [Business network cards](./busnetcards.md) for more details about business network cards, but as a reminder, there are 2 types of business network card you will need to create

1. A business network card that provides fabric level access (used for install/start/upgrade), which means it isn't really a business network card :-)
2. A business network card that provides access to the business network

An example of creating a business network card file (ready for importing into a card store) for a fabric level access could be

```
composer card create -p network-org1.json -u caadmin -c ~/.fabric-ca-client/msp/signcerts/cert.pem -k ~/.fabric-ca-client/msp/keystore/58714e0fc9268bc6ae70c0ac357c52a2f2866742081966f8cf4f8df0ee9df21f_sk -r ChannelAdmin -r PeerAdmin
```

Another example of creating a business network card file for accessing a business network called financial-network could be
```
composer card create -p network-org1.json -u fred@org1.example.com -c ~/.fabric-ca-client/msp/signcerts/cert.pem -k ~/.fabric-ca-client/msp/keystore/58714e0fc9268bc6ae70c0ac357c52a2f2866742081966f8cf4f8df0ee9df21f_sk -n financial-network
```

It is perfectly possible to create a business network card with the secret returned from registering the user but I highly recommend you never create a business network card with a secret in. I'm sure I've mentioned this before :-)

# Associating an identity with a participant
Once you have the identity, you can then bind (associate) it to a participant using the command `composer identity bind`. 
An example could be
```
composer identity bind -c admin@digitalproperty-network -a net.biz.digitalPropertyNetwork.Person#P1 -e dave@org1.com.pem
```
You specify the participant using it's fully qualified name (-a) and the file containing the public certificate for the identity (-e).

This registers the identity in the Composer identity registry and is marked as `BOUND`. It still needs to be activated and that will happen on the first interaction where that identity is used and is done automatically.

## Composer identity issue
TBD: Information about Composer Identity Issue

```
composer identity issue -c admin@digitalproperty-network -u JDaniels -a net.biz.digitalPropertyNetwork.Person#P1
```

- Identity issue and the 2 priviledges it requires, plus must be able to write the card otherwise you end up with a indetermined state.
- single enrollment only, can never be enrolled again.
- creates a card with a secret in :-(

## listing existing identities and participants
There are 2 commands here that provide information here
- composer identity list
- composer network list

### composer identity list
This is an invaluable command for finding out what identities have been registered to the composer runtime, their identifiers, state (ISSUED, BOUND, ACTIVATED, REVOKED) and what participants they are mapped to. see [The Hyperledger Composer Security Model](./idsandparts.md) for more information.

An example of the command is as follows. You must use a card that has the appropriate permissions see [ACLs](./acls.md) for more information.

```
composer identity list -c admin@digitalproperty-network
✔ List all identities in the business network
-
  $class:      org.hyperledger.composer.system.Identity
  identityId:  60f17aae92d098026db2d3a63ebce520bf99195128d128f8ca9a0d051ac3a649
  name:        admin
  issuer:      ac3dbcbe135ba48b29f97665bb103f8260c38d3872473e584314392797c595f3
  certificate:
    """
      -----BEGIN CERTIFICATE-----
      MIICAjCCAaigAwIBAgIUM/tU3oi+i6zxFGVQWmB3bN/ha2UwCgYIKoZIzj0EAwIw
      czELMAkGA1UEBhMCVVMxEzARBgNVBAgTCkNhbGlmb3JuaWExFjAUBgNVBAcTDVNh
      biBGcmFuY2lzY28xGTAXBgNVBAoTEG9yZzEuZXhhbXBsZS5jb20xHDAaBgNVBAMT
      E2NhLm9yZzEuZXhhbXBsZS5jb20wHhcNMTgwNTAyMTUwMzAwWhcNMTkwNTAyMTUw
      ODAwWjAhMQ8wDQYDVQQLEwZjbGllbnQxDjAMBgNVBAMTBWFkbWluMFkwEwYHKoZI
      zj0CAQYIKoZIzj0DAQcDQgAEUK5cxo+ENFQj06Cyd4VS1d5OASK1isV+rA4IliyO
      1JF2st8X0kRrX/4hU0qNoVOQb9gC7RfS9qUHauiONOdl8aNsMGowDgYDVR0PAQH/
      BAQDAgeAMAwGA1UdEwEB/wQCMAAwHQYDVR0OBBYEFPnv7LH/2JaC7b3n3vxinF16
      whspMCsGA1UdIwQkMCKAIBmrZau7BIB9rRLkwKmqpmSecIaOOr0CF6Mi2J5H4aau
      MAoGCCqGSM49BAMCA0gAMEUCIQCyI20lw2POgyLbnirj4ddhERyN2PcafJiraIao
      WzudagIgKFs1s36f8kjogzhg8P60efd3Z/s21TaI/cu4LMsd2xk=
      -----END CERTIFICATE-----

    """
  state:       ACTIVATED
  participant: resource:org.hyperledger.composer.system.NetworkAdmin#admin
-
  $class:      org.hyperledger.composer.system.Identity
  identityId:  99c6d22ca6323fd40d8b206abf2a54a5191ffc8102d4a64c568ed845d37d9f52
  name:        JDaniels
  issuer:      ac3dbcbe135ba48b29f97665bb103f8260c38d3872473e584314392797c595f3
  certificate:
  state:       ISSUED
  participant: resource:net.biz.digitalPropertyNetwork.Person#P1

Command succeeded
```
In the above we see an activated identity with a name of `admin` and it's associated public certificate in PEM format and it is mapped to the participant `resource:org.hyperledger.composer.system.NetworkAdmin#admin`.

There is also an issued identity called `JDaniels`, and because it is issued, there is no associated certificate as yet. It is mapped to the participant `resource:net.biz.digitalPropertyNetwork.Person#P1`.


### composer network list
You use this command to get a bit more information about participants in the network.

Here is an example of listing the participant registry for `net.biz.digitalPropertyNetwork.Person` to see what participants of this type have been defined. 

```
composer network list -c admin@digitalproperty-network -r net.biz.digitalPropertyNetwork.Person
✔ List business network from card admin@digitalproperty-network
models:
  - org.hyperledger.composer.system
  - net.biz.digitalPropertyNetwork
scripts:
  - lib/DigitalLandTitle.js
registries:
  net.biz.digitalPropertyNetwork.Person:
    id:           net.biz.digitalPropertyNetwork.Person
    name:         Participant registry for net.biz.digitalPropertyNetwork.Person
    registryType: Participant
    assets:
      P1:
        $class:    net.biz.digitalPropertyNetwork.Person
        personId:  P1
        firstName: J
        lastName:  Daniels

Command succeeded
```
This example lists the participant registry for the inbuilt participant type `org.hyperledger.composer.system.NetworkAdmin` to see what participants of this type have been defined. 


```
composer network list -c admin@digitalproperty-network -r org.hyperledger.composer.system.NetworkAdmin
✔ List business network from card admin@digitalproperty-network
models:
  - org.hyperledger.composer.system
  - net.biz.digitalPropertyNetwork
scripts:
  - lib/DigitalLandTitle.js
registries:
  org.hyperledger.composer.system.NetworkAdmin:
    id:           org.hyperledger.composer.system.NetworkAdmin
    name:         Participant registry for org.hyperledger.composer.system.NetworkAdmin
    registryType: Participant
    assets:
      admin:
        $class:        org.hyperledger.composer.system.NetworkAdmin
        participantId: admin

Command succeeded
```

## Revoking identities
To revoke an identity, you need to know it's identifyId (also known as the identifier). Although the command may reference -u or --userId it doesn't use a username. This means you have to use the `composer identity list` command to determine the identifier from the `identityId` property of the identity you wish to revoke. In the previous example for `composer identity list`, `admin` has an identity id of `60f17aae92d098026db2d3a63ebce520bf99195128d128f8ca9a0d051ac3a649`, so if I wanted to revoke that identity I would issue

```
composer identity revoke -c NewAdmin@digitalproperty-network --identityId 60f17aae92d098026db2d3a63ebce520bf99195128d128f8ca9a0d051ac3a649
```

As talked about previously, when you revoke an identity, you revoke the public key of that identity. This means you cannot generate new certificates from the private key of that identity and use that. You will have to create a new private/public key for that identity.

An important aspect. When you revoke this identity you revoke their access to the business network **NOT** the fabric network (Fabric will still allow them to perform fabric level activity), or the fabric-ca (for example if they have issuer authority). You would need to look at what is required to revoke them from the fabric network itself as well as the fabric-ca.

Finally, be careful. You can revoke the identity you make the request with. So if you have have only 1 business network administrator and you revoke it you will lock yourself out of performing administrative access on that business network.

It would be possible to recover from this so long as you have access to an identity with a bound participant you could install and upgrade to a new business network with updated ACLs to give that participant appropriate abilities, so long as you have not revoked your only fabric admin identity at the fabric level of course. 

## updating and removing participants
you cannot update or remove a participant from the CLI, you will either have to write a client application using the composer-sdk or via the rest server or use the rest server explorer UI (if you haven't disabled it)

## What to do before an identity expires or if it has expired
TBD: What to do around certificate expiration

## Common problems
TBD: Common problems
- authorization failure
- identity not registered

## Recommendations
This is my list of recommendations
- Always using certs, not secrets. Secrets may seem quicker but are more prone to problems which could leave you with identities that cannot be used and having to start the process again.
- avoid using `composer identity issue` it's great for development purposes but in production where you want role separation and if the command fails you may not get a useable card. Plus that card has a secret in it (see above). You also need to have an identity that is both an issuer and has some sort of business network identity management authority (see next recommendation)
- avoid using the bootstrap identity for the fabric ca as the identity you bind on a `composer network start`. In fact try to avoid using issuers as identities for business networks. Try to keep the roles separate. The only reason to have an issuer identity bound to a participant is to be able to use `composer identity issue`.
 

## Step by step example of deploying a business network, creating an identity and participant for that business network.
TBD: Step by step
1. enroll the issuer (let's say it's admin)
2. register a bnAdmin identity
3. enroll that bnAdmin identity
4. perform a start with that identity
5. create a card file for that identity
6. import the card
7. register a participant identity
8. enroll that identity
9. create a participant which exists in the business network
10. bind the identity to the participant
11. create a card file for that identity
12. import the card file
13. ping the card to prove it all works

### Adding a new network admin participant
It's possible to add another NetworkAdmin participant if you are using that participant for your business network, refer to [ACLs](./acls.md) for more details about defining permissions for participants.

repeat steps 7-13 but create a NetworkAdmin participant in step 9 as follows
```
composer participant add -c admin@trade-network -d '{"$class":"org.hyperledger.composer.system.NetworkAdmin", "participantId":"restadmin"}'
```