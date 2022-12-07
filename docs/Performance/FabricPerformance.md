# (DRAFT) Hyperledger Fabric 2.5 Performance Considerations (DRAFT)

Hyperledger Fabric performance is a question that comes up frequently as users try to compare it to transactional databases or other blockchain platforms in terms of the maximum TPS (Transactions Per Second). Performance in a Hyperledger Fabric network is complex because in an appropriately deployed network there will be many organisations participating, each with their own hardware and networking infrastructure, along with different solution characteristics such as number of channels, chaincode implementations and policies.

This article will provide an introduction into the considerations needed to improve performance of a Hyperledger Fabric network.

## Fabric Version

Fabric 2.x has performance improvements over Fabric 1.4. Fabric 1.4 is now out of LTS and should not be used in production environments. Fabric 2.5 is the latest LTS version and includes the new Peer Gateway service. When used with the new gateway SDKs, applications will demonstrate improved performance relative to applications based on the legacy SDKs.

## Hardware considerations

Each organisation may provide it's own hardware to host their peer nodes and ordering service nodes. Ordering service nodes may be provided by a single organisation, or may be contributed from various organisations. It's important to realise that individual organisations could impact the whole network (depending on policies such as endorsement policies), therefore each organisation must ensure that they provide adequate resources for the services they deploy.

As Hyperledger Fabric performs a lot of disk I/O it goes without saying that you should ensure you are using the fastest disk storage available to you and that the physical drive it ultimately resides on should be SSD.

A Hyperledger Fabric network is highly distributed across multiple nodes usually hosted on multiple different clusters, on different clouds and even in different countries, therefore high speed network connectivity between nodes is vital and at least 1Gbs should be deployed between all nodes/organisations.

The final piece is how much CPU and Memory should be allocated to peer and ordering service nodes, and if used for peer nodes, CouchDB state databases. There is no real answer to this one but it does affect the performance of the network and even the stability of the nodes if not enough resources are allocated. It's vital therefore that you constantly monitor the CPU, Memory, Disk Space, Disk and Network IO to ensure that you are within the limits of your allocated resources. Do not wait until you see these values consistently hitting 99% utilization before deciding to increase the resources.  A threshold of between 70% and 80% is a good starting point for the maximum expected workload.

Performance generally scales with CPU allocated to peer nodes. Providing each peer and CouchDB (if used) with the maximum CPU capacity is recommended. For orderer nodes a good starting point may be 1 CPU with 2GB of memory.

As the amount of state data grows the database storage may slow down, especially when using CouchDB as the state database. Therefore you may need to add more compute resource to the environment over the life of your Fabric deployment, which just re-emphasises the point that you must monitor your network components and react to any thresholds being exceeded.

## Peer Configuration Considerations

Although covered later it is important to note that using CouchDB for the state database will have a noticeable impact on performance. Refer to the CouchDB considerations section later for more information.

This section highlights some of the considerations for a peer such as the number of active channels and node configuration properties. The default peer and orderer configurations from the peer core.yaml and orderer orderer.yaml are referenced below.

### Number of channels a peer is participating in

If a peer has ample CPU and is only participating in a single channel then it's not possible to drive the peer beyond around 65-70% CPU. This is due to the way the peer does serialisation and locking internally. Multiple active channels on a peer will use the extra resource as it increases parallelism in block processing.

In general, ensure that there is a CPU core available for each channel that is running at maximum load. It is not a hard limit however, if the load in each channel is not highly correlated (i.e., not every channel is contenting for resources at the same time), the numbers of channels can exceed the number of CPU cores.

### Total Query Limit

A Peer limits the total number of records a range or JSON (rich) query will ever return. This is configured in the core.yaml file of a peer

```yaml

ledger:
  state:
    # Limit on the number of records to return per query
    totalQueryLimit: 100000
```

100000 is the default that Fabric provides in the sample core.yaml and in the test docker images. If you are exceeding this limit, you can increase this value but I would suggest that if you are exceeding this limit that the query is returning too many records and you should look to keeping the range within the limit

### Concurrency limits

The peer has some limits to ensure a peer cannot be overwhelmed by clients:

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

The new Peer Gateway service introduced a limit in the `gatewayService` and the default in the sample configuration and test docker images is 500. However this will restrict the TPS of a network so you may need to increase this value. However don't set it too high as it ensures that the peer doesn't get overloaded with requests causing the TPS to actually drop. For high throughput I used values up to 20000 for performance benchmarking.

