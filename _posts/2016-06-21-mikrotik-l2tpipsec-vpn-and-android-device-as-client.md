---
title: Mikrotik L2TP/IPsec VPN and android device as client
date: 2016-06-21 19:29:47.000000000 +03:00
type: post
categories:
- How to
tags:
- Android
- Mikrotik
- VPN
permalink: "/mikrotik-l2tpipsec-vpn-and-android-device-as-client/"
---
Mikrotik should have **a real internet IP** to a certain interface. If it is located behind nat, the modem that provides internet access should be able to forward ipsec-esp packages. **If your mikrotik is behind an internet modem that does not forward ipsec-esp, then you should stop here.**

Your android smart phone must be in version 4 or newer in order to support **L2TP/IPsec**. The android client supports the following :

Authentication algorithm : sha1

Encryption algorithm : 3des

Diffie-Hellman : Group2 (modp1024)

Let's start by creating a PPP Profile on mikrotik.

```
ppp profile add name=ipsec_vpn local-address=192.168.2.1 dns-server=8.8.8.8
```

On local-address= we assing an IP address that will appear as the default gateway for the VPN clients. A good hint is to find a network that it is not used. Then we have to activate the L2TP server of the mikrotik and bind it with a PPP Profile.

```
interface l2tp-server server set enabled=yes default-profile=ipsec_vpn authentication=mschap1,mschap2
```

Once the the L2TP server is activated , we have to define the peering of IPSec and also the default ipsec policy. **WARNING :** On newer RouterOS versions,  **generate-policy** set to **yes** is not supported. On this case just use **generate-policy=port-override**

Older :

```
/ip ipsec policy set [ find default=yes ] src-address=0.0.0.0/0 dst-address=0.0.0.0/0 protocol=all proposal=default template=yes
/ip ipsec peer add address=0.0.0.0/0 port=500 auth-method=pre-shared-key secret=123456 exchange-mode=main-l2tp send-initial-contact=no nat-traversal=yes proposal-check=obey hash-algorithm=sha1 enc-algorithm=3des dh-group=modp1024 generate-policy=yes
```

Newer :

```
/ip ipsec policy set [ find default=yes ] src-address=0.0.0.0/0 dst-address=0.0.0.0/0 protocol=all proposal=default template=yes
/ip ipsec peer add address=0.0.0.0/0 port=500 auth-method=pre-shared-key secret=123456 exchange-mode=main-l2tp send-initial-contact=no nat-traversal=yes proposal-check=obey hash-algorithm=sha1 enc-algorithm=3des dh-group=modp1024 generate-policy=port-override
```

On secret = you should define the pre-shared key that must match the pre-shared key of the client (android phone). After we defined the peering, we must make some changes on the default ipsec proposal.

```
ip ipsec proposal set default auth-algorithms=sha1 enc-algorithms=3des pfs-group=modp1024
```

To complete the configuration, we need to add a user.

```
ppp secret add name=user password=pass service=l2tp profile=ipsec_vpn remote-address=192.168.2.2
```

On **user=** we define the user name and the user password on password=. On remote-address= we define the desired IP address that will be assigned to the client.

Next steps :

If our mikrotik has real internet IP to an interface and we have enabled firewalling, we must allow the UDP ports : 500, UDP: 1701, UDP: 4500 and Protocol 50: ipsec-esp

For the android client, we must set the following :

Name : Home VPN

Type : L2TP/IPSec PSK

Server address : real ip address of mikrotik

IPSec pre-shared key : the value that you set as secret=

Good luck.
