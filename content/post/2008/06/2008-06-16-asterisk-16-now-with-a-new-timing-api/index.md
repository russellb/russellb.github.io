---
title: "Asterisk 1.6, Now with a New Timing API"
date: "2008-06-16"
categories: 
  - "asterisk"
---

For as long as I have been involved with the project, Asterisk has used DAHDI (Zaptel) to provide timing. This has worked great, since the timing was usually provided by telephony hardware. Then, we had the ztdummy kernel module to provide timing from other parts of the kernel.

However, a lot of people have not been happy with this. Most Asterisk users don't have telephony hardware to provide timing. Also, ztdummy does not work on all systems. Also, DAHDI was originally only for Linux.

Starting with Asterisk 1.6.1, you will no longer need DAHDI installed at all to get proper timing in Asterisk. There is a new timing API, and there are already two implementations. There is a DAHDI timing interface (res\_timing\_dahdi) and another (res\_timing\_pthread) that has no special kernel requirements.

Keep in mind, that MeetMe, the Asterisk conferencing application, still requires DAHDI. MeetMe uses the DAHDI conference mixing. It doesn't just use it for timing. However, do not worry! Coming soon, we will have a new bridging API that provides software conference mixing, among many other benefits. The code is pretty much done. It just needs some more documentation polish and a bit of code review. Then, you will not need DAHDI for conferencing either.

[Timing API](http://svn.digium.com/view/asterisk/trunk/include/asterisk/timing.h?view=markup)

[res\_timing\_dahdi](http://svn.digium.com/view/asterisk/trunk/res/res_timing_dahdi.c?view=markup)

[res\_timing\_pthread](http://svn.digium.com/view/asterisk/trunk/res/res_timing_pthread.c?view=markup)

Josh's bridging code: `$ svn co http://svn.digium.com/svn/asterisk/team/file/bridging`
