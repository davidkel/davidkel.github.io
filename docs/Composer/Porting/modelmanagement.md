### [TOC](./TOC.md)
### [Back- Serialization](./serialization.md)

# Creating modelled resources and validation
Composer provided the capability to build instances of the defined model elements, perform serialization and validation of those instances. Both the client and the chaincode/smart contract made use of the `Factory` and `Serializer` for this and the validation was inbuilt into both the client and runtime side. 

Composer's serialization of data was for over the wire transportation. So in the submit transaction for example you passed in an object and the transaction was passed a constituted object to work with. returned objects from the transaction were reconstituted back at the client side for the application to work with. Only if you use the metadata definition of a contract (either by defining the metadata.json or using typescript annotations) do you get this happening at the chaincode side (ie you get an object as a parameter to your function and can return objects) otherwise you still need to work with strings.

## Factory
The factory for both client side and business network side was used to create an instance of an object which matched the model. If you are using typescript to define your model then you would use the standard mechanism for creating an object. For example

```
export class MyModel {
    id: string;
    firstName: string;
    ...
}


const myModelInstance = new MyModel();
```

## Serializer
Composer provided a serializer for various tasks such as converting between a composer object to a plain JSON object. You should be looking to using your own serializer see [serialization](./serialization.md) for more general info about serialisation, but given you won't be working with Composer objects any more calls such as `toJSON` and `fromJSON` are not required.


### [Next - Data storage](./datastorage.md)