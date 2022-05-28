1. If unfinished transactions grows then the network is overworked
2. Too much pressure can result in a worse result (fixed-tps) then reducing it to within a boundary

Too high a send rate on fixed-tps can result in worse figures

+------------------+-------+------+-----------------+-----------------+-----------------+-----------------+------------------+
| Name             | Succ  | Fail | Send Rate (TPS) | Max Latency (s) | Min Latency (s) | Avg Latency (s) | Throughput (TPS) |
|------------------|-------|------|-----------------|-----------------|-----------------|-----------------|------------------|
| create-asset-100 | 48024 | 0    | 399.3           | 19.14           | 1.08            | 10.82           | 352.7            |
+------------------+-------+------+-----------------+-----------------+-----------------+-----------------+------------------+
1:29
+------------------+-------+------+-----------------+-----------------+-----------------+-----------------+------------------+
| Name             | Succ  | Fail | Send Rate (TPS) | Max Latency (s) | Min Latency (s) | Avg Latency (s) | Throughput (TPS) |
|------------------|-------|------|-----------------|-----------------|-----------------|-----------------|------------------|
| create-asset-100 | 45024 | 0    | 374.3           | 6.45            | 0.98            | 3.23            | 367.2            |
+------------------+-------+------+-----------------+-----------------+-----------------+-----------------+------------------+


Also note that in this case fixed-load is not great at finding the max TPS it may have the same problem where it overloads
the network so results in higher latency, lower TPS and maybe more difficult to choose values


3. fixed load can give worse results than fixed-tps but then it might be due to averaging depending on how long it takes to get to a stable tps, the longer the run the more that factor is less of an influence


# Script to attempt to run a test without having to ssh into workers

echo "Launching Workers on Caliper Manager"
ssh root@9.46.97.23 'cd caliper-benchmarks;pwd;bash -s' < lw4.sh &
sleep 10
echo "Launching Caliper Worker 1"
ssh root@9.46.88.143 'cd caliper-benchmarks;pwd;bash -s' < lw.sh &
sleep 10
echo "Launching Caliper Worker 2"
ssh root@9.46.87.246 'cd caliper-benchmarks;pwd;bash -s' < lw.sh &
sleep 10
echo "Launching Manager"
ssh root@9.46.97.23 'cd caliper-benchmarks;pwd;bash -s' < lm.sh



# rate controllers

## fixed-tps
tps

## fixed-load
startingTps
transactionLoad

## fixed-feedback-rate
tps
transactionLoad


## Maximum rate
tps
step
sampleInterval
includeFailed


## Linear Rate
startingTps
finishingTps

## others
composite rate
record rate
replay rate


## stuff
1. cpu reduction
2. memory increase/decrease
3. improve i/o
4. TPS increase


Finding the maximum number of workers per machine if you want them to run at full capacity

Number of workers for number of clients required to perform the test but check
1. CPU used per worker
2. Overall CPU utilisation as the load generator itself may not be able to achieve full throughput
3. for fabric, the fat sdks will use some cpu, the new connector will use much less as the thin sdks do less work
4.
