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
- Client side transaction retry
  - Handling MVCC Read conflicts


      

```

```
## Client error handling and retying transactions
- MVCC_READ_CONFLICT
- Phantom reads
- Timeout waiting for commit events
- successful submission to orderer is no guarantee that it will be included in a block (see timeout).  Successful delivery to orderer does not guarantee it being included in a block and being delivered (successful delivery to orderer does not guarantee tx gets ordererd, and successful ordering does not guarantee tx gets validated by peers)
```
			Event strategy not satisfied within the timeout period. No response received from peers:
      Event strategy not satisfied within the timeout period. No response received from event hubs:       
```

- endorsement policy failure
  - occurs if you don't get enough signatures
  - also occurs if you do get enough signatures but the proposals don't match (ie 1 or more of the signatures sent weren't over the proposal response that was sent (sdk decides which proposal response out of all received is sent)) see https://stackoverflow.com/questions/66237592/endorsement-policy-failure-while-invoking-chain-code-using-even-if-the-transacti
 

```javascript
  public static readonly errorMessagesToRetry: string[] = [
    'MVCC_READ_CONFLICT',
    'ENDORSEMENT_POLICY_FAILURE',
    'CHAINCODE_VERSION_CONFLICT',
    'EXPIRED_CHAINCODE',
    'CKR_OPERATION_ACTIVE',
    'CKR_OPERATION_NOT_INITIALIZED',
    'CKR_DEVICE_ERROR',
    'CKR_SESSION_HANDLE_INVALID',
    'CKR_KEY_HANDLE_INVALID',
    'SERVICE_UNAVAILABLE',
    'UNAVAILABLE: failed to connect to all addresses',
    'Key not found',
    'No endorsement plan available',
    'Stream removed'
  ];
```

Under some circumstances a retry of the transaction is the right thing to do, but there needs to be control so that you don't end up in a permanent loop trying to perform a transaction that will always fail.

The other thing is to avoid is getting into a snowball effect of retries. If you overload a peer with constant proposals then this stops the peer performing validation of existing blocks which could cause the network to stop.

Danger that retrying will just keep the same error (improved now node sdk randomly selects orderers if the orderer was down)

The gateway apis don't allow you to send the same proposal responses (thus keeping the same tx id) which could be used for a timeout situation. If it fails with a transaction id already exists then you know it was committed.

Handling chaincode termination might also be useful. Chaincode dying will create a chaincode stream terminated type error

## block height, eventual consistency and peer catch up
(NEED INFO about submission and query etc)
How blocks are disseminated

## Transaction submit

When you submit a transaction (ie one that will subsequently be sent for ordering) you need to ensure you select enough peers that will satisfy the endorsement policies that will govern the execution of that transaction. That also has to include invoking other chaincodes for which you are allowed to write to keys. Endorsement policies (in fabric 2.2) come from 4 separate places

- chaincode endorsement policy (ie the endorsement policy governing the whole chaincode)
- private collection policy (a new feature in fabric 2), can use discovery hints in the node sdk to include
- State based endorsement policy
- endorsement policies in place for chaincode that is invoked by other chaincode, can use discovery hints in the node sdk to include

Whether you take advantage of discovery within the node sdk or have to resort to doing it yourself will depend on your chaincode implementation. The node SDK is capable of handling the first 2 screnarios and this will provide an implementation that will try to perform high availability by taking into acount unavailable peers, the recent block height of peers and will also try other possible peers in an attempt to satisfy the endorsement policy. So if you can this is the best route to take when trying to create a client application that is resilient to peers being unavailable.

Generally you would use discovery, however there are situations where discovery won't do what you need

- State based endorsement. The node-sdk cannot determine the endorsement policies on keys that chaincode will interact
- making sure you don't send requests that contain private data in the transient map to orgs that should not have visibility of this information and your endorsement policies won't protect against sending to the wrong organisation

In these situations you have a couple of options

- organisation targeting
- peer targeting

### Organisation targeting

Unfortunately due to a bug/working as designed state of the node sdk 1.4, when targeting organisations the chaincode endorsement policy is also taken into account (so for example if your chaincode endorsement policy is 1 of any but you try to target 2 organisations, it will pick 1 from the 2 presented organisations). I don't believe this is the behaviour in the node sdk 2.2

Here you can specifically target organisations and the node sdk will select peers for you. At this time I don't know what sort of retry capabilities are done by the node sdk nor how the peers are selected but I would assume they take block height into account

(NEEDS MORE WORK)

### Peer targeting

(NEEDS MORE WORK)
- danger if you target multiple peers in the same org that you could end up with mismatched proposals if a peer in your org is in catch up.

### proposal submission to orderer

(NEED MORE WORK)

- proposal comparision
  - not in 1.4
  - is in 2.2
    - May not be what you want and can't be turned off
      - eg your endorsement policy may allow 1 of the proposals to not count and doesn't matter if it didn't match
- proposal review
  - Just not possible using the gateway api, which is a real shame
- orderer selection
  - node SDK tries other orderers if it fails to send to an orderer
  - older versions of node-sdk 1.4 always selected the first in the list
  - as of 1.4.13 and node sdk 2.2.2 (CHECK THIS) orderers are randomly selected from the list so it isn't always sending to the same one

Orderer selection attempts to handle unresponsive orderers to ensure the application continues to work

## Transaction Query

(NEEDS WORK)

## Event Handling

(NEEDS WORK)

## Private Data and targeting

(NEEDS WORK)

- transient store
- sending to all peers at the same time (live backup)
- transient store size (if you fill this up you can lose data)

## Data replication and catchup

(NEEDS WORK)

- all orderers hold a copy of the blockchain for each channel
- all peers hold a copy of the blockchain for each channel they join
- world states can be recreated from the channel chain
- PDC chains are only held by peers in the collection
  - implicit collections or PDCS for a single org means only your peers have a copy
    - Really need to think about backup and more than 1 peer (ideally 3)
      - 3 peers so that you have 1 + 1 live backup if 1 peer is offline (eg maintenance) 
  - PDCs rely on other peers to catch up from

