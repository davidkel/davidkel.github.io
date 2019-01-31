### [TOC](./TOC.md)
### [Back - Business network cards and card stores](./cards.md)

# CLI

Composer provided a CLI for operational aspects as well as some simple testing. You will need to look at the fabric and fabric-ca CLI capabilities now. For example examples of using the peer command to install chaincode are 
```
CORE_PEER_ADDRESS=localhost:8051 CORE_PEER_LOCALMSPID=Org1MSP CORE_PEER_MSPCONFIGPATH=crypto-config/peerOrganizations/org1.example.com/users/Admin@org1.example.com/msp peer chaincode install -l node -n demo -p ./mynodechaincode -v 0.0.1
```
if CORE_PEER_ADDRESS is not specified it will default to `localhost:7051`

and to instantiate chaincode
```
CORE_PEER_LOCALMSPID=Org1MSP CORE_PEER_MSPCONFIGPATH=crypto-config/peerOrganizations/org1.example.com/users/Admin@org1.example.com/msp peer chaincode instantiate -o localhost:7050 -C composerchannel -l node -n demo -v 0.0.1 -c '{"Args":["init", "key1", "1", "key2", "2"]}'

```
TODO: reference the docs for peer and fabric-ca-clients CLI


### [Next - Tooling](./tooling.md)