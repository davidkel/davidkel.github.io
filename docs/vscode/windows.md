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
If you have Windows feature update 2004 or higher such as 20H2 then you can skip this step. If you have 1903 or 1909 then microsoft has backported WSL2 to these levels but you need to check whether you have the appropriate update applied, see https://ubuntu.com/blog/ubuntu-on-wsl-2-backported-to-windows-10-1909-extending-reach for more info. The update is KB4566116, and if you don't have it then you can download it from https://www.catalog.update.microsoft.com/Search.aspx?q=KB4566116 and apply it manually.

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
~~If you have Windows feature update 2004 then you can install the Docker Desktop stable version. It currently checks you are at 2004 so will fail if you are on Windows 1903 or 1909. In that case you need to install Docker Desktop edge version. This will install but when you run it will produce an error message but should start anyway~~

I've not tested this, but Docker desktop 3.0 for windows in the stable channel is now available and I would hope that this version works with Windows Home 1903 or 1909 (if you are still one of the unfortunate windows users who haven't been updated to 2004 or 20H2 yet). If you still have problems then I would suggest looking at the edge release again.

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
There have been reports of issues working with fabric due to time differences between the windows host and the WSL2 virtual machine being used.

There is a known issue with WSL2 in the fact that it suffers from clock skew. This is where the clock inside WSL2 doesn't match the clock on your host windows machine. I've also seen this problem with VirtualBox VMs as well. This becomes a real problem if you are running fabric inside a WSL2 instance but trying to communicate with it from the host machine as fabric, for security reasons, needs to be within a threshold of time difference in order to allow the external client to interact.

I'm not sure how a clock skew occurs as I can't recreate it, but I have 

There are many ways to fix the clock skew which can be found on google, the basic premise is that you tell your wsl instance to synchronise. What I don't know is whether you need to do it for all WSL2 instance or just one of your instances as each instance my share the underlying WSL2 lightweight VM and kernel. There are multiple options to sync the clock

* inside of a WSL2 instance (and using a ubuntu instance)
  * sudo ntpdate ntp.ubuntu.com &>/dev/null &
  * sudo hwclock -s
* outside of a wsl2 instance (ie in a windows CMD prompt or Powershell prompt)
  * wsl -u root sh -C "hwclock -S" (this would use the default instance which may not be what you want)
  * wsl -d docker-desktop -u root sh -C "hwclock -S" (to do it specifically for the docker desktop)

One idea I really liked was to have an automated task that ran to fix the clock skew on resume of your laptop. Details can be found at [here](https://stuartleeks.com/posts/fixing-clock-skew-with-wsl-2/)
