# Caliper Benchmark alternative via docker

* mkdir docker-benchmarks
* cd docker-benchmarks
* git clone https://github.com/hyperledger/caliper-benchmarks
* curl -sSL https://raw.githubusercontent.com/hyperledger/fabric/main/scripts/bootstrap.sh | bash -s

```bash
docker run --rm --network host -e CALIPER_BIND_SUT=fabric:2.2 -e CALIPER_WORKSPACE=/state/caliper-benchmarks -e CALIPER_BENCHCONFIG=/state/caliper-benchmarks/benchmarks/api/fabric/test.yaml -e CALIPER_NETWORKCONFIG=/state/caliper-benchmarks/networks/fabric/test-network.yaml -v $PWD:/state hyperledger/caliper:0.5.0-unstable-20220512172343 launch manager
```

```
docker run --rm --network host -e CALIPER_BIND_SUT=fabric:1.4 -e CALIPER_WORKSPACE=/state/caliper-benchmarks -e CALIPER_BENCHCONFIG=/state/caliper-benchmarks/benchmarks/api/fabric/test.yaml -e CALIPER_NETWORKCONFIG=/state/caliper-benchmarks/networks/fabric/test-network.yaml -v $PWD:/state hyperledger/caliper:0.5.0-unstable-20220512172343 launch manager --caliper-fabric-gateway-enabled
```

For the above, prometheus and docker monitors won't work (why ?)

```
Error: connect ECONNREFUSED 127.0.0.1:9090
2022.05.13-10:46:50.599 error [caliper] [prometheus-query-client]       Query error:  ({"errno":-111,"code":"ECONNREFUSED","syscall":"connect","address":"127.0.0.1","port":9090})
```

```
Error retrieving containers: Error: connect ENOENT /var/run/docker.sock
```

can't run it inside the fabric_test network because the connection profile specifies the url as localhost


```
docker run -it --entrypoint /bin/sh --rm --network host -e CALIPER_BIND_SUT=fabric:2.4 -e CALIPER_FLOW_ONLY_TEST=true -e CALIPER_WORKSPACE=/state/caliper-benchmarks -e CALIPER_BENCHCONFIG=/state/caliper-benchmarks/benchmarks/api/fabric/test.yaml -e CALIPER_NETWORKCONFIG=/state/caliper-benchmarks/networks/fabric/test-network.yaml -v $PWD:/state c:fix
```