# A multi-organisational hyperleger fabric network
The business network doesn't care about the fabric network, but the operational logistics obviously will change. This is what the multiple organisation tutorial in the hyperledger composer documentation is all about. It's not so concerned about how to create the hyperledger fabric infrastructure and setup, but how composer would interact with such as a network.

## Starting the BYFN network
This is the fabric network consisting of 2 organisations with 2 peers and 1 CA per oganisation and a single orderer for the network. To get this up and running you need to have installed the previous pre-requisites of `vagrant` and `virtualbox`, but this time you will download a new package

1. create a directory called byfn-network
2. download the [BYFNServer.zip](https://raw.githubusercontent.com/davidkel/ComposerVagrant/master/BYFNServer/zipped/BYFNServer.zip) file
3. unpack the zip file into your new byfn-network directory
4. change to that directory in a command prompt window
5. run the command `vagrant up` in the current directory to bring up the byfn-network for the first time
6. wait for the following to appear

```

    ========= All GOOD, BYFN execution completed ===========


     _____   _   _   ____
    | ____| | \ | | |  _ \
    |  _|   |  \| | | | | |
    | |___  | |\  | | |_| |
    |_____| |_| \_| |____/
```


Once the BYFN server is up, you will need to press CTRL-C to return to a command prompt. Why the BYFN script never returns to the command line is a nuisance, but it's easily handled.

So now you are ready to go. All the necessary connection profiles and identities (represented by private keys and public certificates) for the organisations will have been copied into the `byfn-network\composer` directory where you ran the `vagrant up` command, see next section for more information.

## Artifacts created
The BYFN network has 2 organisations (Org1, Org2), 2 peers per organisation (peer0, peer1), 1 ca per organisation and 1 solo orderer defined to it's own organisation, a single channel has been defined called `mychannel`.
In order for a composer application to be able to interact with this network the following is required and can be found in the `composer` directory.
1. A base connection profile usable by each organisation (requires tailoring for each organisation) - `byfn-network.json`
2. The identity (public certificate and private key) of an Administrator for Org1 who has authority to install chaincode on the peers. These files will be in the `org1` subdirectory with the public certificate being the file ending with `.pem` and the private key being the file ending with `_sk`. 
3. The identity (public certificate and private key) of an Administrator for Org2 who has authority to install chaincode on the peers. These files will be in the `org2` subdirectory with the public certificate being the file ending with `.pem` and the private key being the file ending with `_sk`.
4. The channel has been set up such that both the Org1 and Org2 Administrators are also administrators for the channel.

## Following the composer multi-org tutorial.

### Step 1
Now you have the BYFN fabric up and running, this takes care of most of step 1. You should delete any old business network cards described at the end of step 1

### Step 2
This provides more details about the BYFN network and how it is set up. It also describes where to find the TLS CA Certificates for the Orderer, Org1 and Org2 as well as the Administrative identities (in the form of a public certificate and private key) for each organisation. Finally it also gives information about how to create a connection profile for this network. Here a lot of the work has been done for you. The base connection profile has already been prepared as described above in the `Artifacts created` section.

### Steps 3 and 4
Use the provided connection profile as the base to creating your own connection profile for each organisation.

### Steps 5 and 6
The public certificate and private key for the administrator for each organisation have already been conveniently copied for you as described in the `Artifacts created` section.

### steps 7 and 8
Follow the commands the create a the business network cards but point to the appropriate files in the composer subdirectory (ie remove the `/tmp/`) from the commands to create the business network cards so the ones in your directory are used. 
