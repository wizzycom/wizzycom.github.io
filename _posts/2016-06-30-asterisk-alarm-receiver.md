---
title: Asterisk alarm receiver
date: 2016-06-30 17:53:29.000000000 +03:00
type: post
categories:
- How to
tags:
- Asterisk
permalink: "/asterisk-alarm-receiver/"
---
**WARNING : This solution is not the best way to protect your property.**

First of all, I am not a programmer so many of you will notice my elementary skills on coding. If you have anny issues or suggestions please contact me at info\[at\]wizzycom\[dot\]net

Some information about Ademco Contact ID.

Ademco Contact ID is a protocol that establishes communication between a a security system and a monitoring station . The security system sends a 16-digit code to the monitoring center and the monitoring station converts this to readable information.

Let's see the sections of this 16-digit code :

**1111** 22 **3** 444 **55** 666 **7**

The monitoring station receives the code above ( not a real example, but it is easier to understand the sections ) :

**Section 1**. Account number. This is the account ID of the security system. This info is set by the installer.

**Section 2**. Message type. The value of this is 18 (preffered) or 98 (optional).

**Section 3**. Event qualifier. Three values => 1 ( Νew event ) 3 ( Restored event ) 6 ( Previous reported - still present ).

**Section 4**. Event Code. This describes the issue ( Value 401 is arm/disarm from user ).

**Section 5**. Partition Number. Many security systems have the ability of virtual security systems. This means that you can have two or more individual security systems in one physical system.

**Section 6**. Zone Number. This is the zone number.

**Section 7**. Checksum. This is just a value which confirms that the 16-digit code is correct.

Asterisk PBX has the ability to receive those 16-digits events and by the use of a script you are able to convert is to readable information through alarmreceiver module . How it works :

- The security system sends an event through PSTN
- Asterisk receives this event and executes a script
- The script converts the event and sends an email.

Now for this guide, I have a security system connected to a Linksys SPA. I had some DTMF issues but the solved after setting the following on the SPA :

**DTMF Tx Method = AVT**

**DTMF Tx Mode = Normal**

First we have to configure the asterisk module. So we edit the file /etc/asterisk/alarmreceiver.conf and set the following :

```
[general]

timestampformat = %Y-%m-%d %H:%M:%S
eventcmd = /etc/asterisk/alarm/scripts/alarm-getevent
eventspooldir = /etc/asterisk/alarm/pending
logindividualevents = yes
fdtimeout = 4000
sdtimeout = 4000
loudness = 600
```

Then we have to create the folder structure

```
mkdir /etc/asterisk/alarm
mkdir /etc/asterisk/alarm/pending
mkdir /etc/asterisk/alarm/reported
mkdir /etc/asterisk/alarm/cache
mkdir /etc/asterisk/alarm/scripts
```

Create our MySQL database :

```
mysql -u root -p
create database security_system DEFAULT CHARACTER SET utf8 COLLATE utf8_general_ci;
```

Import [this file](/wp-content/uploads/2016/06/alarm.txt) on the newly created database.

Now we have two scripts. One that is executed every time a event is received ( alarm-getevent

) and another one that is executed through cron every minute ( alarm-process ).

**/etc/asterisk/alarm/scripts/alarm-getevent**

```
#!/bin/sh
#
dbase='mysql -uUSER -pPASSWORD -hHOST security_system'

for i in `find /etc/asterisk/alarm/pending -type f`
do

chmod 777 ${i}
alarmevent=`cat ${i} | sed "1,/events/d" | grep -v "^$"`
alarmtime=`grep TIMESTAMP ${i} | cut -f2 -d=`

now=`date +'%Y/%m/%d %H:%M:%S'`

account=`echo $alarmevent | cut -b1-4`
messagetype=`echo $alarmevent | cut -b5-6`
qualifier=`echo $alarmevent | cut -b7`
eventcode=`echo $alarmevent | cut -b8-10`
grp=`echo $alarmevent | cut -b11-12`
zone=`echo $alarmevent | cut -b13-15`
chksum=`echo $alarmevent | cut -b16`

dtreported=`date --date "$alarmtime" +%s`
dtregistered=`date --date "$now" +%s`

$dbase -vv -e "insert into alarm_events (alarmevent,account,messagetype,qualifier,eventcode,grp,zone,chksum,dtreported,dtregistered) values ('$alarmevent','$account','$messagetype','$qualifier','$eventcode','$grp','$zone','$chksum','$dtreported','$dtregistered')"

mv ${i} /etc/asterisk/alarm/reported

done
```

