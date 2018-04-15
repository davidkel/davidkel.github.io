# Using Windows 10
Hyperledger Composer doesn't officially support windows for developing business networks or for doing any operational aspects such as deployment or identity management. However it's perfectly doable and on Windows you have so much choice on how to do it, it's entirely up to you.
Also this solution presented here, works with Windows Home, Professional and Enterprise. It doesn't require any special features in a specific Windows version.

## Business Network Development
You don't need a real fabric for the most part, you can use composer playground to develop your business network. Either use the [online playground](https://composer-playground.mybluemix.net) which doesn't require any installation at all (Just remember to export your BNA as it is all held in the local storage of your browser) or install it locally (see later about installing to composer) and just use the web browser profile. This allows you to develop and test a lot of the features of a business network but currently can't do the following

1. Test predefined queries
2. Test client code connected to the business network

Obviously you would write unit tests for your client simulating connecting to a business network using the embedded connector. 

### Using a real fabric for development
Composer has no dependency on the architecture of a fabric network. It doesn't matter if it is single org, multiple org, 1 peer per org, 2 peers per org etc. All it needs is a connection profile describing the network (see composer document on connection profiles) with an appropriately configured client section to specify which org you are representing and optional timeouts. From there you can build cards containing appropriate credentials that allow access to the fabric network. From a developer point of view then it doesn't matter what the fabric network looks like, all we need is a fabric network, an appropriate connection profile and credentials. Or even better a pre-built composer business network card which provides an identity that has admin priviledges on the peer and the defined channel.

It's not so straight forward getting hyperledger fabric working well on windows. Hyperledger fabric do document using git bash as a command shell in order to run the various scripts and you can make it work, but it can be a problem to get it working. Not least because at the end of the day, Hyperledger fabric from dockerhub are linux containers so still require a virtual machine running a full distribution of linux in order to run these images.

#### Installation guide
This will install a version of the composer-tools package, `fabric-dev-servers` which is identical to the one described in the single org tutorial.

1. Install [VirtualBox Version 5](https://www.virtualbox.org/wiki/Downloads)
2. Install [Vagrant](https://www.vagrantup.com/downloads.html) Version 1.9 or higher
3. create a directory called singleorg-blockchain
4. download the [FabricDevServer.zip](https://raw.githubusercontent.com/davidkel/ComposerVagrant/master/FabricDevServer/zipped/FabricDevServer.zip) file
5. unpack the zip file into your new singleorg-blockchain directory
6. change to that directory in a command prompt window
7. run the command `vagrant up` in the current directory to bring up the fabric-network for the first time
8. Note the file `PeerAdmin@singleorg-blockchain.card` in your current directory that gets created once the environment has finished starting up the fabric.

```
c:\Users\Dave\fabric-network> vagrant up
...
c:\Users\Dave\fabric-network> dir
...
PeerAdmin@singleorg-blockchain.card
```

#### Available commands

| Command | description |
| ------- | ----------- |
| vagrant up | start up the fabric network |
| vagrant halt | to shutdown the fabric network |
| vagrant ssh -c 'docker ps' | invoke commands such as docker ps or docker logs |
| vagrant ssh -c 'teardownFabric.sh' | invoke the fabric tools commands |
| vagrant destroy | completely shutdown and remove the virtual machine |

## Installing the Composer applications
Choose which environment you prefer

1. Windows Subsystem For Linux
2. Native Windows

### Setting Windows Subsystem for Linux (WSL)
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
Yes this is possible
1. install nvm-windows from [nvm-windows releases](https://github.com/coreybutler/nvm-windows/releases)
2. use nvm to install node 8 
```
nvm install 8
```
3. install the windows tools using `npm install -g windows-build-tools`
4. install git from [git for windows](https://gitforwindows.org/)
5. ensure you have the following configured

```
git config --global core.autocrlf false
git config --global core.longpaths true
```

### Installing Composer
Just follow the standard instructions for installing composer

## Developing business networks and client applications
I recommend using [Visual Studio Code](https://code.visualstudio.com/). If you do use WSL for development (and I would recommend this over using the native windows option), then you can find your files in the WSL shell under the initial path of `/mnt/c` for the Windows C drive, for example `/mnt/c/Users/dave/myfirstbna`.
