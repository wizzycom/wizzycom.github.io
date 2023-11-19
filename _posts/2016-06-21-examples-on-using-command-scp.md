---
title: Examples on using command SCP
date: 2016-06-21 17:49:44.000000000 +03:00
type: post
categories:
- How to
tags:
- Linux
permalink: "/examples-on-using-command-scp/"
---
The command SCP (Secure copy) allows us to copy - file transfer over SSH, providing the same level of security that SSH provides. Let's see some examples :

Copy of the file test.txt to a remote system on path /var/www

```
scp test.txt your_username@remotesystem.ip:/var/www
```

Copy of the file test.txt which is on /usr/src to a remote system on path /var/www

```
scp your_username@remotesystem.ip:/var/www/test.txt /usr/src
```

Copy of a folder named test to a remote system on path /var/www

```
scp -r test your_username@remotesystem.ip:/var/www
```

