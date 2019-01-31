### [TOC](./TOC.md)
### [Back - Historian](./historian.md)

# Events
Composer provided a simple way to emit a chaincode event with a modelled data structure within a transaction processor function for example
```
// emit a notification that a trade has occurred
const tradeNotification = getFactory().newEvent('org.example.trading', 'TradeNotification');
tradeNotification.commodity = trade.commodity;emit(tradeNotification);
```

And a corresponding client SDK ability to receive those events as follows
```
businessNetworkConnection.on(‘event’, event => {...});
```

Events were only emitted once the peer that composer was listening for chaincode events from committed the block. To perform a similar action in the new contract implement requires you to use the setEvent API. Composer also allowed you to emit multiple events. This was done be pushing each event object into an array and sending the array back. So to replicate what composer did you might perform something like this (note that ctx is something provided in the new programming model for node in 1.4 if you are not using the new node programming model then ctx can be replaced with `stub` which is passed as part of the invocation mechanism of chaincode.

```
const events = [];
...
const firstEvent = {‘commodity’: trade.commodity};
events.push(firstEvent);
const secondEvent = {...} // some other event object
events.push(secondEvent);
ctx.stub.setEvent(‘myevent’, Buffer.from(JSON.stringify(events))); // myevent can be whatever you chose as you can listen for events with this value on the client side 
```

Unfortunately there is nocapability in the new 1.4 node sdk fabric-network facility to easily help with chaincode events so at this time you need to do all the work yourself including registering a chaincode event listener, managing replay and performing whatever HA is required on channel event hubs. 

Example code on how you might set up a chaincode event listener is given here. Note how it shows how to wait for the event hub to be connected first before you submit your first transaction. This is important because there is no guarantee that a request to connect to an event hub will be processed before a transaction is committed even if you think you sent the connect request first.

```
async function listenForChaincodeEvents(network, mspid, chaincodeId) {
    const channel = network.getChannel();
    const peers = channel.getPeersForOrg(mspid);
    const eventHub = channel.newChannelEventHub(peers[0]); // choose first peer

    const waitToConnect = new Promise((resolve, reject) => {
        eventHub.connect(true, (err, eventHub) => {
            if (err) {
                reject(err);
            }
            resolve();
        });
    });
    await waitToConnect;

    // register for chaincode events once the event hub has connected successfully otherwise
    // it could fire this event handler during the connect process.
    const handle = eventHub.registerChaincodeEvent(chaincodeId, 'myevent',
    (event, blockNum, txID, status) => {
        if (status && status === 'VALID') {
            let evt = event.payload.toString('utf8');
            evt = JSON.parse(evt);
            // handle the event data, evt will be an array
        }
    },
    (err) => {
        // handle situation where chaincode event listening is lost.
    });   
    return handle; 
}

let ccHandle = await listenForChaincodeEvents(network, 'mycontract');
// ready to start sending transactions
```

Ensure you disconnect the eventHub just before your application terminates. You do not need to disconnect it once you have received the event. Constant connection/disconnection is not a good thing to be doing.
### [Next - Identity Management](./identity.md)