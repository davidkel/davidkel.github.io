# (DRAFT) Hyperledger Fabric 2.5 Performance Considerations (DRAFT)

Hyperledger Fabric performance is a question that comes up frequently as people look to try to compare it transactional databases or other blockchains and determine what the maximum TPS (Transactions Per Second) of a network is. Performance in a Hyperledger Fabric network is complex because in an appropriately deployed network there will be many organisations participating (and these organisations will provision or provide their own infrastructure to host their fabric components), multiple channels, required policies, chaincode implementations, networking infrastructure etc.

This article will provide an introduction into the considerations needed to improve performance of a Hyperledger Fabric network.

## Fabric Version

Fabric 2.x has performance improvements over Fabric 1.4. Fabric 1.4 is now out of LTS and should not be used in production now. Fabric 2.5 is the new LTS version which includes the new Peer Gateway service which provides much better throughput for applications than using the legacy SDKs.

## Hardware considerations

Each organisation may provide it's own hardware to host their Peer and ordering (Depending on how you deploy the ordering service in the network (A dedicated organisation or Ordering services split across organisations) services. It's important to realise that just 1 organisation could impact the whole network (depending on policies such as endorsement policies) therefore each organisation must ensure they provide adequate resources for the services they own within the hyperledger fabric network.

As Hyperledger Fabric performs a lot of disk I/O it goes without saying that you should ensure you are using the fastest disk storage available to you and that the physical drive it ultimately resides on should be SSD.

A Hyperledger Fabric network is highly distributed across multiple nodes usually hosted on multiple different clusters, on different clouds and even in different countries, therefore high speed network connectivity between nodes is vital and at least 1Gbs should be deployed between all nodes/organisations

The final piece is how much CPU and Memory should be allocated to Peers, Orderers and, if used, CouchDB instances ? There is no real answer to this one but it does affect the performance of the network and even the stability of the nodes if not enough resources are allocated. It's vital therefore that you constantly monitor the CPU, Memory, Disk Space, Disk and Network IO to ensure that you are within the limits of your allocated resources. Do not wait until you see these values constantly hitting 99% before deciding to increase the resources. You should be doing so at a much lower threshold to avoid problems that will occur if you run out of resources. A threshold of between 70% and 80% is a good starting point.

Performance generally scales with CPU allocated to peer nodes. Providing each peer and CouchDB (if used) with the maximum CPU capacity is recommended. For orderer nodes a good starting point may be 1 CPU with 2GB of memory.

As the blockchain height grows the hyperledger fabric network performance will reduce. With CouchDB this is more prominent but even using levelDB this can be seen, therefore you will find that during the lifetime of your fabric network will you need to add more compute resource to the environment which just re-emphasises the point that you must monitor your network components and react to any thresholds being exceeded.

## Peer Configuration Considerations

Although covered later it's important to note that using CouchDB for the state database will have a noticeable impact on performance. In fact it is highly recommended that CouchDB is not used, refer to the CouchDB considerations section later for more information.

This section highlights some of the considerations for a peer such as query limit, concurrency limits and active channels.

### Total Query Limit

A Peer limits the total number of records a range or JSON (rich) query will ever return. This is configured in the core.yaml file of a peer

```yaml
ledger:
  state:
    # Limit on the number of records to return per query
    totalQueryLimit: 100000
```

100000 is the default that fabric provides in the sample core.yaml and in the test docker images. If you are exceeding this limit, you can increase this value but I would suggest that if you are exceeding this limit that the query is returning too many records and you should look to keeping the range within the limit

### Concurrency limits

The peer has some limits to ensure a peer cannot be overwhelmed by clients as defined here

