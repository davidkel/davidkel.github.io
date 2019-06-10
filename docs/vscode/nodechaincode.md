# Importing node chaincode into vscode
Support I have a project as follows which includes my client code, my contract code and a shared model directory (used by both the client and contract code) in a directory called `tsproject`

```
├── client-ts
├── contract
└── model
```
if you open vscode on the `tsproject` folder and try to package the contract you get the error
```
Failed to determine workspace language type, supported languages are JavaScript, TypeScript, Go and Java. Please ensure your contract's root-level directory is open in the Explorer.
```

There are 2 solutions to this
1. Open code in the actual contract folder (ie cd contract && code .) but this is not ideal as you lose the model folder and the client folder as well
2. You will need to add the contract folder to the workspace and then save the workspace so that vscode will remember this in the future
   - add folder to workspace
   - locate the contract directory and add

You should now have an Untitled workspace with 2 root folders
- tsproject
- contract

You should now save this workspace with an appropriate name into the tsproject folder using the `save workspace as` menu option in the File dropdown

If you need to start vscode again, opening the tsproject folder in vscode, it will ask you if you want to open the workspace.

When you package, you need to specify the `contract` directory.

you may also want to consider creating a `.vscode` directory in your project folder (in this example `tsproject`) with the file `extensions.json` and contents 
```
{
    "recommendations": [
        "ibmblockchain.ibm-blockchain-platform"
    ]
}
```
This would then get vscode to suggest you need to install the ibm blockchain platform extension if that folder was opened in vscode if it had not already been installed.