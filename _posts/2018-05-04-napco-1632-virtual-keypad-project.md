---
title: Napco 1632 virtual keypad project
date: 2018-05-04 20:04:46.000000000 +03:00
type: post
categories:
- Projects
tags:
- Napco
permalink: "/napco-1632-virtual-keypad-project/"
---
**UPDATE 11/12/2021 : Colin, a blog reader, contributed to the project with a final solution which can be found [here](https://github.com/cborrowman/Napco1632ArduinoMonitor).**

**UPDATE 12/11/2021 : Apostolos, a blog reader, contributed to the project with the following design, on how to connect ESP to the panel bus. At the moment, I am not able to test it, but Apostolos confirms that it works. Please use with caution**

**![]({{ site.baseurl }}/assets/2018/05/napco-esp-design.jpg)**

**UPDATE 29/8/2020 : I was very pleased to see, that other people also share their time and their knowledge to help others. I did abandoned this project until Matt made a comment on this post with a great solution, which I intend to use soon. All credits to Matt for his brilliant solution!!**

**UPDATE 29/7/2018 :****This will be the first unfinished project that is posted on this blog. After spending three months, I got some info that the version of the panel I have,  does not support automation.**

**Another thing that lead me to abandon this project, is that NAPCO does not provide any info on this panel like DSC etc. so it is very difficult to reverse engineer.**

**I will leave this port active though, for someone who will need the info posted here.**

I am currently working on a project regarding some hack on my alarm panel. It's a napco 1632. I am trying to reverse engineer communications in order to build a webapp to control the panel remotely. There are some options provided by napco, but need separate hardware.

My system is equipped with NL-MOD-UL module that gives you the ability to configure the alarm panel through LAN. This module is connected on the serial port of the panel. The panel has the ability to send real-time status messages through the serial port and receive commands like system arm, but napco hasn't developed any application to do that. They have only quickloader for configuration and management of the panel.

Newest modules like STARLINK or IBR-ZREMOTE give the ability to control the panel remotely, but the hardware is expensive. So, as I said,  I am trying to reverse engineer for a solution. The newest modules are not connected on the serial  port of the panel but on the keypad bus. I have already ordered a logic analyzer in order to decode the bus. But until I receive my logic analyzer, I did some attempts to read it via an arduino MEGA. I made a circuit with an optocoupler because the voltage is to high for the arduino to handle.

![]({{ site.baseurl }}/assets/2018/05/arduino-ttl.jpg)

Here is the arduino code : [ttl-debug]

The bus of napco 1632 has four cables :

- **RED** ( +12V )
- **BLACK** ( GND )
- **YELLOW** ( Keypad TX - Panel RX )
- **GREEN** ( Panel TX - Keypad RX )

I was able to read the data that the panel transmits to the keypad. I believe that the bus uses something among TTL or UART. The speed of the bus is :

- Baudrate : 5208
- Data bits : 8
- Parity : Even
- Stop bits : 1

But I will verify it again with the logic analyzer.

The panel sends all the information displayed on the keypad. My keypad is a DXRP1 which has an LCD of two lines and 16 characters per line. The messages from the panel are sent per line in HEX :

313B0000012000000000**31****36****2D****45****4E****54****52****41****4E****43****45****20****20****20****20****20****11**16-ENTRANCE313B0000016000000000**20****20****20****20****20****20****20****20****20****20****20****20****20****20****20****20****CD**BLANK LINE

From the above, green HEX is the info displayed in the line. The last red byte is the checksum of the message. CheckSum8 Modulo 256 is used for checksum. The second 27 bytes indicate a blank line. Another info when the system is ready to arm. Information is displayed on both lines of the keypad :

313B0000012000000200**53****59****53****54****45****4D****20****52****45****41****44****59****20****20****20****20****89**SYSTEMREADY313B0000016000000200**31****35****2D****30****34****2D****31****38****20****30****38****3A****32****38****50****4D****25**15-04-1808:28PM

Also messages for K series keypads like K2AS is transmitted. This keypad has LCD of one line that displays 6 characters  :

31310000012300000100**46****41****55****4C****54****20****23**FAULT

Again the last byte is the message checksum.

Periodically the panel sends keep alive messages to the keypad. These messages are 13 bytes long. I programmed a second keypad just to verify. These messages are sent almost every 100ms :

**Keypad 1:**

**21**4D00000101400000900000**40**

**Keypad 2:**

**22**4D00000101400000900000**41**

In blue is the ID of the keypad. Keypad 1 is HEX 21, keypad 2 is HEX 22 and so on. The last byte is the message checksum.

The keypad in idle, always transmits the following 4 byte message to the panel. This message is transmitted to the panel 10ms after the above 13 byte keep alive message is received by the keypad :

**Keypad 1:**

**21**4401**66**

**Keypad 2:**

**22**44**01****67**

Again the last byte is the message checksum.

There are also some messages that send date and time on the keypad but I was not able to decode them. They might update the clock on the keypads. They are sent from the panel enery 3 seconds. These messages are also 27 bytes long.

MonthDayYearHourMinutesCRC313B000604151808280079060003000000000000000000000000AC313B0006041518082800090600030000000000000000000000001C313B000604151808280079060003000000000000000000000000AC313B0006041518082800090600030000000000000000000000001C313B000604151808280079060003000000000000000000000000AC313B0006041518082800170600030000000000000000000000000E313B0006041518082800090600030000000000000000000000001C313B000604151808280079060003000000000000000000000000AC313B0006041518082900090600030000000000000000000000001B313B000604151808290079060003000000000000000000000000AB

So if any of you reading this blog, have info on the above or tried something similar,  please get in touch. **I will update this post with my newest findings**.

Wizz...

**Update 10/05/2018 :** I just received my logic analyzer and I did a test. Keep alive messages are sent by the panel to the keypad between 80ms and 100ms. The keypad responses 10ms after the message received.

**Update 23/6/2018 :** I have managed to pull the keypad display to my PC, via a python3 script. With the help of a TTL to RS232 conveter, I connected the bus cables (RX and ground) from the optocoupler to the converter and with a RS232 cable to the serial port of the PC. I think I will go with a raspberry PI on this. Here is the python script :

```
import serial

ser = serial.Serial('/dev/ttyS1', baudrate=5208, bytesize=8, parity=serial.PARITY_EVEN, stopbits=1, timeout=1)
while True:
    reading = ser.read(1)
    if reading == b'\x31':
        reading = ser.read(3)
        if reading == b'\x3b\x00\x00':
            reading = ser.read(22)
            if (reading[1:2]) == b'\x20':
                print("1: " + reading[6:].decode('utf-8'))
            else:
                print("2: " + reading[6:].decode('utf-8'))
                print("-------------------")
```