```yaml
peer:
# Limits is used to configure some internal resource limits.
    limits:
        # Concurrency limits the number of concurrently running requests to a service on each peer.
        # Currently this option is only applied to endorser service and deliver service.
        # When the property is missing or the value is 0, the concurrency limit is disabled for the service.
        concurrency:
            # endorserService limits concurrent requests to endorser service that handles chaincode deployment, query and invocation,
            # including both user chaincodes and system chaincodes.
            endorserService: 2500
            # deliverService limits concurrent event listeners registered to deliver service for blocks and transaction events.
            deliverService: 2500
            # gatewayService limits concurrent requests to gateway service that handles the submission and evaluation of transactions.
            gatewayService: 500
```

The new Peer Gateway service introduced a limit in the `gatewayService` and the default in the sample configuration and test docker images is 500. However this will restrict the TPS of a network so you may need to increase this value. However don't set it too high as it ensures that the peer doesn't get overloaded with requests causing the TPS to actually drop. For high throughput I used values upto 20000 for performance benchmarking.

### Number of channels a peer is participating in

If a peer has ample CPU and is only participating in a single channel then it's not possible to drive the peer beyond around 65-70% CPU. This is due to the way the peer does serialisation and locking internally. Multiple active channels on a peer will use the extra resource as it increases parallelism in block processing.

In general, do not have the number of active channels > the number of CPU core. Again, it is not a hard limit. If the load in each channel is not highly correlated (i.e., not every channel is contenting for resources at the same time), we could have more channels than the number of CPU cores.

### CouchDB Cache setting

If you are using CouchDB and have a large number of keys being read (not via queries) then you may want to investigate increasing the CouchDB cache a defined in the peer core.yaml file

```yaml
state:
    couchDBConfig:
       # CacheSize denotes the maximum mega bytes (MB) to be allocated for the in-memory state
       # cache. Note that CacheSize needs to be a multiple of 32 MB. If it is not a multiple
       # of 32 MB, the peer would round the size to the next multiple of 32 MB.
       # To disable the cache, 0 MB needs to be assigned to the cacheSize.
       cacheSize: 64
```

this is the default that is provided in the sample core.yaml and the test docker images

## Orderer Considerations

The orderer uses raft concensus to cut blocks, factors such as the number of orderers in that concensus and block cutting parameters will affect performance

### Number of orderers

The more Orderers you have will impact performance as all orderers are involved in Raft consensus, 3 will give you 1 orderer crash fault tolerance and 5 will give 2 orderer crash fault tolerance. So if you can accept the risk then you could reduce the number of orderers from say 5 to 3.

### SendBufferSize

In fabric prior to 2.5 the SendBufferSize of the orderer was set to 10 as a default which causes a bottleneck. In Fabric 2.5 this was changed to 100 which provides a much better throughput. The configuration for this can be found in orderer.yaml

```yaml
General:
    Cluster:
        # SendBufferSize is the maximum number of messages in the egress buffer.
        # Consensus messages are dropped if the buffer is full, and transaction
        # messages are waiting for space to be freed.
        SendBufferSize: 100
```

### Block cutting Parameters (TBD)

A larger block size and timeout could increase the throughput (but latency would increase as well). The block size is highly related to the transaction arrival rate. For e.g., if the transaction arrival rate is 100 per sec and block size and timeout is 3000 & 10 secs, respectively, the throughput would be 10 tx per second. If the arrival rate is higher for the same configuration, the throughput would be higher.

- You might want to try to configure the ordering service with more transactions per block and longer block cutting times
to see if that helps. We have seen this increase the overall throughput at the cost of additional latency.

The following three parameters work together to control when a block is cut, based on a combination of setting the maximum number of transactions in a block as well as the block size itself.

#### Absolute max bytes

Set this value to the largest block size in bytes that can be cut by the orderer. No transaction may be larger than the value of Absolute max bytes. Usually, this setting can safely be two to ten times larger than your Preferred max bytes. Note: The maximum size permitted is 99MB.

#### Max message count

Set this value to the maximum number of transactions that can be included in a single block.

#### Preferred max bytes

