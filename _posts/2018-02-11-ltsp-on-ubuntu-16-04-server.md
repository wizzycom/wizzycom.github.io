---
title: LTSP on ubuntu 16.04 server
date: 2018-02-11 17:20:53.000000000 +02:00
type: post
categories:
- How to
tags:
- Linux
- LTSP
- Ubuntu
permalink: "/ltsp-on-ubuntu-16-04-server/"
---
LTSP ( **L** inux **T** erminal **S** erver **P** roject), adds thin client support to Linux servers. Just imagine PCs without a hard drive, running linux over the network. Pretty cool!

So what do we need to accomplish that ?

We just need a DHCP service (with specific options like 66, 67, 17), a linux box running the  LTSP server and some client PCs connected to the same network. You will find many guides on the internet on how to install a LTSP server. All guides install the DHCP server on the same box. Also they are using DHCP proxy etc. My case has a DHCP server working on mikrotik routerOS.

So, in this guide, I assume that my **local network is 192.168.1.0/24**, my mikrotik device is the gateway of the network and has an interface ( **ether1**) with IP address **192.168.1.1** and my linux box has the ip address **192.168.1.10 and runs ubuntu server 16.04**.

Lets set up the DHCP first. On routerOS apply the following

```
/ip pool add name=lab_pool ranges=192.168.1.11-192.168.1.254
/ip dhcp-server add name=lab interface=ether1 address-pool=lab_pool bootp-support=static
/ip dhcp-server network add address=192.168.1.0/24 gateway=192.168.1.1 netmask=24 dns-server=8.8.8.8 domain=domain.local next-server=192.168.1.10 boot-file-name=ltsp/amd64/pxelinux.0
```

Now our DHCP server is ready. Lets log in to the linux box and install LTSP server with the following commands. For the server :

```
sudo apt-get install ltsp-server-standalone
```

For the client environment :

```
sudo ltsp-build-client
```

The above command will install by default an 64bit ubuntu image. **If you prefer a 32bit image** then run the following :

```
sudo ltsp-build-client --arch i386
```

Reboot the server and connect a client PC on the network. Set it up to boot from network. If everything is working correctly, you will see a login screen on the client PC, but you will not be able to login. This is due to a bug on the nbd-server that cannot authenticate clients accessing the network storage and there is no gnome-shell installed on the client image we created.

To overcome the first issue until is fixed, we need to make some changes to files **/etc/nbd-server/conf.d/ltsp\_amd64.conf** and **/etc/nbd-server/conf.d/swap.conf**. On both files, comment out the following line :

**authfile = /etc/ltsp/nbd-server.allow**

For the second issue, we have to install gnome in our client image.

**You should always follow the procedure bellow if you want to upgrade the client image or install new packages.**

So, first we make sure that the variable LTSP\_HANDLE\_DAEMONS=false is active. This will prevent the server from restarting its own daemons when upgrading the chroot:

```
export LTSP_HANDLE_DAEMONS=false
```

Then we chroot to our image and we mount /proc.

```
sudo chroot /opt/ltsp/amd64
mount -t proc proc /proc
```

Now we can upgrade or install new packages, like gnome.

```
apt-get update && apt-get dist-upgrade
apt-get install gnome-shell
```

Make sure that there are no errors when you install packages. Also you might notice a message like **Can not write log, openpty() failed (/dev/pts not mounted?)**. This is just a cosmetic bug.

After the upgrade or package installation, exit the chroot :

```
exit
```

Run the following, if there was a kernel upgrade on the client image.

```
sudo ltsp-update-kernels

```

Unmount /proc and rebuild the client image.

```
sudo umount /opt/ltsp/amd64/proc
sudo ltsp-update-image
```

Reboot any client PC to load the new image. You can login on the client PC with the user that you log in on the server linux box. On that box you can create more users and they will be able to login from the client PCs. All client data are saved on the linux box, in every user's home folder.

Good luck!
