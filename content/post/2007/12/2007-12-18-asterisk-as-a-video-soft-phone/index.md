---
title: "Asterisk as a video soft phone"
date: "2007-12-18"
categories: 
  - "asterisk"
tags: 
  - "asterisk-commits"
---

Asterisk trunk recently got a pretty cool new feature. You can now use Asterisk as a _highly_ configurable video soft phone. The commit to trunk is [here](http://lists.digium.com/pipermail/asterisk-commits/2007-December/018359.html "here").

The way it works is pretty neat. Asterisk already had a couple of console channel drivers: chan\_oss and chan\_alsa. These channel drivers allow you to use a local OSS or ALSA sound device as an endpoint for a call. These interfaces are commonly used to interface with overhead paging systems. They are also commonly used by people to use Asterisk as an extremely powerful soft phone.

Now, Asterisk as a softphone just got **a lot** cooler.

The OSS console channel driver, chan\_oss, now has video support. This means that you can make video calls from the Asterisk CLI. For the video source, you currently have a couple of options. The first is to use a webcam. The second, which I find quite interesting, is you can use an X11 screen grabber. That means you can have a section of your local display that gets captured and sent along as the video stream.

There is also a "skinnable" dialpad for use as a graphical softphone interface for dialing.

The code uses libavcodec from ffmpeg for video transcoding. As the commit message states, it currently supports h261, h263, h263+, h264, and mpeg4.

Many thanks to Luigi Rizzo, Sergio Fadda, and Marta Carbone for the great new feature!
