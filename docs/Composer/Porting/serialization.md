### [TOC](./TOC.md)
### [Back - Modeling language](./modeling.md)

# Serialization
Serialisation is an important part of both client to/from business network as well as business network to/from world state. Composer provided serialization capabilities to handle this all for you so you only ever worked with composer model objects, and it also took care of references (especially when those references had been resolved to an object). As part of your replacement work you will need to implement your own serializer for use between client/contract and contract/ledger. The serializer could just be as simple as `JSON.parse` to deserializer and `JSON.stringify` to serialize.

In general it would be reasonable to use the same serializer for both client/contract exchanges as well as contract/world state exchanges.

## DateTime
One problem that you could face is around the DateTime type in Composer. Composer understood the shape and types of the model at runtime but unless you implement this concept yourself you won't understand the types at runtime.

So imagine the situation where you just use JSON.stringify to serialize the date object. That resolves to a string. The question is how do you deserialize it and recreate the date object ? JSON.parse won't work because it will just create a property with a string value which isn't what you want, you will want a date object again. You will need to ensure that your serializer/deserializer handles this situation.





### [Next - Resource creation and data validation](./modelmanagement.md)