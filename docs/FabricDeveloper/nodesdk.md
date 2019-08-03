# Node SDK tips
This section provides general information about developing node.js cliernt application. It covers my experience of using `fabric-network`, the so called `low-level` api as well as the `low-level` apis in fabric-client and fabric-ca-client. The fabric-network api provides an easier to use api and solves some of the complexitities of the `low-level` api such as event handling. Event handling is one of the major issues that application developers got wrong when trying to develop node client applications. `fabric-network` unfortunately only provides a subset of capabilities, so you will still find yourself having to work with `fabric-client` and `fabric-ca-client` for your applications. A summary of `fabric-network` capabilities are
1. Identity Storage Management through Wallets
2. Submit/Notify (incorporating transaction event handling) with HA Capabilities
3. Query with HA capabilities
4. pluggable transaction event strategies
5. pluggable query handlers
6. pluggable wallets (maybe, not sure this was published)
7. pluggable event stuff for other types of events such as chaincode/block (this seems horrible and complex personally)


## HSM Support


## pluggable commit handlers
well hidden feature of fabric-client (also used by fabric-network)

## pluggable endorsement handlers
well hidden feature of fabric-client (also used by fabric-network)

## performing special queries using `fabric-network`
the low level node sdk provides some access to things like querying by block number or by transaction id. You can still do this using fabric-network by obtaining the underlying `channel object`, but there is another way which utilises the benefit of the more robust query handlers available in fabric-network.

```javascript
	const BlockDecoder = require('fabric-client/lib/BlockDecoder');

	const ProtoLoader = require('fabric-client/lib/ProtoLoader');

	const queryProtoLoc = require.resolve('fabric-client/lib/protos/peer/query.proto');
	const queryProto = ProtoLoader.load(queryProtoLoc).protos;

	const ledgerProtoLoc = require.resolve('fabric-client/lib/protos/common/ledger.proto');
	const ledgerProto = ProtoLoader.load(ledgerProtoLoc).common;

	// assume we have an appropriate gateway with an identity which can perform all the 
	// queries
	const channel = 'mychannel';
	const network = await gateway.getNetwork(channel);
	const qscc = network.getContract('qscc');
	const lscc = network.getContract('lscc');
	const cscc = network.getContract('cscc');

    let resources = await this.qscc.evaluateTransaction('GetBlockByNumber', channel, '7');
    console.log(BlockDecoder.decode(resources).data.data[0].payload);

    resources = await this.qscc.evaluateTransaction('GetTransactionByID', channel, 'a494d037b2930263a533bab78efbe7bb2635fcda9b44ae58e03a275bad6b9331');
    console.log(BlockDecoder.decodeTransaction(resources));

    resources = await this.qscc.evaluateTransaction('GetChainInfo', 'mychannel');
    console.log(ledgerProto.BlockchainInfo.decode(resources));

	// requires channel admin authorities
	let resources = await this.cscc.evaluateTransaction('GetConfigBlock', 'mychannel');
	console.log(BlockDecoder.decode(resources).data.data[0].payload)
	
    resources = await this.lscc.evaluateTransaction('getchaincodes');
    console.log(queryProto.ChaincodeQueryResponse.decode(resources));
}
```
Using evaluate transaction by default will go to a peer in your organisation and it doesn't matter which peer you go to. For queries that are peer specific (eg getinstalledchaincodes) evaluateTransaction is not a suitable way to invoke. 

Now unfortunately because we are accessing modules and files inside of the node-sdk which aren't published externally in any official capacity then this is not a supported mechanism, but it's unlikely to change in the v1 releases of the node-sdk. However it could change in the v2 node-sdk, but hopefully this might get exposed in a more official manner.

Also it means you have to do all the decoding work that the sdk does as well.
