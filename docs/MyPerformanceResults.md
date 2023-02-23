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