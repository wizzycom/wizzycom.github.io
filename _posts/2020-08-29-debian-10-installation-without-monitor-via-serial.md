---
title: Debian 10 installation without monitor via serial
date: 2020-08-29 17:29:55.000000000 +03:00
type: post
categories:
- How to
tags:
- Linux
permalink: "/debian-10-installation-without-monitor-via-serial/"
---
I tested some solutions I found over the internet, but some were too hard and time consuming and some others didn't work as expected.

The whole idea is to make changes to the .iso image or the image already transferred to a bootable USB stick, in order for grub to send all output to the serial port. I used many tools and many ways to do this and I found a simple solution that suited me. What I did was quite simple.

First, use [Rufus](https://rufus.ie/) (and only Rufus) to transfer the .iso image to your USB stick. Use the default settings and transfer the .iso image.

Then, there are some file changes. All the changes/additions are marked with bold. The position of the changes matters, so  be careful. Below are the final files that worked.

**isolinux/adtxt.cfg**

```
label expert
menu label E^xpert install
kernel /install.amd/vmlinuz
append priority=low vga=off console=ttyS0,115200n8 initrd=/install.amd/initrd.gz --- console=ttyS0,115200n8
include rqtxt.cfg
label auto
menu label ^Automated install
kernel /install.amd/vmlinuz
append auto=true priority=critical vga=off console=ttyS0,115200n8 initrd=/install.amd/initrd.gz --- console=ttyS0,115200n8
```

**isolinux/isolinux.cfg**

```
# D-I config version 2.0
# search path for the c32 support libraries (libcom32, libutil etc.)
serial 0 115200
console 0
path
include menu.cfg
default vesamenu.c32
prompt 0
timeout 0
```

**isolinux/txt.cfg**

```
label install
menu label ^Install
kernel /install.amd/vmlinuz
append vga=off console=ttyS0,115200n8 initrd=/install.amd/initrd.gz --- console=ttyS0,115200n8
```

So, make the changes, connect your serial cable on the first serial port of your system and set the serial port to

**Speed** : 115200

**Data bits** : 8

**Stop bits** : 1

**Parity** : None

Good luck
