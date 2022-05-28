cd ~/mqtt && docker run -d --rm --name mqtt -p 1883:1883 -p 9001:9001 -v $PWD:/mosquitto/config eclipse-mosquitto:latest


directory mqtt contains mosquitto.conf (no security)
persistence false
listener 1883
allow_anonymous true


npx caliper launch manager --caliper-workspace ./ --caliper-networkconfig networks/aws-network.yaml --caliper-benchconfig <BENCHMARK FILE eg benchmarks/api/fabric/empty-contract-test.yaml> --caliper-flow-only-test  --caliper-worker-remote true --caliper-worker-communication-method mqtt --caliper-worker-communication-address mqtt://localhost:1883

npx caliper launch worker --caliper-worker-remote true --caliper-worker-communication-method mqtt --caliper-worker-communication-address mqtt://<IP_ADDR_OF_CALIPER_MANAGER>:1883 --caliper-workspace ./ --caliper-networkconfig networks/aws-network.yaml --caliper-benchconfig <BENCHMARK FILE eg benchmarks/api/fabric/empty-contract-test.yaml> --caliper-fabric-gateway-enabled --caliper-fabric-gateway-localhost false


for worker benchmark file is irrelevant, file just has to exist, however the manager will tell the worker the path to the workload file so this be setup correctly on the worker

for manager network config file is irrelevant but must exist

easiest way is to use caliper-benchmarks and make that the caliper workspace