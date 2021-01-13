# Guidelines for writing Highly Available Node Clients

This builds on top of the writing node clients section. A client application needs to do 3 things

1. Submit transactions (ie commit to the blockchain)
2. evaluate transactions (to retrieve some data)
3. process events

Each area requires consideration on how to handle

- One or more peers are down
- One or more peers are in catch up
  - worst case scenario is where a peer is joined to the channel and there is a large block height
- One or more orderers are down
- Handling MVCC Read conflicts
- Handling other types of error
