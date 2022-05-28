### [TOC](./TOC.md)

# running business networks in development mode

One of the issues people have with composer is that they wanted to run a business network in a real fabric but it was such a pain to have to keep changing the version number of the business network and perform a chaincode upgrade to work with a new version of a business network. The process took a few minutes and would leave old chaincode containers running. Although this process was automated via playground, playground isn't an IDE and really was only meant as a getting started tool, not for continued business network development. 

This section talks about how to run a business network with fabric in development mode. The advantage of this is that you could run the business network from a terminal connected to a real fabric and test. You can then stop the process make some changes to the business network and start the process up again instantly picking up the new changes to test.

By example, here are the steps to do this.

(if you need to get started :-) )

## setup from scratch if required
- curl -O https://hyperledger.github.io/composer/latest/prereqs-ubuntu.sh
- chmod u+x prereqs-ubuntu.sh
- mkdir ~/fabric-dev-servers && cd ~/fabric-dev-servers
- curl -O https://raw.githubusercontent.com/hyperledger/composer-tools/master/packages/fabric-dev-servers/fabric-dev-servers.tar.gz
- tar -xvf fabric-dev-servers.tar.gz

## start fabric in development mode
- npm install -g composer-cli
- ./startFabric.sh -d
- ./createPeerAdminCard.sh

## clone sample networks to get started
- cd ~
- git clone https://github.com/hyperledger/composer-sample-networks
- cd composer-sample-networks/packages/basic-sample-network
- update package.json to include dependencies required for the business network (you don't have to do this normally as the composer network install command will inject these dependencies for you)
```
  "dependencies": {
    "composer-runtime-hlfv1": "0.20.8",
    "composer-common": "0.20.8"
  },
```
- npm install --production

## build the bna and install it on the peer
build a bna (it never get's used but something has to be installed on the peer with the right chaincode id and version number
- composer archive create --sourceType dir --sourceName . -a dummy.bna
- composer network install -c PeerAdmin@hlfv1 -a dummy.bna

## run the business network code
```
CORE_CHAINCODE_ID_NAME=basic-sample-network:0.2.6 node node_modules/composer-runtime-hlfv1/start.js --peer.address localhost:7052
```
Note that CORE_CHAINCODE_ID_NAME must match the business network name and version in the format `name:version`

## tell fabric to start the business network
- composer network start -c PeerAdmin@hlfv1 -n basic-sample-network -V 0.2.6 -A admin -S adminpw
- composer card import -f admin@basic-sample-network
- composer network ping -c admin@basic-sample-network

Note now the code in the window is executed, you can now stop the code with Ctrl-C modify it and restart the business network with the above
command again and the new code will be used.

So for example you could install playground to drive the business network, or you could install the composer-rest-server. Everything will work as before except you now have control to stop the business network to make changes and just re-start it.

## modify the business network 
let's demonstrate a modification. So startup playground
- create a sample participant (P1) and sample asset (A1) with a value of 100
- run the sample transaction specifying the new value to be 98. As you see the asset has a value of 98.
- go to the window where the business network is running
- stop the business network by pressing CTRL-C
- change the line
```
tx.asset.value = tx.newValue;
```
to be
tx.asset.value = (tx.newValue * 2).toString();
console.log('------>  I have made a modification');
```
- restart the business network
```
CORE_CHAINCODE_ID_NAME=basic-sample-network:0.2.6 node node_modules/composer-runtime-hlfv1/start.js --peer.address localhost:7052
```

- in playground run the sample transaction again and specifying the new value to be 34. This time we see the asset now has the value 68 and our console message appears in the window where the business network is running.