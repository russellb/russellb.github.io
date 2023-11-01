---
title: "More Work on Asterisk as a Softphone ... Inside Firefox?!"
date: "2008-05-19"
categories: 
  - "asterisk"
tags: 
  - "asterisk-dev"
---

[Luigi Rizzo](http://info.iet.unipi.it/~luigi/), a community developer for the Asterisk project has posted a message to the developers mailing list talking about some of the work that he and his students are working on. Previously, I pointed out how they had done work to allow you to [use Asterisk as a softphone that supports video](http://www.russellbryant.net/blog/index.php/2007/12/18/asterisk-as-a-video-soft-phone/). Now, it sounds like they are taking this even further.

He talks about:

**1) Merge console channel drivers in Asterisk.**

Currently, there are three console channel drivers in Asterisk: chan\_oss, chan\_alsa, and chan\_console. chan\_console uses libportaudio to act as a cross platform channel driver. However, it does not have all of the features of chan\_oss. Once chan\_console has all of the features of the others, chan\_console will be the single console channel driver.

**2) Add support for multiple audio and video inputs for a single call.**

This will allow switching between inputs within a call, as well as implementing interesting audio and video effects such as picture-in-picture or split screen during a call.

**3) Make Asterisk run as a Firefox extension**

This one completely blows my mind.

The goal here is to allow Asterisk to be embedded into Firefox so that the interface for making phone calls can be written within the web browser. It sounds like they already have this working, too! I can't wait to see what this turns in to.

Admittedly, it seems like it could be overkill, but at the same time, Asterisk is an extremely powerful and flexible telephony platform. If the functionality of Asterisk was available as the backend for people that are capable of developing a great UI and user experience, the possibilities are quite interesting.

See the full mailing list post [here](http://lists.digium.com/pipermail/asterisk-dev/2008-May/032977.html).