### CouchDB Cache setting

If you are using CouchDB and have a large number of keys being read (not via queries) then you may want to investigate increasing the CouchDB cache to avoid database lookups:

```yaml

state:
    couchDBConfig:
       # CacheSize denotes the maximum mega bytes (MB) to be allocated for the in-memory state
       # cache. Note that CacheSize needs to be a multiple of 32 MB. If it is not a multiple
       # of 32 MB, the peer would round the size to the next multiple of 32 MB.
       # To disable the cache, 0 MB needs to be assigned to the cacheSize.
       cacheSize: 64
```

## Orderer Considerations

The ordering service uses raft concensus to cut blocks, factors such as the number of orderers in concensus and block cutting parameters will affect performance

### Number of orderers

The more ordering service nodes you have will impact performance as all orderers are involved in Raft consensus. Five ordering service nodes will provide two orderer crash fault tolerance (a majority must remain available) and is a good starting point. Additional ordering service nodes may reduce performance. You have the option to deploy different sets of ordering service nodes for each channel if you find that a single set of ordering service nodes becomes a bottleneck.

### SendBufferSize

Prior to Fabric v2.5 the SendBufferSize of the orderer was set to 10 as a default which causes a bottleneck. In Fabric 2.5 this was changed to 100 which provides much better throughput. The configuration for this can be found in orderer.yaml:

```yaml
General:
    Cluster:
        # SendBufferSize is the maximum number of messages in the egress buffer.
        # Consensus messages are dropped if the buffer is full, and transaction
        # messages are waiting for space to be freed.
        SendBufferSize: 100
```

### Block cutting parameters in channel configuration

A larger block size and timeout could increase the throughput (but latency would increase as well). The block size is highly related to the transaction arrival rate. For example, if the transaction arrival rate is 100 per second and block size and timeout is 3000 & 10 secs, respectively, the throughput would be 10 tx per second. If the arrival rate is higher for the same configuration, the throughput would be higher.

You might want to try to configure the ordering service with more transactions per block and longer block cutting times to see if that helps. We have seen this increase the overall throughput at the cost of additional latency.

The following three parameters work together to control when a block is cut, based on a combination of setting the maximum number of transactions in a block as well as the block size itself. These are defined when you create or update a channel configuration. If you use configtxgen and configtx.yaml as a starting point for creating channels then the following section applies in configtx.yaml

```yaml
Orderer: &OrdererDefaults
    # Batch Timeout: The amount of time to wait before creating a batch.
    BatchTimeout: 2s

    # Batch Size: Controls the number of messages batched into a block.
    # The orderer views messages opaquely, but typically, messages may
    # be considered to be Fabric transactions.  The 'batch' is the group
    # of messages in the 'data' field of the block.  Blocks will be a few kb
    # larger than the batch size, when signatures, hashes, and other metadata
    # is applied.
    BatchSize:

        # Max Message Count: The maximum number of messages to permit in a
        # batch.  No block will contain more than this number of messages.
        MaxMessageCount: 500

        # Absolute Max Bytes: The absolute maximum number of bytes allowed for
        # the serialized messages in a batch. The maximum block size is this value
        # plus the size of the associated metadata (usually a few KB depending
        # upon the size of the signing identities). Any transaction larger than
        # this value will be rejected by ordering.
        # It is recommended not to exceed 49 MB, given the default grpc max message size of 100 MB
        # configured on orderer and peer nodes (and allowing for message expansion during communication).
        AbsoluteMaxBytes: 10 MB

        # Preferred Max Bytes: The preferred maximum number of bytes allowed
        # for the serialized messages in a batch. Roughly, this field may be considered
        # the best effort maximum size of a batch. A batch will fill with messages
        # until this size is reached (or the max message count, or batch timeout is
        # exceeded).  If adding a new message to the batch would cause the batch to
        # exceed the preferred max bytes, then the current batch is closed and written
        # to a block, and a new batch containing the new message is created.  If a
        # message larger than the preferred max bytes is received, then its batch
        # will contain only that message.  Because messages may be larger than
        # preferred max bytes (up to AbsoluteMaxBytes), some batches may exceed
        # the preferred max bytes, but will always contain exactly one transaction.
        PreferredMaxBytes: 2 MB
```

#### Absolute max bytes

