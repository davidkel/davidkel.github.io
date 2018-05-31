# Using Windows
Hyperledger Composer doesn't officially support windows for developing business networks, client applications (including playground and rest-server) or for doing any operational aspects such as deployment or identity management. However it's perfectly doable and on Windows you have a choice on how to do it, it's entirely up to you.
Also this solution presented here, works with Windows Home, Professional and Enterprise. It doesn't require any special features in a specific Windows version. It will also work with windows 7 but you need to use the native solution as Windows Subsystem for Linux is not available prior to windows 10. This solution won't work if you are running windows in a VM. Virtual box (used here) doesn't support nested virtualisation.

## Business Network Development
You don't need a real fabric for the most part, you can use composer playground to develop your business network. Either use the [online playground](https://composer-playground.mybluemix.net) which doesn't require any installation at all (Just remember to export your BNA as it is all held in the local storage of your browser) or install it locally (see later about installing to composer) and just use the web browser profile. This allows you to develop and test a lot of the features of a business network, but to test other aspects such as client code or the rest server still requires a real fabric.

### Using a real fabric for development
Composer has no dependency on the architecture of a fabric network. It doesn't matter if it is single org, multiple org, 1 peer per org, 2 peers per org etc. All it needs is a connection profile describing the network (see composer document on connection profiles) with an appropriately configured client section to specify which org you are representing and optional timeouts. From there you can build cards containing appropriate credentials that allow access to the fabric network. From a developer point of view then it doesn't matter what the fabric network looks like, all we need is a fabric network, an appropriate connection profile and credentials. Or even better a pre-built composer business network card which provides an identity that has admin priviledges on the peer and the defined channel.

#### Hyperledger Fabric recommends git bash (msys, cygwin are similar)
**Warning: This is untested**
It's not so straight forward getting hyperledger fabric working well on windows. Hyperledger fabric do document using git bash as a command shell in order to run the various scripts and you can make it work, but it can be a problem to get it working. It does require docker for windows (not docker toolbox) which means it will only work on Windows 10 professional or enterprise.
Other people have been able to follow the hyperledger composer tutorial as well by modifying the docker-compose.yaml file and using git bash, if you prefer you can go down this route however the purpose of Hyperledger Composer is not to help you build fabric networks (you should be looking at Hyperledger Ledger Fabric documentation to do that) but how to develop Hyperledger Composer applications which are independent of a hyperledger fabric network but will help you create a connection profile used by hyperledger composer to connect to a specific network setup. So having a pre-working built network without the fuss seems to be a good approach.
If you do then the minimum that apparently is required is

- Windows 10 professional or enterprise
- Hyper-v enabled
- Docker for Windows
- Cygwin, git bash or msys

In order to be able to get a hyperledger fabric environment up and running you need to issue the following commands
```
export  COMPOSE_CONVERT_WINDOWS_PATHS=1
export  MSYS_NO_PATHCONV=1
```
to ensure all works with docker-compose and docker.

#### Installation guide
This will install a version of the composer-tools package, `fabric-dev-servers` which is identical to the one described in the single org tutorial.

