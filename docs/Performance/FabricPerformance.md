# (DRAFT) Hyperledger Fabric 2.5 Performance Considerations (DRAFT)

Hyperledger Fabric performance is a question that comes up frequently as users try to compare it to transactional databases or other blockchain platforms in terms of the maximum TPS (Transactions Per Second). Performance in a Hyperledger Fabric network is complex because in an appropriately deployed network there will be many organisations participating, each with their own hardware and networking infrastructure, along with different solution characteristics such as number of channels, chaincode implementations and policies.

This article will provide an introduction into the considerations needed to improve performance of a Hyperledger Fabric network.

## Fabric Version

Fabric 2.x has performance improvements over Fabric 1.4. Fabric 1.4 is now out of LTS and should not be used in production environments. Fabric 2.5 is the latest LTS version and includes the new peer Gateway service. When used with the new gateway SDKs, applications will demonstrate improved performance relative to applications based on the legacy SDKs.

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

A peer limits the total number of records a range or JSON (rich) query will return in order to avoid runaway situations. This is configured in the core.yaml file of a peer:

```yaml

ledger:
  state:
    # Limit on the number of records to return per query
    totalQueryLimit: 100000
```

100000 is the default that Fabric provides in the sample core.yaml and in the test docker images. If you are exceeding this limit, you can increase this value, however you should consider alternative designs that don't require such extensive query scans.

### Concurrency limits

The peer has some limits to ensure a peer cannot be overwhelmed by excessive concurrent client requests:

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

The new peer Gateway service introduced a limit in the `gatewayService` with a default of 500. However, this may restrict the TPS of a network so you may need to increase this value to allow more concurrent requests. Good results have been seen with 20000 concurrent requests, higher values may be counter-productive.

### CouchDB Cache setting

If you are using CouchDB and have a large number of keys being read repeatedly (not via queries) then you may want to increase the peer's CouchDB cache to avoid database lookups:

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

The ordering service uses raft consensus to cut blocks, factors such as the number of orderers in consensus and block cutting parameters will affect performance.

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

The following three parameters work together to control when a block is cut, based on a combination of setting the maximum number of transactions in a block as well as the block size itself. These are defined when you create or update a channel configuration. If you use configtxgen and configtx.yaml as a starting point for creating channels then the following section applies in configtx.yaml:

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

Set this value to the largest block size in bytes that can be cut by the ordering service. No transaction may be larger than the value of Absolute max bytes. Usually, this setting can safely be two to ten times larger than your Preferred max bytes. Note: The maximum recommended size is 49MB based on the headroom needed for the default grpc size limit of 100MB.

#### Max message count

Set this value to the maximum number of transactions that can be included in a single block.

#### Preferred max bytes

Set this value to the ideal block size in bytes, but it must be less than Absolute max bytes. A minimum transaction size, one that contains no endorsements, is around 1KB. If you add 1KB per required endorsement, a typical transaction size is approximately 3-4KB. Therefore, it is recommended to set the value of Preferred max bytes to be around Max message count times expected averaged transaction size. At run time, whenever possible, blocks will not exceed this size. If a transaction arrives that causes the block to exceed this size, the block is cut and a new block is created for that transaction. But if a transaction arrives that exceeds this value without exceeding the Absolute max bytes, the transaction will be included. If a transaction arrives that is larger than Preferred max bytes, then a block will be cut with a single transaction, and that transaction size can be no larger than Absolute max bytes.

Together, these parameters can be configured to optimize throughput of your orderer.

#### Batch timeout

Set the Timeout value to the amount of time, in seconds, to wait after the first transaction arrives before cutting the block. If you set this value too low, you risk preventing the batches from filling to your preferred size. Setting this value too high can cause the orderer to wait for blocks and overall performance and latency to degrade. In general, we recommend that you set the value of Batch timeout to be at least max message count / maximum transactions per second.

## Application Considerations

When designing the application architecture, decisions can affect the overall performance of the network and the application. Here we cover some of the considerations.

### Avoid CouchDB for high throughput applications

