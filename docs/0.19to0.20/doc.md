# Migrating from Composer 0.19 to Composer 0.20
As you know, Composer 0.19 is only compatible with Hyperledger Fabric 1.1 and Composer 0.20 is only compatible with Hyperledger fabric 1.2 - 1.4. Hyperledger fabric allows you to upgrade from previous versions starting at 1.1 to Hyperledger fabric 1.4 however in the past you could not upgrade Composer from 0.19 to 0.20. Starting with Composer 0.20.8, that restriction was removed as the composer runtime data structures has not changed between 0.19 and 0.20. This document provides simple guidance for upgrading your version of Composer to move with your upgrade of hyperledger fabric. It is highly recommended that you perform local testing of your applications on Composer 0.20.8 (or higher) first using a local fabric setup first.

## What problems are you likely to see if you don't upgrade
Hyperledger Fabric changed the way chaincode error messages are returned in 1.2, this required a change to the fabric node sdk as well. Composer 0.19 requires the node sdk for fabric 1.1 which cannot handle these error messages coming back from the business network and composer runtime logic so you just get the cryptic message
```
Unexpected end of JSON input
```


## Upgrading the business network

**IMPORTANT**: Once you have migrated your Composer application to 0.20.8, you cannot migrate it back to 0.19. The checks in the 0.19 version will stop these from being allowed.

- Ensure your applications have been shut down prior to performing the upgrade to hyperledger fabric 1.4
- Download the composer-cli version 0.20.8 (or higher). You should be explicit about the version so that you know exactly which versions you are working with
```
npm install composer-cli@0.20.8
```
- Open the package.json of your business network definition to update the version of your business network to a newer version number and see if any of the following lines are listed in the dependencies, ie any dependency starting with the text `composer-` (the version listed could be different) and if they do exist then you should delete them. Repackage your bna file. For example if you see lines like
```
“composer-common”: “0.19.20”,
“composer-runtime-hlfv1": “0.19.20”
```
you should delete those lines

- Perform an upgrade of your business network with your new version of the business network by first installing the new business network then performing an upgrade
1.composer network install -c (your peer admin business network card) -a (your new business network archive)
2.export ENDORSEMENT_HANDLER=’’
3.composer network upgrade -c (your channel admin business network card) -n (your business network name) --networkVersion (your new business network version) 
Step 2 is required due to a bug in the node-sdk used by composer 0.20.8 to ensure that timeouts are correctly utilised from those in your business network card.
(include any other options on these cli commands as required if youused them in previous install/start/upgrade, for example endorsement policies)

- Confirm the upgrade is successful by doing a network ping and checking the Composer runtime version is at least 0.20.8
```
Composer network ping –c (an appropriate card for the business network)
The connection to the network was successfullytested: some_card_name
Business network version: 1.17.12
Composer runtime version: 0.20.8
participant: org.hyperledger.composer.system.NetworkAdmin#admin
identity: org.hyperledger.composer.system.Identity#0b598c1abc0d595bb4d9ae94f462abf5d3a42dbd441e42f94ebc2158d6a9735c
```

## Upgrading your Composer client applications
For all client applications and this includes the the composer-cli, composer-rest-server and composer-playground, you MUST ensure the following environment variable is set and can be used by the applications
```
ENDORSEMENT_HANDLER=’’
```
This ensures that the timeout values in your cards are to be used, otherwise you cannot override the timeout values and you will default to 45 seconds for request timeouts.

If you are using node.js applications using the composer sdks then you need to change your composer dependencies of 

- composer-client
- Composer-admin

To use version 0.20.8.

If you are using the composer rest server then you need to reference version 0.20.8 of the rest server which includes whether you reference it through npm install or via docker (maybe to push the rest server to cloud foundry).