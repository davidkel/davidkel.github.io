### [TOC](./TOC.md)
### [Back - Identity Management](./identity.md)

# BNA

This was composer's business network packaging format. hyperledger fabric does have it's own packaging format for chaincode. Look at the `peer chaincode package`.

see [peer chaincode](https://hyperledger-fabric.readthedocs.io/en/release-1.4/commands/peerchaincode.html) for more information about the CLI package chaincode

```
peer chaincode package mycc.cds -l node -n mycc -v 0.0.1 -p ./mycc
```

Then to install you can do
```
peer chaincode install mycc.cds
```
### [Next - Business Network Cards and Card Stores](./cards.md)