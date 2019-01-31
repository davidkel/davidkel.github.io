### [TOC](./TOC.md)
### [Back - Introduction](./introduction.md)

# Modeling language
Composer provided a modeling language to help you define the structure of your data. It allowed for the modeling of the following types of data

- Assets
- Participants
- Transactions
- Concepts
- Events
- Enums

And these resources can be conceptually grouped together into a namespace. You could specify field types, optional fields, defaults for fields, regular expressions to provide field validation on top of ensuring it was the right type. You could also specify relationships to other types, arrays of types.

You could annotate your models with generic decorators which you could introspect at runtime. 

A specific set of annotations which made meaning to composer were as follows
```
@returns(...)
```
```
@commit(false)
```
Which will be covered more when looking at client side invocation of chaincode functions.

Finally you could split your models across multiple files, npm modules and import these definitions into your model file to make use of them. 

Composer would also provide model validation to ensure that the model was never violated. 

All in all the modeling capabilities in Hyperledger Composer were very comprehensive and Composer would manage the validation and storage on your behalf. 

The new node.js programming models in hyperledger fabric 1.4 do not provide any modeling capabilities, It can provide the ability to validate input values match a particular defined schema (as defined by the contract metadata you provide or via typescript annotations). It is up to the developer to provide whatever data management and validation capabilities are required themselves. 

You may have seen Hyperledger Concerto which takes the Composer modeling capabilities into a standalone component. However you should consider the following if contemplating using this.
1. It's not got an active community or backed by a company
2. It may not be fully up to date with bug fixes that have gone into composer since it was split out
3. One of the concerns reported back to composer was the performance of the modeling runtime managing the data and performing validation which may or not be a concern depending on whether TPS is important to you or not.

It is recommended you look at Typescript as opposed to javascript for developing both your client and chaincode applications. Typescript allows you to model your data and provide compile time validation of your code. If you do also want runtime validation you can look to see if there are any options to provide runtime validation for typescript or you could build your own validation in.

### [Next - Resource creation and data validation](./datastorage.md)
