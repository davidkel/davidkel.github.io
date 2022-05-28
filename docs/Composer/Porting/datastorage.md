### [TOC](./TOC.md)
### [Back - Modeling Language](./modeling.md)

# Composer Data Storage
This section covers the data structure and data keys as well as the meta-data composer uses when storing information around the definitions both in the applications model definitions and also the system model.

The section will help you keep working with existing information, it will show the metadata that is included with the data that is stored with a recommendation on which ones to keep. Plus it will explain the composite keys used so you can access your data. It will also include the metadata composite keys that are created which you can either make use of, delete or leave alone.

## Your Asset and Participant data
Composer stores your asset and participant information in the world state and it creates composite keys as appropriate. A Composite key in fabric is made up of a base key with 1 or more key extensions (for more information please see the documentation on `createCompositeKey`). The composer base key as made up as follows
```
[Type]:[class]

where type can be `Asset`, `Participant` or `Transaction`
and class is the fully qualified identifier of the declaration.
```
Composer then uses a single key extension based on the value of the field which identifies the specific Asset or Participant

So lets consider the following model definition

```
namespace org.example.trading

asset Commodity identified by tradingSymbol {
  o String tradingSymbol
  o String description
  o String mainExchange
  o Double quantity
  --> Trader owner
}

participant Trader identified by tradeId {
  o String tradeId
  o String firstName
  o String lastName
}

transaction Trade {
  --> Commodity commodity
  --> Trader newOwner
}
```

The Commodity Asset will have a base key of `Asset:org.example.trading.Commodity` and the extension key will be the value of the `tradingSymbol` field.

The Trader Participant will have a base key of `Participant:org.example.trading.Trader` and the extension key will be the value of the `tradeId` field.

## Metadata added to the data stored in the world state.
On top of the data that is defined by your model, composer will augment 3 fields to your data before storing it in the world state. Lets consider an example here. Suppose we have a Commodity defined as the following
```
{
    tradingSymbol: 'AcmeShares',
    desciption: 'Shares in the Acme corporation',
    mainExchange: 'NASDAQ',
    quantity: 1234,
    owner: 'resource:org.example.trading.Trader#T1'
}
```
Composer will store the following JSON string into a composite key of `Asset:org.example.trading.Commodity` (base) + `AcmeShares` (extension) 
```
{
    "$class": "org.example.trading.Commodity",
    "$registryId": "org.example.trading.Commodity",
    "$registryType": "Asset",
    "tradingSymbol": "AcmeShares",
    "desciption": "Shares in the Acme corporation",
    "mainExchange": "NASDAQ",
    "quantity": 1234,
    "owner": "resource:org.example.trading.Trader#T1"
}
```

Here is an example of reading that data using pure fabric and the new programming model (ie using the context object)

```
const stub = ctx.stub;
const compositeKey = stub.createCompositeKey('Asset:org.example.trading.Commodity' , ['AcmeShares']);
const assetBuffer = await stub.getState(compositeKey);
const assetStr = assetBuffer.toString('utf8'); // convert the buffer to a string
const asset = JSON.parse(assetStr);  // convert the JSON string into a javascript object
```

(if you don't use the new programming model, then you are passed the stub directly in your chaincode function).

Here is an example of creating a new entry for some new Commodity passed which is in javascript object format

```
const stub = ctx.stub;
const compositeKey = stub.createCompositeKey('Asset:org.example.trading.Commodity' , [newAsset.tradingSymbol]);
newAsset.$class = 'org.example.trading.Commodity'
await stub.putState(compositeKey, Buffer.from(JSON.stringify(newAsset)));
```
Note in the above example the `$class` field is replicated, but the others are not. It's recommended keeping at least the `$class` field if you plan to use rich queries as this then becomes a useful field to index on. You may also want to consider keeping the `$registryType` as well as it may be helpful to you to know what type of data is stored.

## Other keys that composer created
Outside of the keys that are used to store the actual business network data, composer maintained other keys to help facilitate it's runtime, these keys are not something you need to make use of, unless you feel the need to replicate yourself some of the composer capabilities. The Composer meta-data keys will all start with a `$`. Here is a quick summary of those meta-data keys (the key split is denoted by a `~`)
| key | description |
| $syscollections~$sysregistries | defines the existence of the $sysregistries collection |
| $syscollections~$sysdata | defines the existence of the $sysdata collection |
| $sysdata~metadata | composer runtime metadata |
| $sysregistries~(Asset/Participant/Transaction):NAMESPACE+NAME | defines the existence of the individual registries |
| $syscollections~(Asset/Participant/Transaction):NAMESPACE+NAME | defines the existence of the individual registries collection |

Note that Composer also uses registries to store the identity mapping information and the historian records so there will also be keys for these data entries as well. See [Identity Management](./identity.md) and [Historian](./historian.md) for more details.


### [Next - Query Language](./querylang.md)