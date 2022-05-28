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

With the release of 1.4.2 node sdk fabric-network, support for for listening for various types of events (transaction, block, chaincode) is included. From a composer porting point of view we are only interested in chaincode events. 
Example code on how you set up a chaincode event listener for the above emitted event code is shown below. 

```
this.contract.addContractListener('unique-id-1', 'trade-network',
    (err, event, blkNum, txid, status) => {
        console.log('event received', status, event, blkNum, txid);
        if (err) {
            // handle an error
        } else if (status && status === 'VALID') {
            // only if a valid block is committed should we emit an event
            let evt = event.payload.toString('utf8');
            evt = JSON.parse(evt);
            if (Array.isArray(evt)) {
                for(const oneEvent of evt) {
                    // TODO: process each event in the array
                }
            }
            else {
                // TODO: only a single event, ie not an array to be handled
            }
        }
    }
);

```

Ensure you disconnect the gateway(s) you are using at application termination. 
### [Next - Identity Management](./identity.md)