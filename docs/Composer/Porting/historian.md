### [TOC](./TOC.md)
### [Back - ACLs](./acls.md)

# Historian
The Hyperledger Composer Historian is a specialised registry which records successful transactions, and in Composer, it references the Composer business network participants and mapped identities that submitted them.  It stores the list of transactions (that affect many different things) in a registry called ‘HistorianRecord’, defined in the Hyperledger Composer system namespace and thus, the data stored there has relationships to other Composer specific registries
It’s important to note  that several operations performed by the Hyperledger Composer runtime are classed as transactions and these 'system transactions' will add HistorianRecord assets for:
•	Adding, removing and updating assets (standard ‘CRUD’ operations, e.g. ‘AddAsset’, ‘UpdateAsset’ 
•	Adding, removing and updating participants
•	Issuing, binding, activating and revoking identities
•	Updating the business network definition itself (upgrading the business network is a transaction)

This is in addition to the custom transactions that you may have created in your model (these submitted transactions will also be registered in the Historian record). Custom transactions can affect one or multiple asset/participant registries, please note, different to the ‘system’ CRUD operations shown above.

There is no equivalent to historian in fabric. Historian was used to record the transaction object that were submitted (which resulted in 1 or more TP functions being invoked) and any events emitted. You would have to implement your own version of recording function invocation and event emission yourself. Access was either via rich queries to query the historian or via the `getHistorian` client SDK api.



Historian records are stored with a base key of `Asset:org.hyperledger.composer.system.HistorianRecord` and the extension key will be the value of the `transactionId` field.

```
asset HistorianRecord identified by transactionId {
  o String        transactionId
  o String        transactionType
  --> Transaction transactionInvoked
  --> Participant participantInvoking  optional
  --> Identity    identityUsed         optional
  o Event[]       eventsEmitted        optional
  o DateTime      transactionTimestamp
}
```

On the client side you could get access to this registry by using the `getHistorian` on a business network connection that then provided a `registry` object to be able to work with the HistorianRecord Asset.

## Getting the History of an Asset by Identifier (Composite Key) – Chaincode side
The most common use case for people is to get the history of an asset. Using historian for this was not ideal. In fact fabric itself provides a chaincode API `getHistoryForKey` to allow you to get the history of a key. As a key will map 1-1 to an asset or participant see [Data Storage](./datastorage.md) you can use this to get the history of an asset or participant. However in fabric 1.4 the results returned by the api were not guaranteed to be in chronological order.

Hyperledger Fabric manages a `key`'s history using a separate indexed GoLevelDB database to store the history of all keys in the Block storage. Its index points to blocks, and transactions within the block, which modified a given key (i.e. one that you use to query its history).
To get the history of transactions stored in Fabric, you would either use 
•	the Fabric Chaincode class method ‘getHistoryForKey’  as documented here -> https://fabric-shim.github.io/master/fabric-shim.ChaincodeStub.html#getHistoryForKey__anchor . Importantly, this is performed ‘chaincode side’ (you should never use the Client to pull history data FYI**). 
•	The Fabric Client class method ‘queryTransaction’ as documented here -> https://fabric-sdk-node.github.io/Channel.html#queryTransaction

For the ‘getHistoryForKey’ approach, you must provide the Composite key for the asset to target ; it returns a history of key / values for the asset concerned and includes the history of transactions IDs that targeted this key, and also whether the transaction in question was a Delete operation. This is the means to build up a history of transactions, details for which are stored in the relevant block. You would need to use the Fabric Node SDK to queryTransaction method to return the transaction – see below, for more on that).  For the ‘getHistoryForKey’ approach - an example below shows the use of a composite key, with qualifying namespace and identifier, as the key to pass to the `getHistoryForKey` function; 
getHistoryForKey(‘biz.acme.example:DigiBank0001’)

The History DB mentioned earlier is enabled In the Fabric config file called ‘core.yaml’, this HistoryDatabase should be set to true to enable the History retrieval described above:
```
history:
    # enableHistoryDatabase - options are true or false
    # Indicates if the history of key updates should be stored.
    # All history 'index' will be stored in goleveldb, regardless if using
    # CouchDB or alternate database for the state.
    enableHistoryDatabase: true
```

Each transaction in the blocks, contains the read-write set that modified one or more key/value pairs, who invoked the transaction (e.g. attributes from its certificate, if set up) , which members endorsed it etc, when querying the blockchain.

## Getting the History of Changes by Transaction ID (client-side)
Another aspect to getting ‘history’  is getting the history of a transaction – that is to say, the transactional unit of work. This means getting the read/write set for the transaction id in question, bearing in mind that the transaction may have targeted a number of different assets/keys or asset states.
Getting the historical ‘read/write’ set is a client operation (the client instance is available through the gateway please note), and to do this you will need to use the ‘queryTransaction’  method to query the ledger for the processed transaction, by transaction ID.
An example of getting the transaction payload, from the ledger, for a particular transaction ID using the Fabric 1.4 fabric-network APIs,  is shown below – we’re using an ‘Admin’ identity to perform the query task:
 
```
    const { FileSystemWallet, Gateway } = require('fabric-network');    //etc
 ….then …..

const userName = 'Admin@org1.example.com';
// Load connection profile; will be used to locate a gateway
let connectionProfile = yaml.safeLoad(fs.readFileSync('../gateway/networkConnection.yaml', 'utf8'));
// Set connection options; identity and wallet
let connectionOptions = {
  identity: userName,   // employee executing the transaction
  wallet: wallet,    // eg. FileSystemWallet wallet set up earlier in your code
  discovery: { enabled:false, asLocalhost: true }
};
// Connect to gateway using application specified parameters
await gateway.connect(connectionProfile, connectionOptions);
// get the client instance
const client = await gateway.getClient();

// Example below shows a sample historical transaction ID/peer/channel – please replace with your own 😊 
var trxnID = 'f113265574af19af3e22eac5e7b18ddabc43a424d582a6d7e766c3f95b142550';
var peer = 'peer0.org1.example.com';
var channel = await client.getChannel('mychannel');
if(!channel) {
      console.log('Channel ' + channelName + 'was not defined in the CCP');
      throw new Error('Failed to find channel');
}
let response_payload =  await channel.queryTransaction(trxnID, peer);
// for each ‘actions’ in the array below, get the transaction payload – e.g. actions[0] is one element below
console.log('transaction payload is ' + Buffer.from(JSON.parse(response_payload.transactionEnvelope.payload.data.actions[0].payload.action.proposal_response_payload.extension.response.payload)));
```

More information can be found at https://fabric-sdk-node.github.io/Channel.html#queryTransaction 


### [Next - Events](./events.md)