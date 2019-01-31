### [TOC](./TOC.md)
### [Back - Data Storage and Registries](./datastorage.md)

# Query Language
Composer querys are converted to mango queries. These composer queries could either be `pre-converted` or converted on the fly. Pre-converted where the ones that were stored in the .qry file, whereas use of the buildQuery api call would convert the given query to a mango query. 

You will need to recreate your queries using the apache couchdb mango language then use the `getQueryResult` or `getQueryResultWithPagination` within your chaincode/smart contract. There is no inbuilt capability in the node-sdk to run mango queries directly in the same way you could just use the `buildQuery` and `query` apis in your client code. You would need to write a chaincode/smart contract function which invokes the getQueryResult call for the client.

In order to improve performance and allow for sorting the results, couchdb requires the use of `indexes` for information it stores. Composer would automatically generate indexes based on the queries defined in the `.qry` file for you. You will need to create these indexes yourself now. If you continue to use the `$class` field in your data (and it is highly recommended that you do) then you should include this field in your index. 
An example of a mango query and index file for trade `trade-network` business network for the query selectCommodotiesbyOwner

### Composer Query
```
query selectCommoditiesByOwner {
  description: "Select all commodities based on their owner"
  statement:
      SELECT org.example.trading.Commodity
          WHERE (owner == _$owner)
}
```

### Mango query
```
const lookupOwner = 'fred';
const query = `{"selector":{"\\\\$class":"${CommodityClass}","owner":"${lookupOwner}"}}`;
const iterator = await ctx.stub.getQueryResult(query);
```

And an example of getting all the results is

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


### couchdb index file
```
{
    "index": {
        "fields": [
            "\\$class",
            "owner"
        ]
    },
    "name":"selectCommoditiesByOwner",
    "ddoc":"selectCommoditiesByOwner",
    "type":"json"
}
```
Note the use of the backslashes as this is important for these fields to work.

### [Next - ACLs](./acls.md)