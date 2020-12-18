# Guidelines for writing Node Chaincode

## Should I use the contract API ?


## Private Data
* how to write to PDCs you can't read from
* how to handle where invoker is from a foreign org


## Queries
* $or etc
* limit and pagination option
* close iterators

## Chaincode events

## Creator
* Don't use ABAC
* Don't compare whole certificates

## State Based Endorsement
* consider private data collection endorsement policies if appropriate
 
## Chaincode no-nos
* non deterministic
* implementing any sort of cache on data sent from client
  
## Serializer

## unit testing
- @theledger/fabric-mock-stub (or your own simulator)
- mocking the stub apis directly ?
- abstract the code and mock at a higher level
- always have functional tests and test in fabric itself
- how to unit test with the contract api ?