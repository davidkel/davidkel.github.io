# Node Chaincode
This section is provides general information about developing node.js chaincode. It covers my experience of using the so called `low-level` api which actually still has to be used by chaincode using the `high-level` contract api as well as using the `contract` api which removes the need for some small boilerplate code and adds extra capabilities such as 
1. contract meta-data (describes the contract, converts input bytes to javascript objects for methods based on the metadata, supports custom serializers)
2. The ability to accomodate multiple contracts in the same chaincode implementation (they will be able to see each other's data)

(TODO: Need a table section)


## How to develop node chaincode
There are no hyperledger fabric tools to help with development of node chaincode (there are free commercial ones however such as the IBM Blockchain platform extension for VSCode), so this section provides guidance on using development mode of a peer to help in developing your chaincode.

- Ensure you start your fabric peer in development mode.
- npm install your chaincode package
- typescript compile it (if required)
- Start the chaincode and get it to register the the peer
  - If using the contract api
```
npm start -- --peer.address grpc://localhost:7052 --chaincode-id-name trade-network:0.0.1
```
Ensure your package.json has a start entry, eg
```
    "scripts": {
        "start-debug": "node --inspect-brk ./node_modules/fabric-shim/cli.js start",
        "start": "fabric-chaincode-node start",
        "build": "tsc",
        "build:watch": "tsc -w",
    },
```
  - If not using the contract package
```
CORE_CHAINCODE_ID_NAME="trade-network:0.0.1" node chaincode/simpleChaincode.js --peer.address grpc://localhost:7052
```
- install some dummy chaincode package with name and version with appropriate name and version
- should now be able to instantiate that name and version of the chaincode
- you can now drive this chaincode from your client application or even from the vscode extension if you import the connection profile. 
- To update the chaincode, make changes to your source code, go to the window where the chaincode is running and stop it (CTRL-C) then restart it using the above command again.

### how to debug this node chaincode
You can also debug your chaincode using VSCode as follows

```
npm run start-debug -- --peer.address grpc://localhost:7052 --chaincode-id-name trade-network:0.0.1
```
add a `attach to node.js` configuration
Then attach vscode to the debug session
add breakpoint to ts file




