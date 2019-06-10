# How to debug javascript/typescript using ibm blockchain extension

If you are importing an existing node project (REFERENCE nodechaincode.md) and you haven't done so already you will need to create the `.vscode` directory in the chaincode source directory. Then create or update the `launch.json` file in this directory. If you have used the IBM blockchain extension to create a project from scratch then you don't have to do this step.

- create a `.vscode` directory inside your chaincode source directory (in my example it would be the mycc directory) (POINT TO PREVIOUS or GIVE DIR AGAIN)
- create the file `launch.json` in this directory with the following content

### For typescript
```
{
    // Use IntelliSense to learn about possible attributes.
    // Hover to view descriptions of existing attributes.
    // For more information, visit: https://go.microsoft.com/fwlink/?linkid=830387
    "version": "0.2.0",
    "configurations": [
        {
            "type": "fabric:node",
            "request": "launch",
            "name": "Debug Smart Contract",
            "preLaunchTask": "tsc: build - tsconfig.json",
            "outFiles": [
                "${workspaceFolder}/dist/**/*.js"
            ]
        }
    ]
}
```
Note that this assumes your typescript compilation options place the compiled code into the `dist` directory


If the file already exists then you just need to add the extra configuration to the configurations array, ie 
```
        {
            "type": "fabric:node",
            "request": "launch",
            "name": "Debug Smart Contract",
            "preLaunchTask": "tsc: build - tsconfig.json",
            "outFiles": [
                "${workspaceFolder}/dist/**/*.js"
            ]
        }
```

### For javascript
```
{
    // Use IntelliSense to learn about possible attributes.
    // Hover to view descriptions of existing attributes.
    // For more information, visit: https://go.microsoft.com/fwlink/?linkid=830387
    "version": "0.2.0",
    "configurations": [
        {
            "type": "fabric:node",
            "request": "launch",
            "name": "Debug Smart Contract",
        }
    ]
}
```
Note that this assumes your typescript compilation options place the compiled code into the `dist` directory


If the file already exists then you just need to add the extra configuration to the configurations array, ie 
```
        {
            "type": "fabric:node",
            "request": "launch",
            "name": "Debug Smart Contract",
        }
```


## ensuring the local fabric is setup in development mode
- if local_fabric under fabric gateways as an infinity symbol next to it then the fabric is setup to run in development mode, if not then
- first start the local fabric
- expand the nodes tree in local fabric ops
- right click peer0.org1.example.com and select toggle development mode
- The local fabric will then restart

## debugging your Node chaincode
Note that for typescript you can set breakpoints in the typescript source code rather than in the javascript generated from the typescript.
- Switch to the debug view
- From the dropdown select `Debug Smart Contract`
- Click the Play button to the left of the dropdown to start a debug session
- Now we should be ready to debug and that includes debugging instantiation as the chaincode has not been instantiated yet so you will need to do that. You can either
  - Switch back to the blockchain extension to instantiate using the `+ Instantiate` in the local fabric Ops window 
  - Press the blockchain button (far right) in the debug icon panel and instantiate there (meaning you can stay in the debug view of vscode)
- Note that chaincode output is in the debug console.
  
## modify code and resume
if you change the code, it isn't automatically picked up, but it's easy to get it loaded, even for typescript, just press the restart button and it will recompile the typescript if it is typescript and restart the chaincode attaching it to the peer.

## Danger
- Timeout is set to infinite
- vscode UI may make calls to the chaincode and if you debug logic that it invokes it can hang the UI.
