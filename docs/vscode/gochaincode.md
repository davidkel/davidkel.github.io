# Importing go chaincode into vscode

Suppose you have some Go Chaincode you have been developing and now want to import it into vscode for use by the extension. You need to import it at the correct level to ensure that the extension can use it.

For this example I will have some chaincode in the following directory structure
```
└── testcc
    ├── bin
    ├── pkg
    │   └── linux_amd64
    └── src
        ├── github.com
        ├── golang.org
        └── mycc
```
A pretty standard go workspace which is located at `~/mycode` so my GOPATH will be `~/mycode/testcc`

To ensure my GOPATH is set correctly and that vscode will pick it up, I do the following to launch vscode and pick up the correct folder
```
$ GOPATH=~/mycode/testcc code ~/mycode/testcc/src/mycc
```
It's important that your Go chaincode folder is at the top level seen by vscode. If it's not then you will encounter cryptic errors such as 
```
The Go smart contract is not a subdirectory of the path specified by the environment variable GOPATH. Please correct the environment variable GOPATH.
```
So for example if I had opened up code pointing to a higher directory for example
```
$ GOPATH=~/mycode/testcc code ~/mycode/testcc
```

then you will encounter this error.

Let's first check that vscode has the correct GOPATH. 
- open the terminal view
- echo $GOPATH in that terminal view to show the current GOPATH

This is working chaincode so we should now be able to deploy it, so we should be able to package it, so in the IBM Blockchain extension view, go to smart contract packages and click the `...` menu to package the smart contract.

If all goes well then it will package ready for use in the local fabric.

But you may have noticed that from a Go extension point of view this structure is not correct at all. Ideally the Go extension wants to see the go workspace rather than just chaincode source directory. So we try the following

```
$ GOPATH=~/mycode/testcc code ~/mycode/testcc/src/mycc
```

But we know this then won't allow us to package our smart contract. The solution is to use the workspace feature of vscode to link the chaincode folder to the root directory of the workspace

- right click in the main explorer window and select `Add Folder to Worksace...`
- drill down the tree to the folder containing the chaincode source code (in my example the folder is mycc) and press add
- Now you have an untitled workspace you might want to save this workspace so that when you open vscode again it will recreate this setup. Make sure you save it in a suitable directory. I store mine in ~/mycode/testcc
```
$ GOPATH=~/mycode/testcc code ~/mycode/
```
  
Now we can package our smart contract
- go to smart contract packages and click the `...` menu to package the smart contract
- you will be prompted as to which workspace folder to use, so select your chaincode source folder (in my example it is mycc)
- Now it should package successfully.

**Always look in the Output tab and select the `Blockchain` dropdown menu to see the files that have been packaged into the smart contract.**

you may also want to consider creating a `.vscode` directory in your project folder (in this example `tsproject`) with the file `extensions.json` and contents 
```
{
    "recommendations": [
        "ibmblockchain.ibm-blockchain-platform",
        "ms-vscode.go"
    ]
}
```
This would then get vscode to suggest you need to install the ibm blockchain platform extension and the go extension if that folder was opened in vscode if they have not already been installed.

## how to let vscode point to the go workspace (ie GOPATH) and still be able to package
TODO: need to add the folder /src/mycc to your workspace (ie that folder will appear as a root folder in the workspace)