Set this value to the largest block size in bytes that can be cut by the orderer. No transaction may be larger than the value of Absolute max bytes. Usually, this setting can safely be two to ten times larger than your Preferred max bytes. Note: The maximum recommended size is 49MB based on the headroom needed for the default grpc size limit of 100MB.

#### Max message count

Set this value to the maximum number of transactions that can be included in a single block.

#### Preferred max bytes

Set this value to the ideal block size in bytes, but it must be less than Absolute max bytes. A minimum transaction size, one that contains no endorsements, is around 1KB. If you add 1KB per required endorsement, a typical transaction size is approximately 3-4KB. Therefore, it is recommended to set the value of Preferred max bytes to be around Max message count times expected averaged transaction size. At run time, whenever possible, blocks will not exceed this size. If a transaction arrives that causes the block to exceed this size, the block is cut and a new block is created for that transaction. But if a transaction arrives that exceeds this value without exceeding the Absolute max bytes, the transaction will be included. If a transaction arrives that is larger than Preferred max bytes, then a block will be cut with a single transaction, and that transaction size can be no larger than Absolute max bytes.

Together, these parameters can be configured to optimize throughput of your orderer.

#### Batch timeout

Set the Timeout value to the amount of time, in seconds, to wait after the first transaction arrives before cutting the block. If you set this value too low, you risk preventing the batches from filling to your preferred size. Setting this value too high can cause the orderer to wait for blocks and overall performance to degrade. In general, we recommend that you set the value of Batch timeout to be at least max message count / maximum transactions per second

## Application Considerations

When designing the application architecture, decisions can affect the overall performance of the network and the application. Here we cover some of the considerations.

### Avoid CouchDB for high throughput applications

