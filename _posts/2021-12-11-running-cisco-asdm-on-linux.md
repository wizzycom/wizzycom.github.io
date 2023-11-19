---
title: Running Cisco ASDM on linux
date: 2021-12-11 15:36:29.000000000 +02:00
type: post
categories:
- How to
tags: []
permalink: "/running-cisco-asdm-on-linux/"
---
I have been looking for some months on how to do this. Searching over the internet, there many old guides that most of them do not work, except [this one](https://www.labsrc.com/running-cisco-asdm-under-linux/).

By the way, I use Arch Linux.

So, first, you will need to download the latest x64 Oracle JRE for linux from [here](https://java.com/en/download/). In my case, **Linux x64**.

Create a new Java folder on your home directory and extract the Oracle JRE archive there:

```
mkdir ~/Java
tar -xvf ~/Download/jre-8u311-linux-x64.tar.gz -C ~/Java
```

In order to connect to an ASA system, you can run the following :

```
~/Java/jre1.8.0_311/bin/javaws https://xxx.xxx.xxx.xxx/admin/public/asdm.jnlp
```

where **xxx.xxx.xxx.xxx** is the management IP of the ASA

If the connection is successful, a desktop shortcut will be created on **~/Desktop**. You will, need to make a copy of this shortcut to bypass a security issue which disables the launcher after every use. Every successful connection, will create a desktop shortcut.

Here, there are two options. You just decide to live with this, or disable this option and create trusted entries for every system that you connect to. I have chosen the second option, I don't want any desktop shortcuts.

So, we have to make some changes to our Java environment. First, we have to disable the shortcut creation on java console by running:

```
~/Java/jre1.8.0_311/bin/jcontrol
```

On tab Advanced, select Never allow in option Shortcut creation. Then on tab security, add the ASA url as an exception e.g. **https://xxx.xxx.xxx.xxx**

If you have to many systems, or you need to exclude a subnet, you can edit the following file:

```
~/.java/deployment/security/exception.sites
```

Lastly, you can create a command-line shortcut in order to easily execute ASDM. You can find some info [here](https://www.labsrc.com/running-cisco-asdm-under-linux/) or [here](https://www.youtube.com/channel/UCSlW3CAwEX3yoyswZOMCFig).

This is achieved by adding the following to your user's **.bashrc** file:

```
# Cisco ASDM Launch Function
function asdm {
~/Java/jre1.8.0_311/bin/javaws -Xnosplash -wait https://$1/admin/public/asdm.jnlp
}
```

Good luck
