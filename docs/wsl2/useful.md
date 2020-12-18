# Using WSL2 for your main linux development

This page is going to be a collection of experiences for using WSL2 as my main linux development environment. I moved from using VirtualBox because of

* VirtualBox VMs Would abort during suspend or shutdown
* Upgrading to newer versions of VirtualBox sometimes wouldn't start existing VMs anymore
* Trying to display the desktop environment on a 4k monitor or QHD laptop screen was a challenge to get it to be easily readible (I do note that VirtualBox was much better than VMWare when I tried the 2 but Linux desktops were not great for HiDPi type environments and only the more heavyweight desktops were getting better in that regard)
* When displaying the desktop the performance wasn't always great
* When using a windows XServer I would find that sometimes performance was poor (maybe due to VirtualBox network latency) and sometimes it wasn't but I couldn't find an easy way to influence and keep NAT involved
* VirtualBox doesn't play well with hyper-v and even 6.1.16 still doesn't do a great job. This meant not being able to help with testing windows specific stuff that requires Docker For Windows without having a dual boot mechanism to switch between the 2.

## Moving to WSL2

WSL2 is not without it's own quirks but I have gone with Ubuntu 18.04 as my starting environment. The quirks I have come across so far are

* the /etc/hosts is a direct copy of the windows version in system32/drivers/etc 
* Multiple WSL2 distrubitions are not network isolated
* WSL2 network is not NAT controlled so things differ in the way I worked with my virtualbox vm
* WSL2 paths also include the full windows path
* shutdown and reboot commands don't work
* There is no systemd, so you need to start the docker daemon manually (but you can run full docker inside WSL2)
* You have access to your local filesystem inside WSL2 (not the other way round yet, but that feature will come in future feature releases of windows)
* wsl has a command line that allows you to create and remove and control running instances of WSL2

### GUI applications

At this time I use VcXsrv on windows and run the GUI app from within a WSL2 shell. This has worked very well for the GUI apps I use

* VSCode
* git-cola
* firefox
* gedit
  
I've not tried a full desktop environment yet from WSL2 but I would think it should work quite well once you install an appropriate desktop environment into wsl2

I need to start the XServer with access checking disabled (only cause I am too lazy to work out how to only allow the WSL2 instance). I also fix the Xserver to a specific display number just to be sure, then I have the following script as part of my `.bashrc` to ensure all gui apps go to my desktop

```bash
echo "exporting display"
export DISPLAY=$(awk '/nameserver / {print $2; exit}' /etc/resolv.conf 2>/dev/null):5.0
export LIBGL_ALWAYS_INDIRECT=1
```

or 

```bash
echo "exporting display"
winip=$(ip route | grep default | awk '{print $3}')
export DISPLAY=$winip:5.0
export LIBGL_ALWAYS_INDIRECT=1
```

I can even share the clipboard between windows and linux apps (although you need to know the rules of when the clipboard might get overwritten otherwise you might think it wasn't working)

### Docker
Docker installs and you configure it in the same way you do in any other linux environment, but because WSL2 doesn't have systemd. All you need to do is start the docker daemon so another piece of script for my `.bashrc`

```bash
RUNNING=`ps aux | grep dockerd | grep -v grep`
if [ -z "$RUNNING" ]; then
    sudo dockerd > /dev/null 2>&1 &
    disown
fi
```

### Clock Skew
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


### AppImages (not specific to WSL2 but wanted to mention them)
I love these, just download and run. Shame VSCode have refused to create them :-( VSCodium however is creating them

```bash
~$ ./VSCodium-1.49.3-1601685407.glibc2.16-x86_64.AppImage &
```

### Other advantages

**Vagrant**
I used to like vagrant (not their config file and it's dependency on having to be written in Ruby) and virtual box images for vagrant were plentiful but there are alternatives

* WSL2 has a command line to allow you to control and run different instances
* Vagrant does work with hyper-v and you can have hyper-v enabled as well as WSL2
* Multipass is a nice simple tool for creating hyper-v linux VMs quickly

**Resource Usage:**
 WSL2 processes make use of resource in a similar manner to windows processes, they have full access to all the CPUs/Memory and no diskspace limitations. They seem to share well with all other windows processes (well, I've not had a problem yet) making them very responsive. You can restrict a WSL2 instance's resources if necessary

> TODO: Need to put the info in here

**Speed:**
 Open a shell and WSL2 is fully ready for use. It's a lot faster than resuming a VM from suspend or starting from new.

**Snapshoting:**
 You can use the WSL CLI to export an instance, which should be equivalent to creating a snapshot of your setup. Not tried it yet but must do it on my current environment.

**A more Complete environment:**
 With WSL2 and Hyper-v enabled. I can run multiple linux VMs and WSL2 instances, I can use docker for windows outside of a linux environment.
 There is a `But`. Hyper-v is still rubbish at Linux VM Guis (not tried windows guis)

**Other:**
 WSL2 uses a lightweight VM using hyper-v technology. hyper-v technology is a type 1 hypervisor and this would hopefully mean that it should be able to run linux apps at near native speed ie non of the clever trickery employed by type 2 hypervisors such as binary translation in VMWare. Also hope it means that there isn't a concern about having to add support for newer linux kernels.

### Disadvantages

There are some disadvantages and these are around running full distros with desktops in hyper-v. Basically it's crap. Microsoft, working with Canonical, have created ubuntu images with pre-configured xrdp, but my experience with xrdp is that although it supports passthrough of audio, the gui is just not as responsive compared to X2Go or Tiger VNC. If you don't want to use these images then the only alternative is to set up something yourself which isn't too much of a hassle but rather than having a native display driver environment employed by VirtualBox and VMWare, you are stuck with a remote display technology. These display driver technologies are far more comprehensive such as 3d support etc but when I last tried still weren't great with HiDPI and 4K monitors


## Hints and Tips

### Nested Virtualisation
Here is a useful tip for hyper-v virtual machines (not for WSL2 machines), if you want to have nested virtualisation you need to enable it specifically while the VM is stopped of course.

```
(Admin Powershell) Set-VMProcessor -VMName <VM Name> -ExposeVirtualizationExtensions $true
```
