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

So now you are ready to go. All the necessary connection profiles and identities (represented by private keys and public certificates) for the organisations will have been copied into the `byfn-network\composer` directory where you ran the `vagrant up` command

## Following the composer multi-org tutorial.
TBD
