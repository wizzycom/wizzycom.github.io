---
title: Running TP-Link Easy Smart Configuration Utility on linux
date: 2021-12-11 15:54:31.000000000 +02:00
type: post
categories:
- How to
tags: []
permalink: "/running-tp-link-easy-smart-configuration-utility-on-linux/"
---
I have been looking for a way to run this utility on linux (because it is a java app) and I found [this guide](https://shred.zone/cilla/page/383/setting-up-tp-link-tl-sg108e-with-linux.html) explaining how to achieve this.

By the way, I use Arch Linux.

So, first, you will need to download the latest x64 Oracle JRE for linux from [here](https://java.com/en/download/). In my case, **Linux x64**.

Create a new Java folder on your home directory and extract the Oracle JRE archive there:

```
mkdir ~/Java
tar -xvf ~/Download/jre-8u311-linux-x64.tar.gz -C ~/Java
```

Then download the Easy Smart Configuration Utility from [here](https://www.tp-link.com/us/support/download/).

First, **you need to install the utility on a Windows system**, in order to extract the executable file. After the installation, go to **C:\\Program Files (x86)\\TP-LINK\\Easy Smart Configuration Utility and grab Easy Smart Configuration Utility.exe**.

Create a new folder on your linux system user home directory, paste the **Easy Smart Configuration Utility.exe** and rename it to **easysmart.jar**

```
mkdir ~/Easysmart
(paste file)
mv Easy Smart Configuration Utility.exe easysmart.jar
```

Then, add the following lines to your .bashrc file:

```
# TP-Link Easysmart
function easysmart {
    cd ~/Easysmart
    sudo iptables -t nat -A PREROUTING -p udp -d 255.255.255.255 --dport 29809 -j DNAT --to XXX.XXX.XXX.XXX:29809
    ~/Java/jre1.8.0_311/bin/java -jar ~/Easysmart/easysmart.jar
    sudo iptables -t nat -D PREROUTING -p udp -d 255.255.255.255 --dport 29809 -j DNAT --to XXX.XXX.XXX.XXX:29809
    cd ~
}
```

The iptable commands are handling the switch discovery by adding a prerouting rule, which is removed when you exit the utility. Don't forget to replace XXX.XXX.XXX.XXX with your system's IP address.

Lastly, by typing **easysmart**, the utility will execute and will search all network interfaces for a switch.

Good luck
