---
title: DHCP server in Debian 9
date: 2019-02-24 17:03:39.000000000 +02:00
type: post
categories:
- How to
tags:
- Debian
- DHCP
- Linux
permalink: "/dhcp-server-in-debian-9/"
---
These are the steps that I followed, in order to make a DHCP server in Debian 9 Stretch.

First of all we need to assign an IP address on our main ethernet interface. For this guide, I will use interface **enp0s2**. So we have to edit the file **/etc/network/interfaces** and add the following :

```
auto enp2s0
 allow-hotplug enp2s0
 iface enp2s0 inet static
         address 192.168.1.1/24
         dns-nameservers 8.8.8.8
         dns-search mydomain.com
```

Then we need to download our dhcp package.

```
sudo apt install isc-dhcp-server
```

After the package installation we backup the default configuration file and we create a new one for our setup

```
sudo mv /etc/dhcp/dhcpd.conf /etc/dhcp/dhcpd.conf.backup
sudo nano /etc/dhcp/dhcpd.conf
```

Just add the following on **/etc/dhcp/dhcpd.conf**

```
option domain-name "mydomain.com";
 option domain-name-servers 8.8.8.8;
 default-lease-time 3600;
 max-lease-time 7200;
 authoritative;
subnet 192.168.1.0 netmask 255.255.255.224 {
   range 192.168.1.10 192.168.1.253;
   option subnet-mask 255.255.255.0;
   option broadcast-address 192.168.1.255;
   option routers 192.168.1.254;

```

For ip reservations you can add the following on **/etc/dhcp/dhcpd.conf**

```
host station1 {
    option host-name "station1.mydomain.local";
    hardware ethernet 00:11:22:33:44:AA;
    fixed-address 192.168.1.100;
```

Last, we have to assing on which interface the DHCP server will listen. We edit the file **/etc/default/isc-dhcp-server** and we set the interfaces seperated by comma

```
INTERFACES="enp0s2"
```

Enable and start the DHCP service

```
sudo systemctl enable isc-dhcp-server.service
sudo systemctl start isc-dhcp-server.service
```

At anytime if you want to see the leases of the DHCP server, simply execute the following command

```
sudo dhcp-lease-list
```

Good luck
