### [TOC](./TOC.md)

### [Back - Deploying Business Networks](./deploy.md)


# Upgrading business networks
Deploying a new release of a business network follows a similar pattern to deploying your initial release of a business network

- install the new business network onto the peers
- request upgrade to upgrade the currently running business network to the new version.

TBD: Add information again about business network cards

Upgrading, similar to start, is something that has to be agreed across all participanting organisations. So each organisation must first install the new version of the business network onto their peers using the `composer network install` command. Once all peers in the network are ready and agreement has been reached then the `composer network upgrade` can be used to upgrade the business network. 

You can actually use this process to downgrade to a previous version of the business network as well just be selecting an older version number (so long as that version is still installed on the peers of course)

## Endorsement policy
You can also change the endorsement policy at this time and the `upgrade` command uses the same command line options as `start` for defining the endorsement policy.

If you just wanted to change the endorsement policy then you can use this process. You would just change the version number of your business network without any other changes and install this version onto the peers. Then you would use the `upgrade` command to upgrade to this version but specify the new endorsement policy.

## Chaincode containers
Every time you upgrade, fabric follows the same process [here](./deploy.md#fabric-1.1-node.js-chaincode) for building and starting a chaincode container. This means upgrade can take some time to complete. There is one other thing that is not obvious. **Hyperledger Fabric leaves the old chaincode containers running**. The reason is that at the moment they don't know if those chaincode containers could still be being used by a different channel to the one you have just done an upgrade on (remember a business network is deployed to a channel, but fabric can share chaincode containers across channels as chaincode containers should be stateless). 
If you do lots and lots of upgrades (especially in a development environment) you will have lots and lots of chaincode containers running. One of the common errors when this causes a problem is the error message

```
Error: Error trying to upgrade business network. 
Error: No valid responses from any peers. Response from attempted peer comms was an error: 
Error: 8 RESOURCE_EXHAUSTED: received trailing metadata size exceeds limit Command failed
```

As the administrator of your hyperledger fabric network, only you can decide if these chaincode containers can be quiesced.

## Composer Runtime version changes at upgrade
There is the likelyhood that when you perform an upgrade you might also change the version of the composer runtime that get's referenced. The `composer network install` command as described in the [Under the covers](./deploy.md#under-the-covers) will insert an appropriate reference to the composer runtime into the package.json if they aren't explicitly there. If you have changed versions of the `composer-cli` you are using then it will put the version of that command being used in. If you don't want this then you should fix the version you are currently using in the package.json file of the business network yourself. You can always find the version number of the runtime by doing a `composer network ping`

```
The connection to the network was successfully tested: basic-sample-network
	Business network version: 0.1.14
	Composer runtime version: 0.19.1   <--------
	participant: org.hyperledger.composer.system.NetworkAdmin#admin
	identity: org.hyperledger.composer.system.Identity#b4b1df68b1f66a76357b4dd12c2ad1c49770cb4c553d94e36cbc1e08713ac9ef
```

If you allow the runtime to also be upgraded, then there is a check in the to reject a version change if it isn't a micro version change. So for example you can upgrade from 0.19.0 to 0.19.2, but you cannot upgrade from 0.19.0 to 0.20.0. A minor version change indicates a possible backward incompatibility and so this upgrade is disallowed.

You can also use the upgrade process to just upgrade the runtime and not change the business network but you will still have to change the version number of the business network in package.json, install the new business network on all the peers then perform an upgrade.

### [Next - Managing identities and participants](./managingids.md)
