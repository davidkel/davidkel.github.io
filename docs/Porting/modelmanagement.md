### [TOC](./TOC.md)
### [Back - Modeling language](./modeling.md)

# Creating modelled resources and validation
Composer provided the capability to build instances of the defined model elements, perform serialization and validation of those instances. Both the client and the chaincode/smart contract made use of the `Factory` and `Serializer` for this and the validation was inbuilt into both the client and runtime side. 

Composer's serialization of data was for over the wire transportation. So in the submit transaction for example you passed in an object and the transaction was passed a constituted object to work with. returned objects from the transaction were reconstituted back at the client side for the application to work with. Only if you use the metadata definition of a contract (either by defining the metadata.json or using typescript annotations) do you get this happening at the chaincode side (ie you get an object as a parameter to your function and can return objects) otherwise you still need to work with strings.


### [Next - Data storage](./datastorage.md)