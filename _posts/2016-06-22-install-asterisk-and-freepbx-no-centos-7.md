---
title: Install Asterisk and FreePBX no CentOS 7
date: 2016-06-22 17:50:49.000000000 +03:00
type: post
categories:
- How to
tags:
- Asterisk
- CentOS
- FreePBX
- Linux
permalink: "/install-asterisk-and-freepbx-no-centos-7/"
---
We must have a fresh installation of CentOS 7 with internet access. We begin by disabling selinux because problems were observed during the installation with selinux enabled.

```
sed -i 's/\(^SELINUX=\).*/\SELINUX=disabled/' /etc/sysconfig/selinux
sed -i 's/\(^SELINUX=\).*/\SELINUX=disabled/' /etc/selinux/config
```

We will install the development tools because until now there are no binaries for asterisk and CentOS 7 :

```
yum -y groupinstall "Development Tools"
```

Then we will install some necessary packages :

```
yum -y install lynx mariadb-server mariadb php php-mysql php-mbstring tftp-server \
httpd ncurses-devel sendmail sendmail-cf sox newt-devel libxml2-devel libtiff-devel \
audiofile-devel gtk2-devel subversion kernel-devel git php-process crontabs cronie \
cronie-anacron wget vim php-xml uuid-devel sqlite-devel net-tools gnutls-devel php-pear \
unixODBC mysql-connector-odbc iksemel iksemel-devel jansson jansson-devel pjproject \
pjproject-devel libsamplerate-devel gmime-devel
```

```
pear install Console_Getopt
```

We activate our database and start the installation wizard to define root password and remote access . What you will choose on the wizard, depends on your needs:

```
systemctl enable mariadb.service
systemctl start mariadb
mysql_secure_installation
```

We activate our web server :

```
systemctl enable httpd.service
systemctl start httpd.service
```

We create the Asterisk user :

```
adduser asterisk -M -c "Asterisk User"
```

Let's download the sources needed :

```
cd /usr/src
wget http://downloads.asterisk.org/pub/telephony/asterisk/asterisk-13-current.tar.gz
wget http://downloads.asterisk.org/pub/telephony/dahdi-linux-complete/dahdi-linux-complete-current.tar.gz
wget http://mirror.freepbx.org/modules/packages/freepbx/freepbx-13.0-latest.tgz
```

In case that we have FXS/FXO hardware installed or we want to use conference, we should compile and install the Dahdi package :

```
tar xvfz dahdi-linux-complete-current.tar.gz
cd dahdi-linux-complete-*
make all
make install
make config
```

We compile and install the Asterisk package :

```
tar xvfz asterisk-13-current.tar.gz
cd asterisk-*
```

Inside the contrib folder is a script that installs various necessary packages. You will notice that the script cannot find a specific package. Don't worry, it is installed by another name :

```
contrib/scripts/install_prereq install
```

Here we go!

```
./configure --libdir=/usr/lib64
```

If we would like to use MP3 files on Asterisk, we have to download the MP3 sources. There is also a script in contrib folder that does the job :

```
contrib/scripts/get_mp3_source.sh
```

We choose menuselect just to select ULAW sound packages and the format\_mp3 option:

```
make menuselect
```

We press save and start the compile :

```
make
make install
make config
ldconfig
systemctl disable asterisk
```

Let's give some permissions :

```
chown asterisk. /var/run/asterisk
chown -R asterisk. /etc/asterisk
chown -R asterisk. /var/{lib,log,spool}/asterisk
chown -R asterisk. /usr/lib64/asterisk
chown -R asterisk. /var/www/
```

Some changes required by FreePBX on php.ini :

```
sed -i 's/\(^upload_max_filesize = \).*/\120M/' /etc/php.ini
sed -i 's/^\(User\|Group\).*/\1 asterisk/' /etc/httpd/conf/httpd.conf
sed -i 's/AllowOverride None/AllowOverride All/' /etc/httpd/conf/httpd.conf
```

We have to restart the web server for changed to take affect :

```
systemctl restart httpd.service
```

We begin installing FreePBX. FreePBX has setup wizard. It asks paths that will put the scripts, some info for the web server and the database. We just give answers according to our case :

```
tar xfz freepbx-13.0-latest.tgz
cd freepbx
./start_asterisk start
./install
```

We need to create the following startup script. FreePBX does not provide a startup script :

```
nano /etc/systemd/system/freepbx.service
[Unit]
Description=FreePBX VoIP Server
After=mariadb.service

[Service]
Type=oneshot
RemainAfterExit=yes
ExecStart=/usr/sbin/fwconsole start
ExecStop=/usr/sbin/fwconsole stop

[Install]
WantedBy=multi-user.target
```

When everything is done, we just activate the whole thing :

```
systemctl enable freepbx.service
systemctl start freepbx.service
```

Good luck!!
