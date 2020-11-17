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

### AppImages (not specific to WSL2 but wanted to mention them)
I love these, just download and run. Shame VSCodium stopped producing them and VSCode have refused to create them :-(

```bash
~$ ./VSCodium-1.49.3-1601685407.glibc2.16-x86_64.AppImage &
```

### Other advantages

**Vagrant**
I used to like vagrant (not their config file and it's dependency on having to be written in Ruby) and virtual box images for vagrant were plentiful but there are alternatives

* WSL2 has a command line to allow you to control and run different instances
* Vagrant does work with hyper-v and you can have hyper-v enabled as well as WSL2
* Multipass is a nice simple tool for creating hyper-v linux VMs quickly

**Resource Usage**
WSL2 processes make use of resource in a similar manner to windows processes, they have full access to all the CPUs/Memory and no diskspace limitations. They seem to share well with all other windows processes (well, I've not had a problem yet) making them very responsive. You can restrict a WSL2 instance's resources if necessary

> TODO: Need to put the info in here

**Speed**
Open a shell and WSL2 is fully ready for use. It's a lot faster than resuming a VM from suspend or starting from new.

**Snapshoting**
You can use the WSL CLI to export an instance, which should be equivalent to creating a snapshot of your setup. Not tried it yet but must do it on my current environment.

**A more Complete environment**
With WSL2 and Hyper-v enabled. I can run multiple linux VMs and WSL2 instances, I can use docker for windows outside of a linux environment.

**other**
WSL2 uses a lightweight VM using hyper-v technology. hyper-v technology is a type 1 hypervisor and this would hopefully mean that it should be able to run linux apps at near native speed ie non of the clever trickery employed by type 2 hypervisors such as binary transaction in VMWare. Also hope it means that there isn't a concern about having to add support for newer linux kernels.

### Disadvantages

There are some disadvantages and these are around running full distros with desktops in hyper-v. Basically it's crap. hyper-v working with canonical have created ubuntu images with pre-configured xrdp, but my experience with xrdp is that although it supports passthrough of audio, the gui is just not as responsive when I've used X2Go or Tiger VNC. Plus if you don't go with the small number of prebuilt options in hyper-v. Trying to get xrdp configured working and stable is not a trivial. Of course VirtualBox and VMWare do this much better apart from the handling of 4k screens and HiDPI screens.