---
title: Custom images on cisco 79XX telephone sets
date: 2016-06-30 17:53:56.000000000 +03:00
type: post
categories:
- How to
tags:
- Cisco
permalink: "/custom-images-on-cisco-79xx-telephone-sets/"
---
To be able to add images on devices, you must already have a TFTP-Server and of course **the necessary .xml configuration files.**

Every cisco telephone set has it's own demands. The image format must be on .PNG with specific dimensions

**Telephone set****Image dimensions (pixels)****Thumbnail dimensions(pixels)****TFTP Folder**7906/791195 X 3423 X 8/Desktops/95x34x17941/7961/7942/7962320 X 19680 X 49/Desktops/320x196x47945/7965/7975320 X 21280 X 53/Desktops/320x212x167970/7971320 X 21280 X 53/Desktops/320x212x12

Every image folder on the TFTP-Server must contail the file List.xml (e.g. /Desktops/95x34x1/List.xml) with the following contents :

```
<CiscoIPPhoneImageList>
<ImageItem Image="TFTP:Desktops/320x212x12/TN-CampusNight.png"
URL="TFTP:Desktops/320x212x12/CampusNight.png"/>
</CiscoIPPhoneImageList>
```

For every custom image we add two lines among CiscoIPPhoneImageList tags. First goes the thumbnail (after Image=) and then the complete image (after URL=).

If everythig go as expected in menu User Preferences > Background Images you will find your custom images.
