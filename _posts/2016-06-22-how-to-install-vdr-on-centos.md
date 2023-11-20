---
title: How to install VDR on CentOS
date: 2016-06-22 17:51:35.000000000 +03:00
type: post
categories:
- How to
tags:
- CentOS
- Linux
- VDR
permalink: "/how-to-install-vdr-on-centos/"
---
VDR is an application that allows us to do video streaming using as source a DVB-S or DVB-T adapter. This guide was tested on CentOS 6.4 x64. I used VDR version 1.7.22 and version 0.5.2 of streamdev server plugin. A prerequisite is that the system must have installed a DVB adapter.

Let's install some groups and packages

```
yum groupinstall "Additional Development" "Development tools"
yum install fontconfig fontconfig-devel freetype freetype-devel gettext gettext-devel libcap libcap-devel libjpeg libjpeg-devel
```

Create the folder structure

```
mkdir /usr/local/vdr
mkdir /usr/local/vdr/bin
mkdir /usr/local/vdr/lib
mkdir /usr/local/vdr/etc
mkdir /usr/local/vdr/etc/vdr
mkdir /usr/local/vdr/etc/vdr/plugins
mkdir /usr/local/vdr/spool
mkdir /usr/local/vdr/spool/video
mkdir /usr/local/vdr/spool/epg
```

Download, decompress and compile VDR and stream-dev plugin

```
cd /usr/src
wget ftp://ftp.tvdr.de/vdr/Developer/vdr-1.7.22.tar.bz2
tar -xvf vdr-1.7.22.tar.bz2
cd vdr-1.7.22
cd PLUGINS/src
wget http://projects.vdr-developer.org/attachments/download/953/vdr-streamdev-0.5.2.tgz
tar -xvf vdr-streamdev-0.5.2.tgz
cd ../../
make
make plugins
cd PLUGINS/src/streamdev-0.5.2
make
cd ../../../
```

If the compile procedure fails with error on vdr.c, then edit the file vdr.c and add the following line after the line #include <stdlib.h> and run make again.

```
#include <linux/types.h>
```

Move files to the correct paths

```
cp vdr /usr/local/vdr/bin/vdr
cp *.conf /usr/local/vdr/etc/vdr/
cp PLUGINS/lib/* /usr/local/vdr/lib/
cp PLUGINS/src/streamdev-0.5.2/streamdev-server/ /usr/local/vdr/etc/vdr/plugins/ -R
```

Before we proceed we have to install package dvb-utils which is available at Atrpms repo. So we have to create the file atrpms-stable.conf on /etc/yum.repos.d

```
nano /etc/yum.repos.d/atrpms-stable.repo
```

...and paste the following

```
[atrpms-stable]
name=RHEL 6 - atrpms-stable - $releasever - $basearch
baseurl=http://dl.atrpms.net/el6-$basearch/atrpms/stable/
gpgcheck=1
enabled=1
priority=20
exclude=*release
```

Save the file and install the GPG-KEY for the repo.

```
rpm --import http://packages.atrpms.net/RPM-GPG-KEY.atrpms
```

Install the dvb-utils

```
yum install dvb-utils
```

VDR does not have an init-script, so we have to create one. Create a file named runvdr on /etc/init.d

```
nano /etc/init.d/runvdr
```

...and add the following

```
#!/bin/bash
### BEGIN INIT INFO
# Provides: VDR
# Required-Start: $network
# Required-Stop: $network
# Default-Start: 3 5
# Default-Stop: 0 1 2 6
# Description: Start, Stop or Restart VDR
### END INIT INFO

CONFIG_DIR=/usr/local/vdr/etc/vdr
LIB_DIR=/usr/local/vdr/lib
VIDEO_DIR=/usr/local/vdr/spool/video
EPG_DIR=/usr/local/vdr/spool/epg

case "$1" in
 start)
 echo "Starting VDR"
 /usr/local/vdr/bin/vdr -d -D 0 --port=0 --video=$VIDEO_DIR --epgfile=$EPG_DIR --config=$CONFIG_DIR --lib=$LIB_DIR -P streamdev-server
 ;;
 stop)
 echo "Shutting down VDR"
 killall -q vdr
 ;;
 restart)
 echo "Restart VDR"
 $0 stop
 sleep 5
 $0 start
 ;;
 status)
 echo "Status VDR"
 ps -A | grep -q -w vdr && echo "...is running" || echo "...is not running"
 ;;
 *)
 echo "Usage: $0 {start|stop|restart|status}"
 exit 1
 ;;
esac
exit 0
```

Save the file and give it correct permissions

```
chmod +x /etc/init.d/runvdr
chkconfig --add runvdr
chkconfig runvdr on
```

Finally we should allow to have access to the service and to create the channels.conf file containing the channels that we would like to stream. So edit the file /usr/local/vdr/etc/vdr/plugins/streamdev-server/streamdevhosts.conf and change :

```
#0.0.0.0/0 # any host on any net (DON'T DO THAT! USE AUTHENTICATION)

to

0.0.0.0/0 # any host on any net (DON'T DO THAT! USE AUTHENTICATION)
```

Also edit the file /usr/local/vdr/etc/vdr/svdrphosts.conf and change  :

```
#0.0.0.0/0 # any host on any net (USE THIS WITH CARE!)

to

0.0.0.0/0 # any host on any net (USE THIS WITH CARE!)
```

The contents of channels.conf file are depending on the service that will like to stream. Now it's up to you. In my case I need to stream terrestrial TV. So I added the following :

Edit the file /usr/share/dvb/dvb-t/gr-Athens (also other country files exist) and add :

```
T 674000000 8MHz 3/4 NONE QAM16 8k 1/8 NONE
T 682000000 8MHz 3/4 NONE QAM16 8k 1/8 NONE
T 690000000 8MHz 3/4 NONE QAM16 8k 1/8 NONE
T 722000000 8MHz 3/4 NONE QAM16 8k 1/8 NONE
T 738000000 8MHz 3/4 NONE QAM16 8k 1/8 NONE
T 810000000 8MHz 3/4 NONE QAM16 8k 1/8 NONE
T 826000000 8MHz 3/4 NONE QAM16 8k 1/8 NONE
T 850000000 8MHz 3/4 NONE QAM64 8k 1/8 NONE
```

Save and start scanning with the folloing command :

```
scan /usr/share/dvb/dvb-t/gr-Athens -o vdr > /usr/local/vdr/etc/vdr/channels.conf
```

When the above procedure is finished, a new channels.conf file will be created. After that, start the service...

```
service runvdr start
```

Open firefox ( you need vlc plugin installed ), go to  http://serverip:3000 and select channels. If everything goes as expected you will see a channel list.

Good luck
