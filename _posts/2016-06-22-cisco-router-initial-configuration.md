---
title: Cisco router initial configuration
date: 2016-06-22 17:52:07.000000000 +03:00
type: post
categories:
- How to
tags:
- Cisco
permalink: "/cisco-router-initial-configuration/"
---
There is no golden rule on this. Everyone has it's own way to do a basic configuration on a cisco router. Here is mine.

Some routers are pre-configured by cisco. The first time that the router powers up, it will ask for a username and password which is always cisco/cisco. This is one-time-password. If you login from console and you don't change this, then you will be locked out. In case of pre-configured router, I always erase the running configuration by issuing :

```
write erase
reload
```

After the reload, the router will boot without any configuration. so this is where I start. First thing are the passwords and access to the router

```
en
conf t
hostname my_router
no ip domain-lookup
ip domain name my_domain
service password-encryption
crypto key generate rsa 2048
enable secret a-password
username a-username privilege 15 secret a-password
line vty 0 4
login local
transport input ssh
transport outpt telnet ssh
line con 0
login local
```

I set time, timezone-summertime and logging if applicable

```
clock timezone ATHENS 2 0
clock summer-time ATHENS recurring last Sun Mar 3:00 last Sun Oct 4:00
service timestamps debug datetime msec localtime show-timezone
service timestamps log datetime msec localtime show-timezone
```

Everything else does not belong to the initial config :-)

The above configuration can be applied also on switches
