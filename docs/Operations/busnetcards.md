### [TOC](./TOC.md)
### [Back - Connection Profiles](./connectionprofiles.md)

# Business network cards
(insert diagram)

TBD
- Amalgomation of 2 things
  - Connection Profile
  - either
     - crypto material + username (only used as a label)
     - username + secret (both used to retrieve information from fabric ca server)
  - name of the business network (optional)
  - roles (optional)


## Card Files vs Cards
Discussion and documentation around business network cards use the term card for 2 different things so it's time to be clear. A card lives in a card store, A card file is a `COPY` of a card put into a file. A card file allows you to transport copies of cards to other systems outside of using a shared card store (to be discussed later). 


## Secret Cards and Card Files
There are 2 types of Card

- One that has a certificate and optionally a private key (you won't have a private key if your card is able to interact with an HSM. HSMs are covered later in this book)
- One that has only a secret stored in the metadata of the card.

### So why is it important to know about the difference ?
(Insert diagram here)
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

### Business Network Cards that don't represent a business network and roles
**TBD**

A Business network card can have roles associated with it, you will see the option in `composer card create` to apply roles to cards. You only need to apply roles to cards if you plan to use the cards in `Composer Playground`. Also you only can add roles to cards that do not define a business network. These are the special cards you would create as described above. 

The following roles are available

| Role | Description |
| ---- | ------------|
| PeerAdmin | You specify this role if the card is to be used to install business networks on a Peer and thus contains the Peer Admin credentials |
| ChannelAdmin | You specify this role if the card is to be used to start or upgrade a business network on a Channel and thus contains the Channel Admin credentials |

In both the case of the composer sample fabric server and fabric samples themselves the channel admin's are usually the admins for the peers in the network.



### further reading
TBD: The fabric-ca-server bootstrap id is a different id that needs to be covered 

### [Next - NetworkAdmin Participant Type](./networkadmin.md)
