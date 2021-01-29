# Guidelines for writing Node Chaincode

## Node Vs Java Vs Go performance

Go will always be the most performant chaincode. In fabric 2.2 the node runtime was moved off of node 8 to node 12 so you now have a better runtime with more language support and also the node 1.8Gb memory limit has been removed which means node is not restricted memorywise is as it was before

Node however is single threaded. It will not make use of more than 1 CPU to run you chaincode (although it can benefit from more than 1 cpu because node itself will run background tasks on different threads)

Java is by far the worst performing chaincode to the point where at this time of writing I really cannot recommend using java chaincode.

## What are the differences between the Contract API and the Shim

Node was the first implementation of the contract api. Java and Go also have a contract api and may differ in how they work. I will focus on the node contract api.

The contract api exists to improve and assist in the interaction between client and chaincode. It also exists to provide what might be considered a more modern/intuitive way of writing chaincode. Note that this also is where the term smart contract was first introduced but unfortunately can introduce confusion as to what is the difference between a smart contract and chaincode. You can consider from a contract api point of view that the chaincode package will consist of 1 or more smart contracts (the more part will be mentioned later)

What it doesn't do is provide any layer between the transaction working with private data or the world state. There has been talk of the ledger api which might provide something there. I'm not convinced of the value of a ledger api expect to provide a different api that tries to do the same thing and may end up limiting you to what you can do compared to the simple put/get/delete/query apis that currently exist.

### Capabilities

The contract API provides the following capabilities

- dispatcher
  - removes the need for boiler plate code if coding to the shim
  - Reality is that's not a lot of code though, but does present a more object orientated way to create your chaincode
- context
  - all contract transaction methods get the same context
    - this is synonimous to a shim implmentation passing the stub to all chaincode transactions
  - context contains the shim stub for interacting with the states (private/world)
  - context contains a usable client identity class to determine for information about the invoker
  - you can extend this context for your own needs.
  - invokes unknownTransaction method if can't dipatch the request
- serializer (can it handle complex types in typescript annotations ?)
  - inbuilt JSON serializer/deserializer but opaque as to it's workings (no equivalent client side)
  - can be replaced with own
    - code doesn't work properly but can be made to work
    - completely undocumented
  - Is annotation aware for typescript annotations
  - Means you don't have to mess with Buffer objects for return values
- introspection and data validation support
  - can provide explicit metadata file to describe the contract conforming to JSON Schema
    - actually has a cli to help in the generation of explicit metadata
      - undocumented
  - can be retrieved by a client using the system contract method getMetadata
  - will be used to validate input parameters and return values if enough detail provided
  - also defines the actual exposed transactions for the dispatcher
  - metadata can be derived for javascript/typescript (no annotations) if no explicit metadata provided
    - can't perform data validation
    - includes all methods in a contract (can be controlled but isn't documented. HINT: It's the use of `_` in a method name)
    - typescript annotations can be used to provide more information
      - This is poorly documented
- introduced the ability to plug in different types of api to provide alternatives to the contract api
  - shim has a service provider interface
  - completely undocumented
- Can support multiple "contracts" in a single chaincode
  - used to allow getting metadata for a contract by defining an included system contract which get's automatically included in your chaincode
  - Still wondering what the value of this is, I've seen no good use case for this capability from a chaincode creator point of view
  - people incorrectly think this is a way to split a chaincode implementation into multiple files. You can do that already
    - people then can't work out how to call their 2nd contract code from the 1st contract code as the contracts don't have references to each other
- aspect like concepts for transactions
  - before/after transaction method can be implemented
  - helps remove a little more boiler plate code
  - also includes unknownTransaction (see dispatcher)

