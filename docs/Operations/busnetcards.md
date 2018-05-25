### [TOC](./TOC.md)
### [Back - Connection Profiles](./connectionprofiles.md)

# Business network cards
TBD:(insert diagram)


Business network cards comprise of the following bits of information
  - Connection Profile
  - Credential information which can be in 2 forms
     -  username (only used as a label)+ x509 certificate + optionally a private key (optional if you are using HSM)
     - username + secret (both used to retrieve x509 certificate from fabric ca server)
  - [Name of the business network](./busnetcards.md#name-of-the-business-network) (optional)
  - [roles](./busnetcards.md#roles-in-a-business-network-card) (optional)

## Card Files vs Cards
Discussion and documentation around business network cards use the term card for 2 different things so it's time to be clear. A card lives in a card store, A card file is a `COPY` of a card put into a file. A card file allows you to transport copies of cards to other systems outside of using a shared card store (to be discussed later).

### Card Stores
To use a card it must exist in the card store. If you have a card file then you must import it into the card store before you can use that card. 
Hyperledger Composer is flexible in that it can work with different types of card store implementations, for example by default the card store is defined in the file system at `${HOME}/.composer/cards`. See the [Cloud Wallets](./cloud-wallets.md) section for more details about how hyperledger composer persists important information such as cards and credential information as well as how to configure different persistence facilities.

#### Import/Export/Deletion of Cards and Card Files
(TBD: Diagram required here)
This is important to note, whenever you use the composer CLI or AdminConnection API to import a card file, export a card or delete a card, more is actually going on than just storing the card in the card store. Depending on the `x-type` specified in the connection profile (see [here](./connectionprofiles.md#x-type) for more info) a connector defined by the `x-type` may also need to perform some sort or import/export/delete type activity. So the recommendation is to always use the CLI or AdminConnection api's to perform these operations rather than for example trying to do it yourself as you know where the card store is located. More information about this is in the [Cloud Wallets](./cloud-wallets.md) section.

## Connection Profiles
These are covered in detail [here](./connectionprofiles.md) but in summary a connection profile provides a description of your hyperledger fabric network, including
1. The physical components, ie peers, orderers, fabric cas, their urls and tls certificates required to connect
2. The logical components, ie channels, timeouts you want to apply, organisations and the organisation you are a member of

## Credential information stored in a card
There are 2 types of credential information that a card could hold

- One that has a certificate and optionally a private key (you won't have a private key if your card is able to interact with an HSM. HSMs are covered later in this book)
- One that has only a secret stored in the metadata of the card.

But let me make a recommendation now, **Don't use Cards with Secrets for production**. Cards with secrets cause more confusion about how security works than anything else around the Hyperledger Composer security model. Avoiding using secrets may seem impossible, but as you read through this book I hope that the alternatives will become more clear.

### So why is it important to know about the difference ?
TBD:(Insert diagram here)
A card that only has a secret means it **CANNOT** interact with a business network. As you recall in order to be able to access a Hyperledger Fabric network, even before you try to interact with a business network you require a public certificate and a private key. However Hyperledger Composer  deals with this on your behalf. It will take the username and secret in that card, create a private key, register the username with the fabric-ca-server defined in the connection profile, and then request a public certificate from the fabric-ca-server. The fabric-ca-server will also record that username and explicitly reject any further requests to request a certificate for that user again. It reports this as an `Authorization Failure`. 
Now the card has been converted to one that contains a certificate and private key and that card can interact with a business network. This is all done under the covers.

### So What's the problem then ?
Well if you had a `Card File` with just a secret in it, you can **ONLY** import it once. If you imported it again somewhere is (ie into a different card store), whoever uses it first will get the certificates and private key and the other user with just get `Authorization Failure`. This is why we refer to these cards as `Single Use Only` cards

Another important thing to remember is that if you have imported a card which only has a secret, but you have never used the card, if you export it you will get a Card File that still only has a secret in it.

### What should I do about it ?
The recommended practice is

1. Import the card file with a secret into the card store
2. Delete that card file from your file system
3. Ping the card to register and load the certificate/private key
4. Now it is safe to export the card and share it as required.

## Name of the Business Network
This identifies the specific business network that this card can be used with. This implies that the card's identity is registered to the business network runtime and is bound to a participant.

### Business network cards without a business network defined in them
It is possible to have a business network card without a business network defined in it, which kind of makes the name of these cards strange... But you can and what does this card represent ? Composer wanted to use these cards for being able to perform fabric level operations such as installing a business network, starting a business network and upgrade a business network. These actions are not something the business network itself get's involved with directly but are actions upon them controlled by hyperledger fabric itself. For more information about this the [Deploy](./deploy.md) and [Upgrade](./upgrade.md) sections of this book provide more detail.

These `special` cards (which aren't business network cards) must not contain a business network name. If you do then they will not work. If you have followed the tutorials for hyperledger composer you will have encountered these types of cards already.

## Roles in a business network card
A Business network card can have roles associated with it, you will see the option in `composer card create` to apply roles to cards. You only need to apply roles to cards if you plan to use the cards in `Composer Playground`. Also you only can add roles to cards that do not define a business network. These are the special cards you would create as described above. 

The following roles are available

| Role | Description |
| ---- | ------------|
| PeerAdmin | You specify this role if the card is to be used to install business networks on a Peer and thus contains the Peer Admin credentials |
| ChannelAdmin | You specify this role if the card is to be used to start or upgrade a business network on a Channel and thus contains the Channel Admin credentials |

In both the case of the composer sample fabric server and fabric samples themselves the channel admin's are usually the admins for the peers in the network.

## Composer CLI commands for managing business network cards
The Composer CLI provides card and card file management commands
| Command | Description |
| ------- | ------------|
| composer card create | Creates a card file which can be imported into a card store |
| composer card import | Imports a card file into a card store |
| composer card export | Exports a card to a card file |
| composer card delete | Deletes a card from a card store |
| composer card list | List all cards in the store, or list a specific card in the store |

### Listing a specific card in the card store
This is a useful command to understand a card in the card store, specifically the current credential information. For example a card that has a public certificate and private key looks like
```
$ composer card list -c admin@digitalproperty-network
userName:            admin
description:
businessNetworkName: digitalproperty-network
identityId:          f515e20ee5457ee8d58df9ece2fd1fa4aff1913f6bfead200e58c26c744d54c9
roles:               none
connectionProfile:
  name:   testfabric
  x-type: hlfv1
credentials:         Credentials set
```
The identityId is important, this has to be and will be unique. userName for example doesn't have to be unique. You could have 2 cards with the same userName but they will have different identityId values. It's this value you can use to see if the identity is registered to the business network runtime. More information can be found in the [Managing Id's and Participants](./managingids.md) section.
Also note that there are no roles defined here. That is because this is a business network card, not a `special` card to be used by Composer Playground to deploy and upgrade business networks.

Here is a card that only has a secret defined, and it's this type of card that can cause problems, as described previously

```
$ composer card list -c NewAdmin@digitalproperty-network
userName:            NewAdmin
description:
businessNetworkName: digitalproperty-network
identityId:
roles:               none
connectionProfile:
  name:   testfabric
  x-type: hlfv1
credentials:         One time use only secret set
```

Finally here is a `special` card that is used to fabric level install/start/upgrade actitivies and can also be used in Composer Playground
```
userName:            FabricAdmin
description:
businessNetworkName:
identityId:          114aab0e76bf0c78308f89efc4b8c9423e31568da0c340ca187a9b17aa9a4457
roles:
  - ChannelAdmin
  - PeerAdmin
connectionProfile:
  name:   testfabric
  x-type: hlfv1
credentials:         Credentials set
```



### Other commands that create business network cards
There are some other commands that create business network cards, for example `composer network start` and `composer identity issue`. These commands are discussed in more detail in the [Deploy](./deploy.md) and [Managing Id's and Participants](./managingids.md) sections.


### [Next - NetworkAdmin Participant Type](./networkadmin.md)