CouchDB performance is noticably slower than embedded LevelDB, sometimes by a factor of 2x slower. The only additional capability that a CouchDB state database provides is JSON (rich) queries as stated in the [Fabric state database documentation](https://hyperledger-Fabric.readthedocs.io/en/release-2.5/deploypeer/peerplan.html#state-database). You should also never allow direct access to the CouchDB data (for instance via Fauxton UI) to ensure the integrity of the state data.

CouchDB as a state database also has other limitations which will have impacts on performance and requires additional hardware resource (and cost). Additionally, JSON queries are not re-executed at validation time and thus there is no built-in protection from phantom reads as there is with range queries (your application must be designed to tolerate phantom reads when using JSON queries, for example when records are added to state between the time of chaincode execution and block validation). For these reasons, consider using range queries based on additional keys rather than CouchDB JSON queries.

Alternatively, consider using an off-chain store to support queries, as seen in the [Off-chain sample](https://github.com/hyperledger/Fabric-samples/tree/main/off_chain_data). Using an off-chain store for queries gives you much more control over the data and query transactions do not affect the Fabric peer and network performance. It also enables you to use a fit-for-purpose data store for off-chain storage, for example you could use a SQL database or an analytics service more aligned with your query needs.

CouchDB performance degrades more than LevelDB as the amount of state data increases, requiring you to provide adequate resources for CouchDB instances for the life of the application.

### Use the new Peer Gateway Service

The peer Gateway Service and the new Fabric-Gateway client SDKs are a substantial improvement over the legacy Go, Java and Node SDKs. Not only do they provide much improved throughput, they also provide better capability reducing the complexity of a client application. For example the Gateway SDKs will automatically collect enough endorsements to satisfy not only the chaincode endorsement policy but also any state-based endorsement policies that get included when the transaction is simulated, something that was not possible with the legacy SDKs.

It also reduces the number of network connections a client needs to maintain in order for a client to submit a transaction. Previously clients may need to connect to multiple peer and orderer nodes across organizations. The peer Gateway Service service enables an application to target a single trusted peer, then the peer Gateway Service connects to other peer and orderer nodes to gather endorsements and submit the transaction on behalf of the client application. Of course, you may want to target multiple trusted peers for high concurrency and redundancy.

One point to consider is that shifting from legacy SDKs to the new Peer Gateway service reduces the client CPU and memory resource requirements. However, it does increase the peer resource requirements slightly.

See the [Sample gateway application](https://github.com/hyperledger/Fabric-samples/blob/main/full-stack-asset-transfer-guide/docs/ApplicationDev/01-FabricGateway.md) for more details about the new peer Gateway Service.

### Payload size

The amount of data that is submitted to a transaction, along with the amount of data written to keys in a transaction will affect the application performance. Note that the payload size includes more than just the data. It includes structures required by Fabric plus client and endorsing peer signatures.

Suffice to say large payload sizes are an anti-pattern in any blockchain solution. Consider storing large data off-chain and storing a hash of the data on-chain.

### Chaincode language

Go chaincode performs best, followed by Node chaincode.  Java chaincode performance is the least performant and would not be recommended for high throughput applications.

### Node chaincode

Node is an asynchronous runtime implementation that utilises only a single thread to execute code. It does however run background threads for activities such as garbage collection, however when allocating resources to a Node chaincode, for example in Kubernetes where you are limited to available resources, it doesn't make sense to allocate multiple vCPUs for Node chaincode. The number of vCPUs in a system usually refers to the number of concurrent threads that can be executed. It is worth monitoring performance of Node chaincode to see how much vCPU it uses but it probably doesn't make sense to allocate anything more than a max of 2 vCPUs for Node chaincode. In fact you could not assign any resource restrictions to Node chaincode as it is self limiting.

Prior to Node 12, a Node process was limited to 1.5Gb Memory by default and would require you to pass a parameter to the Node executable in order to increase this when running a node chaincode process. You should not be running Node chaincode processes on anything less than Node 12 now and Hyperledger Fabric 2.5 mandates that Node 16 or later should be used. There are various parameters that can be provided to the Node process when you launch your Node chaincode, however it's unlikely you would ever need to override the defaults of Node so no tuning would be required.

### Go chaincode

The Golang runtime provides an excellent implementation for concurrency. It is capable of using all CPUs available to it and thus is only limited by the resources allocated to the chaincode process to use. No tuning is required.

### Chaincode processes and channels

Hyperledger Fabric will reuse chaincode processes across channels if the chaincode id and versions match. For example if you have a peer joined to 2 channels (channel_a and channel_b) and you have deployed one chaincode to each channel with the same id and version number, then the peer will only interact with 1 chaincode process for both those channels. It will not try to work with a separate chaincode process for each channel. This means that you may be putting more load on that chaincode process than expected especially if it's node chaincode that is self limiting. If this is a problem you should consider using Go for your chaincode language or you could deploy the same chaincode with a different id or version to the other channel and that will ensure there is a chaincode process per channel.

### Endorsement policies

For a transaction to be committed as valid, it must contain enough signatures to satisfy the chaincode endorsement policy and any state-based endorsement policies. The peer Gateway service will only send requests to enough peers to satisfy this collection of policies (and will also try other peers if the preferred ones are not available). Thus we can see that endorsement policies will affect performance as it dictates how many peers and thus how many signatures are required to ensure that a transaction can be committed.

## Couchdb considerations

As mentioned earlier CouchDB is not recommended for high throughput applications, but if you do plan to use it these are the things that need to be considered.

### Resources

Ensure you monitor the resources of the CouchDB instances, as the larger the state database becomes the more resources CouchDB will consume.

### CouchDB Cache

When using external CouchDB state database, read delays during endorsement and validation phases have historically been a performance bottleneck. In Fabric v2.x, the peer cache replaces many of these expensive lookups with fast local cache reads.

The cache will not improve performance of JSON queries.

See `CouchDB Cache setting` in the Peer Considerations section for information on configuring the cache.

### Indexes

Ensure you use indexes and don't use queries that can't use indexes. For example use of the query operators $or, $in, $regex result in full data scans. See [Hyperledger Fabric Good Practices For Queries](https://hyperledger-fabric.readthedocs.io/en/release-2.5/couchdb_as_state_database.html#good-practices-for-queries)

Optimise your queries, complex queries will take more time even with indexing. Ensure your queries result in a bounded set of data. Remember Fabric may also limit the total number of results returned.

You can check peer and CouchDB logs to see how long queries are taking and also whether a query was unable to use an index from warning log entries stating the query "should be indexed".

If queries are taking a long time, you could try to increase CPU/Memory available to CouchDB or use faster storage.

Check CouchDB logs for warnings such as "The number of documents examined is high in proportion to the number of results returned. Consider adding a more specific index to improve this." These messages show you that your indexes may not be good enough for the query being performed as it is resulting in too many documents having to be scanned.

### Bulk Update

Fabric uses bulk update calls to CouchDB to improve CouchDB performance. A bulk update is done at the block level so including more transactions in a block could potentially improve throughput. However increasing the time before a block is cut to include more transactions will have an impact on latency.

## HSM

Using HSMs within a Fabric network will have an impact on performance. It's not possible to quantify the impact here but things like the performance of the HSM and the network connection to the HSM will impact the Fabric network.

If you have configured Peers, Orderers, Clients to use an HSM then anything that requires signing such as blocks by orderers, endorsements and block events by peers and all client requests will be done by the HSM so you can see that the HSM is involved in all the major activities that the nodes perform.

HSMs are NOT involved in the verification of signatures. This is still done by the nodes themselves.

## Other Miscellaneous considerations

Sending single transactions periodically will have a latency correlating to the block cutting parameters. For example if you send a transaction of 100 Bytes and the BatchTimeout is 2 seconds then the time from submission to being committed will be just over 2 seconds. This is not a true benchmark of your Fabric network performance, it's expected that multiple transactions will be submitted simultaneously to gauge the true performance of a Fabric network.

## Performance benchmarks

The following describes some initial performance benchmarks on Hyperledger Fabric 2.5 builds.

### Hardware and Topology

The Fabric topology used was 2 peer Organisations (PeerOrg0, PeerOrg1) with a single peer node in each and 1 Ordering Service Organisation (OrdererOrg0) with a single orderer service node configured for Raft. TLS was enabled for each node.

Each node had the same identical hardware

- Intel(R) Xeon(R) Silver 4210 CPU @ 2.20GHz
- 40 Cores made up of 2 CPUs. Each CPU has 10 physical cores supporting 20 Threads in total
- 64Gb Samsung 2933Mhx Memory
- MegaRAID Tri-Mode SAS3516 (MR9461-16i) disk controller
- Intel 730 and DC S35x0/3610/3700 Series SSD attached to disk controller
- Ethernet Controller X710/X557-AT 10GBASE-T
- Ubuntu 20.04

The machines were all on the same switch.

Hyperledger Fabric was deployed natively to 3 physical machines (ie the native binaries were installed and executed, no container technology such as Docker or Kubernetes was used).

### Fabric Application Configuration

- LevelDB was used for the state database
- Gateway Service Concurrency limit was set to 20,000
- A single application channel was created and the 2 peers and orderer were joined to this channel
  - The application capabilities were set to V1_4 so as to use the old lifecycle deployment. All other capabilities were set to V2_0 (capability level should not impact performance).
- No system channel exists only the application channel
- Go chaincode without the Contract API was deployed (fixed-asset-base from hyperledger Caliper-Benchmarks)
- Endorsement policy 1 Of Any was specified for the chaincode
- No private data was used
- Default Fabric policies and configurations (note that in 2.5 SendBufferSize now defaults to 100) excluding anything previously mentioned
- No range queries or JSON queries

### Load Generator

Hyperledger Caliper 0.5.0 was used as the load generator and for the report output. Caliper was bound to `Fabric:2.4` which means it used the peer Gateway Service to invoke and evaluate transactions.

The load itself was defined from fixed-asset in Hyperledger Caliper-Benchmarks.

Caliper used 4 bare metal machines to host remote caliper workers and also to host a single caliper manager to generate the load on the Hyperledger Fabric Network.

### Diagram of overall Topology

![Fabric Performance Run Topology](FabricPerformanceTopology.jpg)

### Results

These results were generated against the latest builds of Hyperledger Fabric 2.5 and utilised the [default node and channel config values](https://github.com/hyperledger/fabric/tree/release-2.5/sampleconfig). Specifically the following block cutting parameters were used:

- block_cut_time: 2s
- block_size: 500
- preferred_max_bytes: 2Mb

In order to be able to push enough workload through without hitting concurrency limits, the gateway concurrency limit was set to 20,000.

In summary the following benchmarks are presented here:

- Blind write of a single key with 100 Byte Asset Size (a create asset benchmark)
- Blind write of a single key with 1000 Byte Asset Size (a create asset benchmark)
- Read/Write of a single key with 100 Byte Asset Size (an update asset benchmark)
- Read/Write of a single key with 1000 Byte Asset Size (an update asset benchmark)

#### Blind Write of a single key 100 Byte Asset Size

A Blind write is a transaction that performs a single write to a key regardless of whether that key exists and contains data. This is a `Create Asset` type of scenario.

Caliper test configuration:
- workers: 200
- fixed-tps, tps: 3000

```bash
+------------------+--------+------+-----------------+-----------------+-----------------+-----------------+------------------+
| Name             | Succ   | Fail | Send Rate (TPS) | Max Latency (s) | Min Latency (s) | Avg Latency (s) | Throughput (TPS) |
|------------------|--------|------|-----------------|-----------------|-----------------|-----------------|------------------|
| create-asset-100 | 360200 | 0    | 2995.0          | 2.13            | 0.18            | 0.33            | 2946.7           |
+------------------+--------+------+-----------------+-----------------+-----------------+-----------------+------------------+
```

#### Blind Write of a single key 1000 Byte Asset Size

Caliper test configuration:
- workers: 200
- fixed-tps, tps: 3000

```bash
+------------------+--------+------+-----------------+-----------------+-----------------+-----------------+------------------+
| Name             | Succ   | Fail | Send Rate (TPS) | Max Latency (s) | Min Latency (s) | Avg Latency (s) | Throughput (TPS) |
|------------------|--------|------|-----------------|-----------------|-----------------|-----------------|------------------|
| create-asset-100 | 360200 | 0    | 2993.9          | 3.21            | 0.28            | 1.52            | 2938.9           |
+------------------+--------+------+-----------------+-----------------+-----------------+-----------------+------------------+
```

Here we see that we can achieve roughly the same throughput but latency increases.

#### Read Write of a single key 100 Byte Asset Size

This is a test where the transaction will randomly pick an already existing key with data, read it, then modify that key. The world state was loaded with 1 million assets for this test to reduce the chance of using the same key in two concurrent transactions resulting in MVCC_READ_CONFLICT validation errors. In this example the TPS rate was low enough and fortunate that no MVCC_READ_CONFLICT validation errors were received.

Caliper test configuration:
- workers: 200
- fixed-tps, tps: 2550

```bash
+---------------------------------------+--------+------+-----------------+-----------------+-----------------+-----------------+------------------+
| Name                                  | Succ   | Fail | Send Rate (TPS) | Max Latency (s) | Min Latency (s) | Avg Latency (s) | Throughput (TPS) |
|---------------------------------------|--------|------|-----------------|-----------------|-----------------|-----------------|------------------|
| read-write-assets-previously-read-100 | 306200 | 0    | 2545.2          | 0.21            | 0.03            | 0.06            | 2544.3           |
+---------------------------------------+--------+------+-----------------+-----------------+-----------------+-----------------+------------------+
```

#### Read Write of a single key 1000 Byte Asset Size

The above was repeated using a 1000 byte asset size.

Caliper test configuration:
- workers: 200
- fixed-tps, tps: 1530

```bash
+----------------------------------------+--------+------+-----------------+-----------------+-----------------+-----------------+------------------+
| Name                                   | Succ   | Fail | Send Rate (TPS) | Max Latency (s) | Min Latency (s) | Avg Latency (s) | Throughput (TPS) |
|----------------------------------------|--------|------|-----------------|-----------------|-----------------|-----------------|------------------|
| read-write-assets-previously-read-1000 | 183800 | 0    | 1527.7          | 0.26            | 0.04            | 0.11            | 1527.0           |
+----------------------------------------+--------+------+-----------------+-----------------+-----------------+-----------------+------------------+
```

The above results are probably at the limit of the fabric network under test and that latency is really above an acceptable threshold.

## Acknowledgements

I would like to thank Shivdeep Singh for running the Hyperledger Caliper benchmarks to get the results presented in this blog.

Thanks also to Senthilnathan Natarajan for the initial performance investigations done on Hyperledger Fabric.

Finally, thanks to Dave Enyeart for reviewing and providing feedback on this blog post.