Set this value to the ideal block size in bytes, but it must be less than Absolute max bytes. A minimum transaction size, one that contains no endorsements, is around 1KB. If you add 1KB per required endorsement, a typical transaction size is approximately 3-4KB. Therefore, it is recommended to set the value of Preferred max bytes to be around Max message count * expected averaged tx size. At run time, whenever possible, blocks will not exceed this size. If a transaction arrives that causes the block to exceed this size, the block is cut and a new block is created for that transaction. But if a transaction arrives that exceeds this value without exceeding the Absolute max bytes, the transaction will be included. If a block arrives that is larger than Preferred max bytes, then it will only contain a single transaction, and that transaction size can be no larger than Absolute max bytes.
Together, these parameters can be configured to optimize throughput of your orderer.

#### Batch timeout

Set the Timeout value to the amount of time, in seconds, to wait after the first transaction arrives before cutting the block. If you set this value too low, you risk preventing the batches from filling to your preferred size. Setting this value too high can cause the orderer to wait for blocks and overall performance to degrade. In general, we recommend that you set the value of Batch timeout to be at least max message count / maximum transactions per second

## Application Considerations

When designing the application architecture, decisions can affect the overall performance of the network and the application. Here we cover some of the considerations

### Don't use CouchDB

CouchDB performance is noticably slower than LevelDB sometimes by a factor of 2x slower. The only real capability that you get by choosing CouchDB is JSON (rich) queries within query/evaluate transactions as stated in the fabric documentation [State Database](https://hyperledger-fabric.readthedocs.io/en/release-2.5/deploypeer/peerplan.html#state-database) in the fabric documentation. You should also never have direct access to the CouchDB data (for instance via Fauxton) to ensure integrity of the data.

CouchDB also has other limitations which will have impacts on performance, extra hardware resource required to provide CouchDB for each peer (which incurs additional costs) and no extra protection from fabric (JSON queries are not re-executed at valisation time and thus you don't get any Phantom Protection from fabric as you do with range queries).
If at all possible consider looking at an offchain store to support queries for which fabric provides a sample [Off-chain sample](https://github.com/hyperledger/fabric-samples/tree/main/off_chain_data) to demonstrate exactly this. Doing so gives you much more control over the data and query transactions do not affect the hyperledger fabric network performance. It also then gives you a choice of the type of system to use for off-chain storage, for example you could use a SQL Database to store the information bring the benefits of SQL to your application.

CouchDB performance degrades more than leveldb as the block height increases significantly meaning you will need to make sure you provide adequate resources for CouchDB instances for the life of the application.

### Use the new Peer Gateway Service

The Peer Gateway Service and the new Fabric-Gateway client APIs are a substantial improvement over the legacy Go, Java and Node SDKs. Not only do they provide much improved throughput, it also provides better capability reducing the complexity of a client application. For example it will strive to collect enough endorsements to satisfy not only the chaincode endorsement policy but also any State Based Endorsement policies that get included when the transaction is simulated, something that was not possible with the legacy SDKs.
It also reduces the number of network connections a client needs to maintain in order for a client to submit a transaction. A client could easily require many connections to multiple nodes. This has been reduced to 1 when using the new gateway service.

One point to remember that if you used the legacy SDKs then a client requires more CPU and memory than a client using the new Peer Gateway service so this reduces the client resource requirements but it does increase the peer resource requirements however because the Peer is written in Go and already has a lot of the information stored internally the peer implementation will still require less resources than even a Go client using the legacy SDK.

### payload size

The amount of data that is submitted to a transaction, along with the amount of data written to keys in a transaction (it's unusual for a non query/evaluate transaction to return data but if it does that will also be included in the final payload size) will affect the application performance. Note that the payload size includes more than just the data. It includes structures required by fabric plus signatures, but an application has control of the data it sends and is written to keys so be mindful of this.

Surfice to say large payload sizes are a bad idea and Hyperledger Fabric is not unbounded in this respect. You are likely to hit limits within Fabric and encounter performance and block replication issues especially if CouchDB is used.

### Which chaincode language is most performant

Golang.

Node is the next performant. Java performance is by far the least performant and would not recommend it unless absolutely necessary.

### Endorsement policies

For a transaction to be committed as valid, one of the things it must contain is enough signatures to satisfy the chaincode endorsement policy and any state based endorsement policies. The Peer Gateway service will do a sterling job of only sending requests to enough peers to satisfy this collection of policies (and will also try other peers if the preferred ones are not available). Thus we can see that endorsement policies will affect performance as it dictates how many peers and thus how many signatures are required to ensure this transaction could be committed.

## Couchdb considerations If you really must use it

As mentioned earlier you should try to avoid using CouchDB for the state database and look to providing an off chain store to perform queries on and use leveldb instead, but if you do plan to use it these are the things that need to be considered.

### Resources

Ensure you monitor and the resources of the CouchDB instances, as the block height increases and the more information is stored in CouchDB, CouchDB will consume more resources.

### CouchDB Cache

When using external CouchDB state database, read delays during endorsement and validation phases have historically been a performance bottleneck. In Fabric v2.x, the peer cache replaces many of these expensive lookups with fast local cache reads.

The cache will not improve performance of JSON queries

### Indexes

Ensure you use indexes and don't use queries that can't use indexes. For example use of the query operators $or, $in, $regex result in full data scans. See [Hyperledger Fabric Good Practive For Queries](https://hyperledger-fabric.readthedocs.io/en/release-2.5/couchdb_as_state_database.html?highlight=%24regex#good-practices-for-queries)

Optimise your queries, complex queries will take more time even with indexing and ensure your queries result in a bounded set of data. Remember fabric will also limit the total number of results returned.

You can check peer logs to see how long queries are taking and also whether a query was unable to use an index from warning log entries stating the query "should be indexed".

If queries are taking a long time, you could try to increase CPU/Memory available to CouchDB or use faster storage.

Check CouchDB logs for warnings such as "The number of documents examined is high in proportion to the number of results returned. Consider adding a more specific index to improve this." These messages show you that your indexes may not be good enough for the query being performed as it is resulting in too many documents having to be scanned.

### Bulk Update

Fabric uses bulk update calls to CouchDB to improve CouchDB performance. A bulk update is done at the block level so including more transactions in a block could potentially improve throughput. However increasing the time before a block is cut to include more transactions will have an impact on latency.

## Misc

Sending single transactions periodically will have a latency correlating to the block cutting parameters. For example if you send a transaction of 100 Bytes and the Batch Timeout is 2 seconds then the time from submission to being committed will be just over 2 seconds. This is not a true benchmark of your fabric network performance, it's expected that multiple transactions will be submitted simultaneously to gauge the true performance of a fabric network.

## Performance runs

Any performance runs need to take into account

- Gateway request limit (I used 10000, 20000): it's good to note that on blind writes you can see the backlog of unfinished transactions being limited to 20000 but it will also register failed transactions as they are rejected
- Orderer SendBufferSize
- Use GoLang non contract chaincode
- bind to fabric 2.4 in caliper
- block cutting parameters
- maybe experiment with ValidatorPoolSize: only > no. of cores makes sense
- block cutting parameters
- read/write test
- 100 byte, 16K byte asset size (maybe even 100byte, 1K bytes ?)
- leveldb only
- logging and comparison of responses
- go non-contract implementation
- endorsement policy (1 of)
- no private data
- no range/rich queries
- 2 orgs
- 1 orderer
- theoretical maximum ?
- standard OOTB fabric polices
- no event handling as such
- couchdb cache (not used)
- tls vs non-tls (fabric 3.0) TLS provided by external system ?
https://github.com/hyperledger/fabric-samples/blob/main/full-stack-asset-transfer-guide/docs/ApplicationDev/01-FabricGateway.md