CouchDB performance is noticably slower than embedded LevelDB sometimes by a factor of 2x slower. The only capability that you get by choosing CouchDB is JSON (rich) queries as stated in the [Fabric state database documentation](https://hyperledger-Fabric.readthedocs.io/en/release-2.5/deploypeer/peerplan.html#state-database). You should also never allow direct access to the CouchDB data (for instance via Fauxton UI) to ensure the integrity of the state data.

CouchDB also has other limitations which will have impacts on performance, extra hardware resource required to provide CouchDB for each peer (which incurs additional costs) and no extra protection from Fabric (JSON queries are not re-executed at validation time and thus you don't get any Phantom Protection from Fabric as you do with range queries).

If at all possible consider looking at an off-chain store to support queries for which Fabric provides a sample [Off-chain sample](https://github.com/hyperledger/Fabric-samples/tree/main/off_chain_data) to demonstrate exactly this. Doing so gives you much more control over the data and query transactions do not affect the Fabric network performance. It also enables you to use a fit-for-purpose data store for off-chain storage, for example you could use a SQL database or an analytics service depending on your query needs.

CouchDB performance degrades more than LevelDB as the amount of state data increases, requiring you to provide adequate resources for CouchDB instances for the life of the application.

### Use the new Peer Gateway Service

The Peer Gateway Service and the new Fabric-Gateway client SDKs are a substantial improvement over the legacy Go, Java and Node SDKs. Not only do they provide much improved throughput, they also provide better capability reducing the complexity of a client application. For example the Gateway SDKs will automatically collect enough endorsements to satisfy not only the chaincode endorsement policy but also any state-based endorsement policies that get included when the transaction is simulated, something that was not possible with the legacy SDKs.

It also reduces the number of network connections a client needs to maintain in order for a client to submit a transaction. Previously clients may need to connect to multiple peer and orderer nodes across organizations. The peer Gateway Service service enables an application to target a single trusted peer, then the peer Gateway Service connects to other peer and orderer nodes to gather endorsements and submit the transaction on behalf of the client application. Of course, you may want to target multiple trusted peers for high concurrency and redundancy.

One point to remember that if you used the legacy SDKs then a client requires more CPU and memory than a client using the new Peer Gateway service so this reduces the client resource requirements. However, it does increase the peer resource requirements slightly.

See https://github.com/hyperledger/Fabric-samples/blob/main/full-stack-asset-transfer-guide/docs/ApplicationDev/01-FabricGateway.md

for information about how the Peer Gateway Service differs from the legacy SDKs.

### Payload size

The amount of data that is submitted to a transaction, along with the amount of data written to keys in a transaction will affect the application performance. Note that the payload size includes more than just the data. It includes structures required by Fabric plus client and endorsing peer signatures.

Suffice to say large payload sizes are an anti-pattern in any blockchain solution. Considering storing large data off-chain and storing a hash of the data on-chain.

### Chaincode language

Go chaincode performs best, followed by Node chaincode.  Java chaincode performance is the least performant and would not be recommended for high throughput applications.

### Endorsement policies

For a transaction to be committed as valid, one of the things it must contain is enough signatures to satisfy the chaincode endorsement policy and any state-based endorsement policies. The Peer Gateway service will only send requests to enough peers to satisfy this collection of policies (and will also try other peers if the preferred ones are not available). Thus we can see that endorsement policies will affect performance as it dictates how many peers and thus how many signatures are required to ensure this transaction could be committed.

## Couchdb considerations

As mentioned earlier CouchDB is not recommended for high throughput applications, but if you do plan to use it these are the things that need to be considered.

### Resources

Ensure you monitor the resources of the CouchDB instances, as the larger the state database becomes the more resources CouchDB will consume.

### CouchDB Cache

When using external CouchDB state database, read delays during endorsement and validation phases have historically been a performance bottleneck. In Fabric v2.x, the peer cache replaces many of these expensive lookups with fast local cache reads.

The cache will not improve performance of JSON queries.

See `CouchDB Cache setting` in the Peer Considerations section for information on configuring the cache.

### Indexes

Ensure you use indexes and don't use queries that can't use indexes. For example use of the query operators $or, $in, $regex result in full data scans. See [Hyperledger Fabric Good Practive For Queries](https://hyperledger-Fabric.readthedocs.io/en/release-2.5/couchdb_as_state_database.html?highlight=%24regex#good-practices-for-queries)

Optimise your queries, complex queries will take more time even with indexing and ensure your queries result in a bounded set of data. Remember Fabric may also limit the total number of results returned.

You can check peer and CouchDB logs to see how long queries are taking and also whether a query was unable to use an index from warning log entries stating the query "should be indexed".

If queries are taking a long time, you could try to increase CPU/Memory available to CouchDB or use faster storage.

Check CouchDB logs for warnings such as "The number of documents examined is high in proportion to the number of results returned. Consider adding a more specific index to improve this." These messages show you that your indexes may not be good enough for the query being performed as it is resulting in too many documents having to be scanned.

### Bulk Update

Fabric uses bulk update calls to CouchDB to improve CouchDB performance. A bulk update is done at the block level so including more transactions in a block could potentially improve throughput. However increasing the time before a block is cut to include more transactions will have an impact on latency.

## HSM

using HSMs within a Fabric network will have an impact on performance. It's not possible to quantify the impact here but things like the performance of the HSM and the network connection to the HSM will impact the Fabric network.

If you have configured Peers, Orderers, Clients to use an HSM then anything that requires signing such as blocks by orderers, endorsements and block events by peers and all client requests will be done by the HSM so you can see that the HSM is involved in all the major activities that the nodes perform.

HSMs are NOT involved in the verification of signatures. This is still done by the nodes themselves.

## Other Miscellaneous considerations

Sending single transactions periodically will have a latency correlating to the block cutting parameters. For example if you send a transaction of 100 Bytes and the BatchTimeout is 2 seconds then the time from submission to being committed will be just over 2 seconds. This is not a true benchmark of your Fabric network performance, it's expected that multiple transactions will be submitted simultaneously to gauge the true performance of a Fabric network.

## Performance runs

The following describes some initial performance runs on Hyperledger Fabric 2.5

### Hardware and Topology

The Fabric topology used was 2 peer Organisations (PeerOrg0, PeerOrg1) with a single peer node in each and 1 Ordering Service Organisation (OrdererOrg0) with a single orderer service node configured for Raft.

Each node had the same identical hardware

- Intel(R) Xeon(R) Silver 4210 CPU @ 2.20GHz
- 40 Cores made up of 2 CPUs. Each CPU has 10 physical cores supporting 20 Threads in total
- 64Gb Samsung 2933Mhx Memory
- MegaRAID Tri-Mode SAS3516 (MR9461-16i) disk controller
- Intel 730 and DC S35x0/3610/3700 Series SSD attached to disk controller
- Ethernet Controller X710/X557-AT 10GBASE-T
- Ubuntu 20.04

The machines were all on the same switch.

Hyperledger Fabric was deployed natively to 3 physical machines (ie the native binaries were installed and executed, no container technology such as Docker or Kubernetes was used)

### Fabric Application Configuration

- LevelDB was used for the state database
- Gateway Service Concurrency limit was set to 20,000
- A Single Channel was created
- Go chaincode without the Contract API was deployed (fixed-asset-base from hyperledger Caliper-Benchmarks)
- Endorsement policy 1 Of Any was specified for the chaincode
- No private data was used
- Standard OOTB Fabric Policies and configurations (note that in 2.5 SendBufferSize now defaults to 100) excluding anything previously mentioned
- No tests were done that included Range Queries (and obviously no JSON Queries)

### Load Generator

Hyperledger Caliper 0.5.0 was used as the load generator and for the report output. Caliper was bound to `Fabric:2.4` which means it used the Peer Gateway Service to invoke and evaluate transactions.

The load itself was defined from fixed-asset in Hyperledger Caliper-Benchmarks

Caliper used between 3 and 4 bare metal machines to host remote caliper workers to generate the load on the Hyperledger Fabric Network.

### Results

These results were examples of quick runs I made using Fabric 2.4 and block cutting values of

- block_cut_time: 1s
- block_size: 50
- preferred_max_bytes: 512 KB

they will be updated when the 2.5 results are done which will use OOTB block cutting parameters and will also include 1000 byte asset size.

#### No-op evaluate benchmark

- workers: (Not captured)
- fixed-tps, tps: (Not captured)

This benchmark determines the theoretical maximum a network. The chaincode transaction is a no-op and does nothing except return. The Evaluate means that the send to orderer, block cutting, validate and commit processing never engages. This type of benchmark is useful for determining things like network and some hardware bottlenecks

```bash
+----------------+---------+------+-----------------+-----------------+-----------------+-----------------+------------------+
| Name           | Succ    | Fail | Send Rate (TPS) | Max Latency (s) | Min Latency (s) | Avg Latency (s) | Throughput (TPS) |
|----------------|---------|------|-----------------|-----------------|-----------------|-----------------|------------------|
| no-op-evaluate | 2160240 | 0    | 17917.3         | 0.34            | 0.00            | 0.05            | 17916.0          |
+----------------+---------+------+-----------------+-----------------+-----------------+-----------------+------------------+
```

#### Blind Write of a single key ~100 Byte Asset Size

A Blind write is a transaction that performs a single write to a key regardless of whether that key exists and contains data. This is a `Create Asset` type of scenario

- workers: (Not captured)
- fixed-tps, tps: (Not captured)

```bash
+------------------+--------+------+-----------------+-----------------+-----------------+-----------------+------------------+
| Name             | Succ   | Fail | Send Rate (TPS) | Max Latency (s) | Min Latency (s) | Avg Latency (s) | Throughput (TPS) |
|------------------|--------|------|-----------------|-----------------|-----------------|-----------------|------------------|
| create-asset-100 | 360150 | 0    | 2996.2          | 2.04            | 0.28            | 0.73            | 2983.6           |
+------------------+--------+------+-----------------+-----------------+-----------------+-----------------+------------------+
```

#### Read Write of a single key ~100 Byte Asset Size

This is a test where the transaction will randomly pick an already existing key with data, read it, then modify that key. In the results here you will see some failures.

The world state was loaded with 1 million assets for this test. The results will show some failures however these are MVCC_READ_CONFLICTS and this is to be expected even with a large number of assets preloaded.

- workers: (Not captured)
- fixed-tps, tps: 2750

```bash
+---------------------------------------+--------+------+-----------------+-----------------+-----------------+-----------------+------------------+
| Name                                  | Succ   | Fail | Send Rate (TPS) | Max Latency (s) | Min Latency (s) | Avg Latency (s) | Throughput (TPS) |
|---------------------------------------|--------|------|-----------------|-----------------|-----------------|-----------------|------------------|
| read-write-assets-previously-read-100 | 324105 | 45   | 2696.4          | 0.51            | 0.03            | 0.09            | 2694.5           |
+---------------------------------------+--------+------+-----------------+-----------------+-----------------+-----------------+------------------+
```

### Future considerations

- maybe experiment with reducing logging (removes response comparison as well but would need an appropriate endorsement policy)
- maybe experiment with block cutting parameters
- maybe experiment with ValidatorPoolSize: only > no. of cores makes sense
- maybe experiment with no tls and solo orderer (can't use main as solo has been removed)
- Gateway request limit (I used 10000, 20000): it's good to note that on blind writes you can see the backlog of unfinished transactions being limited to 20000 but it will also register failed transactions as they are rejected
