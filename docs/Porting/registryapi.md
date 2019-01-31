### [TOC](./TOC.md)
### [Back - Request and POST TP Apis](./requestandpost.md)

# Registry API
Composer provided registries which provided a grouping of resources. You would first request the registry you wanted to work with using the appropriate api to request a registry (eg getAssetRegistry, getParticipantRegistry). This concept isn't available in native fabric. They keys used by composer achieve a similar logical grouping and are covered in the next section where appropriate

## Registry API
In Composer once you had a registry you would then access your resource data. The registry api provides the following capabilities for both client and business network TP function. The equivalent for fabric is given but note there is no equivalent client side in the fabric-network api. To achieve it your client code will have to invoke the chaincode/smart contract which provides an equivalent capability and use either submit/submitTransaction or evaluate/evaluateTransaction depending on whether the world state is to be updated or not. See [Client SDK](./client.md) for more information. Also you need to use the appropriate composite key as described in the [Composer Data Storage](./datastorage.md) section

### basic CRUD type operations
To provide the basic CRUD type operations on assets and participants is quite easy. It's up to you whether you want to add checking for existence and throwing an error first before performing some of the operations. The following table provides the registry call and the equivalent fabric api call to use.

| api | equivalent fabric api | client request |
| --- | --------------------- | -------------- |
| get | getState | evaluate |
| add | putState | submit |
| update | putState | submit |
| remove | deleteState | submit |
| exists | const buffer = await getState();return buffer.length !=== 0 | evaluate |

### xxxAll type operations
Composer also provided some `xxxAll` type functions. `addAll`, `removeAll`, `updateAll` simply looped around the array of resources and performed the equivalent single invocation, for example on addAll something similar to this was used.
```
async addAll(resources)
for (const resource of resources) {
    await this.add(resource);
}
```

`getAll` did something different, it uses a feature of fabric to get a collection of values based on a partial composite key. The partial composite key will be a key `without` the identifier of a single asset/participant. So for example if I wanted to get all the Commodity assets in my sample trade-network I would do
```
const iterator = ctx.stub.getStateByPartialCompositeKey('Asset:org.example.trading.Commodity', []);
```
This will return an iterator for you to then iterate through some or all of the results as you see fit.

Here is an example of getting all the results from a returned iterator

```
static async getAllResults(iterator) {
    let results = [];
    let res = {done: false};
    while (!res.done) {
        res = await iterator.next();
        if (res && res.value && res.value.value) {
            let val = res.value.value.toString('utf8');
            if (val.length > 0) {
                results.push(JSON.parse(val));
            }
        }
        if (res && res.done) {
            try {
                await iterator.close();
            }
            catch(err) {
                // log a warning
            }
            return results;
        }
    }
}
```
Note that if you get back a large result set this could have performance and resource implications.

## Resolve and ResolveAll
Composer models defined relationships to other resources. These would usually be defined as a string which referenced the specific asset/participant. This took the form
```
resource:[classname]#[identifier]
```
For example in the trade-network example, here is a commodity with a reference to the `T1` trader in the owner field.
```
{
    tradingSymbol: 'AcmeShares',
    desciption: 'Shares in the Acme corporation',
    mainExchange: 'NASDAQ',
    quantity: 1234,
    owner: 'resource:org.example.trading.Trader#T1'
}
```

The `resolve` api would traverse the resource converting any relationship (including any relationships in a relationship that itself was resolved) to their true value. So for example if Trader T1 looked like
```
{
    tradeId: 'T1',
    firstName: 'Alex',
    lastName: 'Bloggs'
}
```

The resolving the commodity would result in an object with the structure
```
{
    tradingSymbol: 'AcmeShares',
    desciption: 'Shares in the Acme corporation',
    mainExchange: 'NASDAQ',
    quantity: 1234,
    owner: {
        tradeId: 'T1',
        firstName: 'Alex',
        lastName: 'Bloggs'
    }
}
```

This is purely a composer capability linked to it's modeling capabilities and so there is nothing in fabric to assist. You would need to have these resource references yourself as required.

`resolveAll` would just get all (using getAll) the resources in the specified registry and resolve each one.

### [Next - Invoking transactions](./client.md)