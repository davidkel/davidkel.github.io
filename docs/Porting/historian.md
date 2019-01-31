### [TOC](./TOC.md)
### [Back - ACLs](./acls.md)

# Historian

There is no equivalent to historian in fabric. Historian was used to record the transaction object that were submitted (which resulted in 1 or more TP functions being invoked) and any events emitted. You would have to implement your own version of recording function invocation and event emission yourself. Access was either via rich queries to query the historian or via the `getHistorian` client SDK api.

The most common use case for people is to get the history of an asset. Using historian for this was not ideal. In fact fabric itself provides a chaincode API `getHistoryForKey` to allow you to get the history of a key. As a key will map 1-1 to an asset or participant see [Data Storage](./datastorage.md) you can use this to get the history of an asset or participant.

Historian records are stored with a base key of `Asset:org.hyperledger.composer.system.HistorianRecord` and the extension key will be the value of the `transactionId` field.

```
asset HistorianRecord identified by transactionId {
  o String        transactionId
  o String        transactionType
  --> Transaction transactionInvoked
  --> Participant participantInvoking  optional
  --> Identity    identityUsed         optional
  o Event[]       eventsEmitted        optional
  o DateTime      transactionTimestamp
}
```

On the client side you could get access to this registry by using the `getHistorian` on a business network connection that then provided a `registry` object to be able to work with the HistorianRecord Asset.

### [Next - Events](./events.md)