---
title: Traffic capture from a mikrotik device to wireshark
date: 2017-06-15 20:24:08.000000000 +03:00
type: post
categories:
- How to
tags:
- Mikrotik
- Wireshark
permalink: "/traffic-capture-from-a-mikrotik-device-to-wireshark/"
---
Today, for troubleshooting purposes, I needed to capture traffic from a Mikrotik wireless access point that I have. Mikrotik devices have a build-in tool called **Packet sniffer**, which does exactly what I need but what if I had these captures on a remote PC ?

Well we can accomplish this and have the captures on wireshark. All we need is network connectivity, of course, between the Mikrotik device and the PC running wireshark. **I am using wireshark 2.2.7 by the way**.

First we have to connect to the Mikrotik device via winbox and set some parameters to packet sniffer utility in **Tools>Packet Sniffer**. In Streaming tab we check the option Streaming Enabled and we set the IP address of the PC running wireshark. **We hit Apply**.

[![]({{ site.baseurl }}/assets/2017/06/mtik-wireshark1-300x220.png)]

Next, on the **Filter tab**, we set some filters, like the interface we would like to sniff, traffic direction etc. I propose to use filters because if you don't, you might cause high CPU on the mikrotik device. **We hit Apply**.

[![]({{ site.baseurl }}/assets/2017/06/mtik-wireshark2-300x225.png)]

Now if we press the Start button, Mikrotik will send traffic to our server on port 37008. In order to receive only traffic from the Mikrotik device, we need to set up a filter in wireshark telling it to accept UDP traffic only for port 37008.

So lets open wireshark and go to **capture > capture filters**. Then by clicking the " **+**" button, a new line will appear with name New capture filter and an example filter "ip host host.example.com". Set the name to "Mikrotik capture" and the filter to " **udp port 37008**". Press OK.

[![]({{ site.baseurl }}/assets/2017/06/mtik-wireshark3-300x166.png)]

[![]({{ site.baseurl }}/assets/2017/06/mtik-wireshark4-300x166.png)]

Due to protocol conflicts, we have to disable **WCCP** protocol from wireshark. This can be done from **analyze > enabled protocols**. Search for WCCP and uncheck it. Then click OK.

[![]({{ site.baseurl }}/assets/2017/06/mtik-wireshark7-300x190.png)]

On the main screen of wireshark, **click the green flag next to "...using this filter:"** and select the filter that we created earlier. Select your interface and click **capture > start**.

[![]({{ site.baseurl }}/assets/2017/06/mtik-wireshark5-300x163.png)]

Open again, open the Packet filter settings on windox and click start. This will send traffic to your wireshark PC.

Voila !!

[![]({{ site.baseurl }}/assets/2017/06/mtik-wireshark6-300x163.png)]