For validation the only real documentation that defines for format of the explicit metadata file is
[here](https://hyperledger.github.io/fabric-chaincode-node/release-2.2/api/contract-schema.json)

But this doesn't explain how these capabilities are defined using typescript annotations

### Summary

There is a lot of capability provided in the contract api but sadly it's still really difficult to take advantage of the capabilities mainly due to lack of decent documentation. It's also awfully over-engineered and provides capabilities that have come from overthinking what developers might requires and you may find it limiting when it comes to trying to implement more complex scenarios (as I have). It does however make getting started in writing chaincode much easier.

Both apis are unlikely to disappear anytime soon, the shim can be seen as the more powerful but more work required api whereas the contract api makes it easier and provides useful capabilities out of the box.

The push from fabric and IBM is to use the contract API, however the decision is up to you. The guidelines here are about transaction logic and so doesn't matter if you use the contract api or not.

One final point, in chaincode you are able to distinguish between a transaction that is run as part of instantiation/upgrade and a transaction that isn't. This can be useful if you need to have some sort of atomic action done when a chaincode is say first run to populate some security data for example where doing it as a followon transaction is considered a security risk. The contract API hides the distinction so if you do need this then you might have to work on a convoluted approach to checking to see if a transaction has ever been run before to determine. However this approach cannot be used in the upgrade path.

## servicability

- difficult to do
  - every contract will have to implement a transaction to allow for the ability to turn logging on/off. There is no help provided by fabric
  - this would also have to be used to control getting shim/contract api logs, documentation on how to achieve this in Java and Node (but not Go) can be found [here](https://stackoverflow.com/questions/65088231/in-a-hyperledger-fabric-smart-contract-how-do-i-turn-on-logging)

## input/output validation

- do you need it ? ie do you trust the caller implicitly for input ? either way I would recommend you do parameter checking, there can always be a bug in your trusted client code.
- contract API can provide this, but may not be comprehensive enough for your needs

## Private Data

(NEEDS WORK HERE)
- how to write to PDCs you can't read from
- how to handle where invoker is from a foreign org

## Queries

(NEEDS WORK HERE)

- $or etc
- limit and pagination option
- close iterators

## Chaincode events

- SetEvent can only store 1 piece of data. Multiple calls will overwrite the previous call
- Chaincode events are just a block of data. So long as the client understands the format of the data, it's easily possible to store multiple events
- Chaincode events are just a section in a transaction and a transaction is included in the block. It's the SDKs that receive the committed block and extact that section and emit them in whatever manner they deem appropriate for their language. The main point is that events can only be received once a block has been committed. The SDK may or may not check that the transaction in the block is valid or not. You should check your SDK documentation to confirm.

```javascript
const events = [];
...
events.push('event 1');
...
events.push('event 2');
...
stub.setEvent('myevent', Buffer.from(JSON.stringify(events)));
```

client can now receive the event and JSON.parse the payload to get multiple events

## Chaincode level Access Control

Chaincode can implement access control based on the transaction creator, here are some recommendations

- Avoid using ABAC (attribute based attribute control)
  - The management of certificates based on attributes will become a problem if you need to adjust those attributes
    - You would have to revoke all currently issued certificates and issue new ones for example
- Don't compare whole certificates in your chaincode
  - If you do then you have to make sure you update the store of certificates that the chaincode uses before they expire otherwise you run the risk of locking yourself out of your own chaincode if it requires a transaction (or chaincode upgrade) governed by doing a certificate check to update the certificate. A problem that was an achilles heel for Hyperledger Composer
  - Look to using a subset of the certificate such that it's possible to renew a certificate without having to update the certificate referenced by the chaincode

## State Based Endorsement

- consider private data collection endorsement policies if appropriate as these are established up front
  - State Based Endorsements on a key don't become active until that key is committed, so you cannot use SBE's on the creation of a key

## Chaincode no-nos

- non deterministic logic
  - don't generate random numbers in chaincode
  - don't create a new Date object, get the transaction datetime from the stub
- implementing any sort of cache on data sent from client
  - You can't cache information into memory for use by subsequent transactions such as caching information in the world state. Just don't do it.
  
## Serializer

- You will need to write a serializer for data stored on the ledger and retrieved on the ledger. This serializer MUST be deterministic, ie it must produce the same results from the same data on different machines
- Ideally the serializer you use here could be the same one you use to serialize/deserialize data between the chaincode and the client but it doesn't have to be. Again it should be deterministic to ensure proposals compared between different peers in the network match.
- JSON.parse/stringify in node.js is deterministic.
- Your serializer will need to handle special classes such as Date/Map. You might find the reviver and replacer options on the JSON methods useful

## unit testing

- @theledger/fabric-mock-stub (or your own to provide a stub simulation) for interaction with world state (and private state ?)
  - This is more an functional test level where you test the functionality of the chaincode as a whole rather than individual
- Use sinon to mock the stub directly and invoke each transaction passing that in
  - for contract api, you can mock a whole context for example
  
```javascript
class TestContext {
    constructor() {
        console.log(new Error().stack);
        this.stub = sinon.createStubInstance(ChaincodeStub);
        this.clientIdentity = sinon.createStubInstance(ClientIdentity);
        this.logger = {
            getLogger: sinon.stub().returns(sinon.createStubInstance(winston.createLogger().constructor)),
            setLevel: sinon.stub()
        };
    }
}
```

- You could provide a wrapper around all stub interactions and mock at that level of course
- always have functional tests and test in fabric itself, your unit tests or a simulated stub will not test against true fabric behaviour