1. Install [VirtualBox Version 5](https://www.virtualbox.org/wiki/Downloads)
2. Install [Vagrant](https://www.vagrantup.com/downloads.html) Version 1.9 or higher
3. create a directory called fabric-network
4. download the [FabricDevServer.zip](https://raw.githubusercontent.com/davidkel/ComposerVagrant/master/FabricDevServer/zipped/FabricDevServer.zip) file
5. unpack the zip file into your new fabric-network directory
6. change to that directory in a command prompt window
7. run the command `vagrant up` in the current directory to bring up the fabric-network for the first time. The fabric network will be the latest available version that has been published.
8. Note the file `PeerAdmin@fabric-network.card` in your current directory that gets created once the environment has finished starting up the fabric.

```
c:\Users\Dave\fabric-network> vagrant up
...
c:\Users\Dave\fabric-network> dir
...
PeerAdmin@fabric-network.card
```

9. It also provides you with a connection profile and the credentials for the peer and channel administrator (this fabric setup makes them both the same identity). If you wish to build the card yourself, you will find the connection profile in the your `fabric-network` directory called `fabric-network-connection.json` and the sub-directory `admin` has the credentials for Admin@org1.example.com identity copied there for easy access.

#### Available commands
There are some useful commands when using `vagrant`. You can also use the commands available in composer tools (to an extent). Make sure you always run these commands from your `fabric-network` directory.

| Command | description |
| ------- | ----------- |
| vagrant up | start up the fabric network |
| vagrant halt | to shutdown the fabric network |
| vagrant ssh -c 'docker ps' | invoke docker ps |
| vagrant ssh -c 'docker logs <container_name> | get the logs of a fabric node container |
| vagrant ssh -c 'teardownFabric.sh' | invoke the fabric tools commands |
| vagrant destroy | completely shutdown and remove the virtual machine |
| vagrant ssh | connect the the virtual machine and start a bash shell |

#### Single Org Tutorial differences
- Step One: You don't need to perform this step.
- Step Two: You could perform this step but you would have to use `vagrant ssh` to get a shell into the virtual machine where the fabric-network is running.
- Step Four/Step Five: The certificate and private key can be found in the `admin` subdirectory. 

## Installing the Composer applications
Choose which environment you prefer

1. Windows Subsystem For Linux (Windows 10 only)
2. Native Windows

### Using Windows Subsystem for Linux (WSL)
A great feature of windows is the ability to run native linux compiled binaries on windows (with exceptions, you couldn't get the docker daemon running in it for example). Even better is that you can install complete distributions that work the same way as a real linux distribution, such as ubuntu. I recommend the ubuntu distribution. You will need to check what version of Windows 10 you are using then follow appropriate instructions to install the windows subsystem for linux for that level of windows.

Once done, you can run this simple script to install other pre-reqs

```
(#!/bin/bash
#
# Licensed under the Apache License, Version 2.0 (the "License");
# you may not use this file except in compliance with the License.
# You may obtain a copy of the License at
#
# http://www.apache.org/licenses/LICENSE-2.0
#
# Unless required by applicable law or agreed to in writing, software
# distributed under the License is distributed on an "AS IS" BASIS,
# WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
# See the License for the specific language governing permissions and
# limitations under the License.
#

# Usage:
#
# ./prereqs-wsl.sh

# Exit on any failure
set -e

# Update package lists
echo "# Updating package lists"
sudo apt-add-repository -y ppa:git-core/ppa
sudo apt-get update

# Install Git
echo "# Installing Git"
sudo apt-get install -y git

# Install nvm dependencies
echo "# Installing nvm dependencies"
sudo apt-get -y install build-essential libssl-dev

# Execute nvm installation script
echo "# Executing nvm installation script"
curl -o- https://raw.githubusercontent.com/creationix/nvm/v0.33.2/install.sh | bash

# Set up nvm environment without restarting the shell
export NVM_DIR="${HOME}/.nvm"
[ -s "${NVM_DIR}/nvm.sh" ] && . "${NVM_DIR}/nvm.sh"
[ -s "${NVM_DIR}/bash_completion" ] && . "${NVM_DIR}/bash_completion"

# Install node
echo "# Installing nodeJS"
nvm install --lts

nvm use --lts
nvm alias default 'lts/*'

# Install python v2 if required
set +e
COUNT="$(python -V 2>&1 | grep -c 2.)"
if [ ${COUNT} -ne 1 ]
then
   sudo apt-get install -y python-minimal
fi

# Install unzip, required to install hyperledger fabric.
sudo apt-get -y install unzip

# Print installation details for user
echo ''
echo 'Installation completed, versions installed are:'
echo ''
echo -n 'Node:           '
node --version
echo -n 'npm:            '
npm --version
echo -n 'Python:         '
python -V
```

Files owned by windows can be accessed, eg for files on the C drive you find the root of the C drive here `/mnt/c`


### Using Native Windows.
Yes this is possible currently, however there is no guarantee this will continue to work and if it does break, because windows is not officially supported there will be no focus to fix it. This is different from the Windows Subsystem For Linux as the likelyhood is that if it works for linux then it will work in the Windows Subsystem for Linux.

1. download and install node 8 for windows from [node.js](https://nodejs.org/en/download/)
2. Open an administrative powershell window
3. install the windows tools using `npm install -g windows-build-tools`
3. install git from [git for windows](https://gitforwindows.org/)
4. ensure you have the following configured

```
git config --global core.autocrlf false
git config --global core.longpaths true
```

### Installing Composer
Either open a Windows subsystem for linux bash shell (if using the subsystem), or a standard command prompt (if using native) and then just follow the standard instructions for installing composer

## Developing business networks and client applications
I recommend using [Visual Studio Code](https://code.visualstudio.com/). If you do use WSL for development (and I would recommend this over using the native windows option), then you can find your files in the WSL shell under the initial path of `/mnt/c` for the Windows C drive, for example `/mnt/c/Users/dave/myfirstbna` if I had created my bna fille in `c:\Users\dave\myfirstbna`.

## And finally....
Want to try out the Hyperledger Composer Multiple Organisation Tutorial on Windows ?  Follow this [link to the next section](./multiorg.md)

