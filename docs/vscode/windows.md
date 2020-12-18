# Running on Windows
Although it has been possible to run the vscode extension on Windows Pro/Enterprise, it requires the use of hyper-v and you need to set up docker with windows shares in order to work.

With the advent of WSL2, the process required to get the IBP extension working is better and now you can have the extension running on Windows Home as well.

## Installing the pre-reqs to enable Docker on Windows

- ensure you have Windows feature update 2004 or KB4566116 installed
- enable WSL2
- set WSL2 to be the default WSL version
- install the latest kernel component
- install Docker Desktop for Windows (stable or edge depending on whether you have Windows 2004 or not)

### Windows feature update 2004 or not
If you have Windows feature update 2004 then you can skip this step. If you have 1903 or 1909 then microsoft has backported WSL2 to these levels but you need to check whether you have the appropriate update applied, see https://ubuntu.com/blog/ubuntu-on-wsl-2-backported-to-windows-10-1909-extending-reach for more info. The update is KB4566116, and if you don't have it then you can download it from https://www.catalog.update.microsoft.com/Search.aspx?q=KB4566116 and apply it manually.

### enable WSL2
Open a Powershell window as adminstrator and input
```
Enable-WindowsOptionalFeature -Online -FeatureName VirtualMachinePlatform -NoRestart
```

or you can go to `Control Panel` select `Programs and Features` click on `Turn Windows features on or off` and check the `Virtual Machine Platform`
checkbox is ticked and press OK

### Set WSL2 to be the default WSL version
Open a Powershell or Command Prompt as Administrator and enter the following
```
wsl.exe --set-default-version 2
```

### Install the latest kernel component
visit https://docs.microsoft.com/en-gb/windows/wsl/wsl2-kernel and download and install the latest kernel component

### Install Docker Desktop
If you have Windows feature update 2004 then you can install the Docker Desktop stable version. It currently checks you are at 2004 so will fail if you are on Windows 1903 or 1909. In that case you need to install Docker Desktop edge version. This will install but when you run it will produce an error message but should start anyway

## Installing the IBP extension
This should be documented here https://github.com/IBM-Blockchain/blockchain-vscode-extension
but this there are some pre-reqs which are easy to miss defined here https://github.com/IBM-Blockchain/blockchain-vscode-extension#additional-requirements-for-windows

In summary
- Install Node 10 and make sure it is on your PATH
- Install Python and the Microsoft compilers. This is easily done by installing the npm module `windows-build-tools` as described here https://github.com/felixrieseberg/windows-build-tools#windows-build-tools
- Optionally install OpenSSL v1.0.2 Win64 version if you plan to debug node-sdk contracts. Ensure you don't choose the light version and install it into C:\OpenSSL-Win64 directory

### A note about Python
The windows build tools installs Python 2.7 which is currently still ok for use with node-gyp but may stop working in future.

### WSL2 clock skew causes problems for fabric running in docker for windows
