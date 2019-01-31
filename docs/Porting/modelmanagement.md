### [TOC](./TOC.md)
### [Back - Modeling language](./modeling.md)

# Creating modelled resources and validation
Composer provided the capability to build instances of the defined model elements, perform serialization and validation of those instances. Both the client and the chaincode/smart contract made use of the `Factory` and `Serializer` for this and the validation was inbuilt into both the client and runtime side. 

Hyperledger fabric doesn't provide this capability (see discussion about Hyperledger Concerto in [modeling](./modeling.md) as an option) and would require you to provide your own solution as required, for example the new `contract-api` in the node.js fabric-shim provides the facility for custom serializers.

### [Next - Data storage](./datastorage.md)