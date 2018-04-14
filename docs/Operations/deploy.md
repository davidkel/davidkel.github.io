### [TOC](./TOC.md)
### [Back - Fabric CA](./fabric-ca.md)


# Deploying business networks to Hyperledger Fabric 1.1
Starting in 0.19 of Composer, there was a fundamental shift wrt to deploying a business network. Prior to 0.19. The Composer runtime was the chaincode and your business network was a set of data files passed to this runtime and stored in the world state. Updating the business network was managed through an update command that run as a standard Composer transaction and was managed by ACLs. This approach actually had several limitations

1. A business network is something that is shared across organisations, yet any organisation could change the business network without getting the business network signed and having it updated.
2. Updating a business network was managed by ACLs. If you got the ACLs wrong you could lock yourself out of your business network permanently.
3. There would be no way for Composer transaction functions to ever be able to use external 3rd party node.js libraries.
4. It just didn't make sense, the business network is more than just a set of config files and simple scripts. It defines the whole application, not the runtime.

For this reason in 0.19 and beyond, the stanza is this. **Your business network IS the hyperledger fabric chaincode.** What this this mean ?

1. The business network is now the first class citizen. It controls all the dependencies
2. The business network is now deployed in the same way as all other hyperledger fabric chaincode. This means in theory that you don't have to use composer commands to install, start or upgrade a business network. It should be perfectly possible to use the other fabric capabilities such as the SDKs, Peer Commands or other providers facilities to deploy chaincode. In this respect you would not create a business network archive or use a business network archive, but just the indidivual files in a directory similar to a standard node.js package.

deploying business networks requires 2 steps

1. Each organisation need to install the same version of business network onto all endorsing and chaincode querying peers that they own (see connection profiles for what peers belong to which organisation and their roles in the channel).
2. Once all organisations have done this, a trusted 3rd party or agreed representitive from one of the organisations needs to start that business network.

## Fabric 1.1 Node.js Chaincode
An aspect of the Node.js support in Fabric 1.1 that is often not realised is that when it is instantiated (`start` is the terminology used by Composer) the following process takes place

1. fabric creates a Container from the ccenv image
2. It copies the unpacked bna file into this image and runs `npm install --production`
3. It extracts the final result of that and builds a chaincode docker image from baseImage with the final ready to use bna
4. It creates a chaincode container from this image
5. It starts the business network using `npm start`

## Using the Composer CLI commands
Composer provides 2 commands to assist in deploying. We used to provide a single command for this purpose (`deploy`) but it would not be flexible enough to handle the needs of a fabric network, for example it could not handle 2 different organisations need to install the business network, or if the admin ids for installing chaincode onto peers were different from the admin id needed to start a business network on a channel. For this reason `deploy` was dropped leaving the 2 commands to perform the different stages of getting a business network up and running.

### composer network install

examples
```
composer network install -c FabricAdmin@consortium -a b2b-network.bna

composer network install -c FabricAdmin@consortium -a b2b-network.bna -o npmrcFile=/tmp/npmConfig.txt
```

This command will install a business network onto all the peers listed in the channel that are part of your organisation (ie the organisation you have declared in your client section in your connection profile). It will not install to peers in organisations other than that. This makes sense, you would usually only have the admin rights to perform the install on peers your organisation owns. You wouldn't have the right to do it on peers you don't own.

#### npmrcFile option
As described earlier, fabric will perform an npm install on the business network which means that it must be able to communicate with a remote npm server. Company networks may have proxies or may with to run an NPM Server of their own which means that this part when the business network start request is made it would fail with errors trying to build the chaincode container image with problems contacting the npm registry. To help with this you can specify npm configuration options which get placed into the `.npmrc` of the business network to allow you to configure how npm works. This is something that is potentially unique to each organisation participating in a business network so is an extra option on the install command. For example to point to a different npm registry using the above example you may put the following line into `/tmp/npmConfig.txt`

```
registry=http://mycompanynpmregistry.com:4873
```

#### Under the covers
In order for a business network to actually work it needs to depend on the following npm package `composer-runtime-hlfv1`. It also needs to have an start entry in package.json in order to ensure that something happens when fabric starts the chaincode container using npm start. There is no documented requirement for business network developers to add this information to the package.json of the business network. This is because the composer network install command does this for you. It will add the necessary dependencies set to the current installed version of the composer-cli you have as well as the start option. For example if your business network package.json looked like

```
{
    "name": "sample-network",
    "version": "0.0.1",
    "description": "The Hello World of Hyperledger Composer samples",
    "author": "Hyperledger Composer",
    "license": "Apache-2.0"
}    
```

Then the install process will put this version of the package.json onto the peer for you (assuming composer-cli version 0.19.1)

```
{
    "name": "sample-network",
    "version": "0.0.1",
    "description": "The Hello World of Hyperledger Composer samples",
    "scripts": {
      "start": "start-network"
    },
     "author": "Hyperledger Composer",
    "license": "Apache-2.0",
    "dependencies": {
      "composer-common": "0.19.1",
      "composer-runtime-hlfv1": "0.19.1"
    }
}
```
if your package.json already has the composer-* dependencies then these will not be overwritten. This is useful if you want to control the versions yourself outside of the version of the composer-cli you have installed

### composer network start
This will start the fabric instantiation of the business network chaincode

- talk about providing an endorsement policy
- CORE_CHAINCODE_STARTUPTIMEOUT


## Using the Peer Commands
TBD (needs research)



### [Next - Upgrading Business Networks](./upgrade.md)
