# Summary of remote desktop environments

* VNC
* X2Go
* X Servers
  * VcXSrv
  * Cygwin X
  * XMing
* xRdp

## VNC
Use Tiger VNC

create `~/.vnc/xstartup`

```bash
#!/bin/sh

#xrdb $HOME/.Xresources
#xsetroot -solid grey
x-terminal-emulator -geometry 80x24+10+10 -ls -title "$VNCDESKTOP Desktop" &
x-window-manager &
# Fix to make GNOME work
export XKL_XMODMAP_DISABLE=1
#/etc/X11/Xsession
unset DBUS_SESSION_BUS_ADDRESS
exec /usr/bin/mate-session &
```

## X2Go


## XServers
My preference is VcXsrv its a fork of XMing which isn't maintained anymore. Cygwin X isn't very performant compared to this, plus to supports clipboard exchange and is also used by X2Go. There are non free XServers for windows and as such I haven't tried them to see how good they are.

## xRdp

```bash
sudo apt install xserver-xorg-core
sudo apt install xserver-xorg-input-all
sudo apt install xrdp
sudo adduser xrdp ssl-cert
sudo adduser dave xrdp
sudo ufw allow 3389/tcp
sudo sed -i.bak '/fi/a #xrdp multiple users configuration \n mate-session \n' /etc/xrdp/startwm.sh


sudo add-apt-repository ppa:martinx/xrdp-next
sudo apt upgrade -y

sudo /etc/init.d/xrdp restart
```

