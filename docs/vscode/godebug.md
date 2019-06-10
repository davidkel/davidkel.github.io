# Debugging Go Chaincode

You can debug go chaincode using the IBM Blockchain extension. Make sure you have installed the following
1. golang 1.12 or higher and the bin directory is in your PATH.
2. Install the official go vscode extension and also install all the required tools it asks for (you will need at least dlv installed for debugging to work)
3. Ensure you have got the dependent shim code
```
go get -u github.com/hyperledger/fabric/core/chaincode/shim
```
4. use govendor, dep or modules for your other dependencies.

If you are importing an existing go project (REFERENCE gochaincode.md) and you haven't done so already you will need to create the `.vscode` directory in the chaincode source directory. Then create or update the `launch.json` file in this directory. If you have used the IBM blockchain extension to create a project from scratch then you don't have to do this step.

- create a `.vscode` directory inside your chaincode source directory (in my example it would be the mycc directory) (POINT TO PREVIOUS or GIVE DIR AGAIN)
- create the file `launch.json` in this directory with the following content
```
{
    "version": "0.2.0",
    "configurations": [
        {
            "type": "fabric:go",
            "request": "launch",
            "name": "Debug Smart Contract"
        }
    ]
}
```
If the file already exists then you just need to add the extra configuration to the configurations array, ie 
```
        {
            "type": "fabric:go",
            "request": "launch",
            "name": "Debug Smart Contract"
        }
```

## ensuring the local fabric is setup in development mode
- if local_fabric under fabric gateways as an infinity symbol next to it then the fabric is setup to run in development mode, if not then
- first start the local fabric
- expand the nodes tree in local fabric ops
- right click peer0.org1.example.com and select toggle development mode
- The local fabric will then restart

## debugging your Go chaincode
- Switch to the debug view
- From the dropdown select `Debug Smart Contract`
- Click the Play button to the left of the dropdown to start a debug session
- enter a name eg `dd` for the name of the chaincode package
- Now we should be ready to debug and that includes debugging instantiation as the chaincode has not been instantiated yet so you will need to do that. You can either
  - Switch back to the blockchain extension to instantiate using the `+ Instantiate` in the local fabric Ops window 
  - Press the blockchain button (far right) in the debug icon panel and instantiate there (meaning you can stay in the debug view of vscode)
- Note that chaincode output is in the debug console.

## modify code and resume
if you change the code, it isn't automatically picked up (ie no sort of hot fixing). You can however change the code, and press the restart button. The code will be recompiled and restarted

## Danger
- Timeout is set to infinite
- vscode UI may make calls to the chaincode and if you debug logic that it invokes it can hang the UI.





