---
title: "New Channel Driver: chan_console"
date: "2007-12-31"
categories: 
  - "asterisk"
tags: 
  - "asterisk-commits"
  - "asterisk-dev"
---

I just merged a new channel driver into Asterisk trunk which will be in Asterisk 1.6.  The module is called chan\_console.  It is a new console channel driver which uses portaudio as a cross platform audio interface instead of using something like ALSA or OSS directly.  I wrote it to give myself a console channel driver that I could use on my Mac.  However, portaudio supports a number of other audio interfaces, as well.

The audio interfaces that portaudio supports are

- Advanced Linux Sound Architecture (ALSA)
- AudioScience HPI
- Macintosh Core Audio for OS X
- Windows Direct Sound
- JACK Audio Connection Kit
- Unix Open Sound System (OSS)
- Windows Vista WASAPI
- Windows WDM Kernel Streaming
- Windows MME

Asterisk-dev - [Another Module for Testing: chan\_console](http://lists.digium.com/pipermail/asterisk-dev/2007-December/031302.html)

Asterisk-commits - [Commit to Asterisk trunk](http://lists.digium.com/pipermail/asterisk-commits/2007-December/018753.html)
