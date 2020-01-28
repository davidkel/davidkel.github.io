### [TOC](./TOC.md)
### [Back - Composer registry](./registryapi.md)

# Invoking Transactions
The Client SDK consists of a client and admin api. The admin api mainly dealt with operational aspects such as business network install/start/upgrade and card administration. The fabric node-sdk package fabric-client provides the capability to form hyperledger fabric operational aspects such as install and instantiate (which was what a network start did as well as some other aspects). 

The client package was meant for general client side applications to interact with the chaincode/smart contract. Most of which has been covered by other sections. The final api, which is probably the most important one was `submitTransaction`. 

## submitTransaction
The normal situation in Composer was that a transaction was something that resulted in a change to the world state and a block containing this change would be added to the blockchain. Composer used a submit/notify model so that the call returned a promise which was fulfilled when the peers had committed the transaction to the blockchain. the `fabric-network` api provides an equivalent capability and you would either use `submitTransaction` on the `contract` instance or `submit` on a `Transaction` instance.
In Composer you could also optionally declare if the transaction should return modelled data using the `@returns()` annotation if successfully committed to the blockchain. When you submit a transaction using the fabric network you can also get back returned data.

### submitTransaction where transaction is read only
A feature of Composer was to define a transaction as a read only transaction, ie it should not commit it to the blockchain and optionally it could return a value as well (again using the `@returns()` annotation). In this case you would annotate the transaction with the `@commit(false)`. To do the equivalent using the `fabric-network` api you should call `evaluateTransaction` on the `contract` instance or `evaluate` on a `transaction` instance and the result will be returned.

### [Next - Misc other considerations](./misc.md)