Please change **USER**, **PASSWORD**, **HOST** with the credentials needed to access your database.

**/etc/asterisk/alarm/scripts/alarm-process**

```
#!/bin/sh
#
dbase='mysql -uUSER -pPASSWORD -hHOST security_system'

recordcount=`$dbase -B --skip-column-names -e "select * from alarm_events where status = '1' order by dtreported limit 1;"`

if [ -n "$recordcount" ]; then

id=`echo $recordcount | cut -f1 -d' ' `
alarmevent=`echo $recordcount | cut -f2 -d' ' `
account=`echo $recordcount | cut -f3 -d' ' `
messagetype=`echo $recordcount | cut -f4 -d' ' `
qualifier=`echo $recordcount | cut -f5 -d' ' `
eventcode=`echo $recordcount | cut -f6 -d' ' `
grp=`echo $recordcount | cut -f7 -d' ' `
zone=`echo $recordcount | cut -f8 -d' ' `

if [ $eventcode = 401 ]
then
zonedescription=`$dbase -B --skip-column-names -e "select system_users.user_description from system_users where user_id = $zone"`
else
zonedescription=`$dbase -B --skip-column-names -e "select system_areas.area_description from system_areas where area_id = $zone"`
fi

zone=`echo $recordcount | cut -f8 -d' ' `
chksum=`echo $recordcount | cut -f9 -d' ' `
dtregistered=`echo $recordcount | cut -f10 -d' ' `
dtreported=`echo $recordcount | cut -f11 -d' ' `

eventdescription=`$dbase -B --skip-column-names -e "select event_types.call,event_types.sms,event_types.description from event_types where eventcode = $eventcode"`

callaction=`echo $eventdescription | cut -f1 -d' ' `
smsaction=`echo $eventdescription | cut -f2 -d' ' `
description=`echo $eventdescription | cut -f3-30 -d' ' `

if [ $qualifier = 1 ]
then
eventqualifier=New
fi

if [ $qualifier = 3 ]
then
eventqualifier=Restored
fi

if [ $qualifier = 6 ]
then
eventqualifier=Still active
fi

realdatereported=`date -d @$dtreported "+%d/%m/%Y %T"`
realdateregistered=`date -d @$dtregistered "+%d/%m/%Y %T"`

#MAIL

extension=$RANDOM

mailfile=/etc/asterisk/alarm/cache/event_$id.$extension

echo "Hello owner" >> $mailfile
echo "" >> $mailfile
echo "" >> $mailfile
echo "An event has occured on the security system. Please read the event description below." >> $mailfile
echo "" >> $mailfile
echo "" >> $mailfile
echo "===================================================" >> $mailfile
echo " Alarm Notification " >> $mailfile
echo "===================================================" >> $mailfile
echo "" >> $mailfile
echo Reported at $realdatereported >> $mailfile
echo Registration at $realdateregistered >> $mailfile
echo "" >> $mailfile
echo ID $id >> $mailfile
echo Event qualifyer $eventqualifier >> $mailfile
echo Partition/Group $grp >> $mailfile
echo Zone $zonedescription >> $mailfile
echo Event description $description >> $mailfile

/bin/mailx -s "Alarm event - $description - $realdatereported" -r alarm@example.com -S smtp="mail.example.com" user@example.com < $mailfile

updatestatus=`$dbase -e "update alarm_events set status = '0' where id = $id"`

fi
```

Please change **USER**, **PASSWORD**, **HOST** with the credentials needed to access your database. Also make the appropriate email changes.

Give permisions

```
chown asterisk:asterisk /etc/asterisk/alarm -R
chmod +x /etc/asterisk/alarm/scripts/alarm-getevent
chmod +x /etc/asterisk/alarm/scripts/alarm-process
```

Set the following line on cron via crontab -e

```
*/1 * * * * /etc/asterisk/alarm/scripts/alarm-process &> /dev/null 2>&1
```

Restart cron

```
systemctl restart crond
```

Trigger an event from the security system

Good luck
