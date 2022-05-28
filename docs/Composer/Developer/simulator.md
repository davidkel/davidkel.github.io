### [TOC](./TOC.md)

# Composer runtime simulator
Composer business networks never communicated with the underlying DLT technology directly unless they used the getNativeAPI capabilities. Therefore it was possible to provide a simulation of the runtime with a dummy DLT representation to create a simulator. This simulator was exposed as the Embedded Connector used to help developers write unit tests for their business networks.

An experimental feature that was never documented was included in composer that allowed this simulator to look like a business network running on a real fabric. You could interact with it using the composer cli commands, run the composer rest server against it as well as have composer playground interact with it (cannot deploy ???). It provided a way to provide a much faster development environment than a real fabric because chaincode upgrade was instantaneous.

## limitations
With the simulator everything is held in memory, so if you terminate the simulator everything is lost and a restart will be a clean environment. Also it can only support the deployment of a single business network at a time.

## identities required to access the simulator
The simulator doesn't check for access of identities



## running the simulation server
The server could be started in 1 of 2 ways depending on whether you plan to run composer-playground or not.


### using composer-playground
you must run composer-playground on port 15699, so start playground as follows
```
composer-playground -p 15699
```
All web access to playground should now explicitly use port 15699
```
localhost:15699
```

### not using composer playground
- npm install -g composer-connector-server
- composer-connector-server

Do not try to run composer-playground to connect to the simulator. If you want to use playground then
follow the instructions for using composer-playground above.


## creating a card to access the simulation server
first you need a connection profile. The important field is the `x-type` which must be `embedded@proxy`. The name field is whatever you want to call the connection profle and version is ignored but should be present

```
{
    "name": "embedded",
    "x-type": "embedded@proxy",
    "version": "1.0.0"
}
```

composer card create -p connection.json -u admin -s dummy -r PeerAdmin -r ChannelAdmin (has to be admin for install/instantiate)

composer card import -f admin@embedded.card


Then you can use playground as normal (if you are running the simulator for playground use), or the CLI as normal to deploy your business network etc
- composer network install -c admin@embedded -a basic-sample-network.bna
- composer network start -c admin@embedded -n basic-sample-network -V 0.2.6 -A admin -S adminpw (has to be admin, but pw can be anything)
- composer card import -f admin@basic-sample-network.card
- composer network ping -c admin@basic-sample-network

You can start the rest server as follows
- composer-rest-server -c admin@basic-sample-network

You can also then create new identities and bind to participants.
