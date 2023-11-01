---
title: "Asterisk Jitterbuffer support for Applications"
date: "2007-10-09"
categories: 
  - "asterisk"
tags: 
  - "asterisk-commits"
---

In a [post on asterisk.org](http://asterisk.org/node/48317), I described how the generic jitterbuffer works in Asterisk 1.4. Because of the way it was designed, it does not work when a call is connected to an Asterisk application such as Voicemail or MeetMe. It will only work when two channels are bridged together. So, it has been extremely useful for people who are terminating SIP calls to the PSTN, for example.

Someone at Astricon asked me about this, and it made me think of a way to enable the use of the jitterbuffer for calls connected to Asterisk applications. It was quite simple to implement, too. I added support for the generic jitterbuffer to the Local channel driver, which provides a channel interface back into the local Asterisk dialplan.

Since this is a new feature, it was only added to trunk, our development tree, which will soon become Asterisk 1.6. However, I have posted an unofficial backport of the patch to Asterisk 1.4.

To use the feature, you simply use the new 'j' option in the Dial() string, in addition to the 'n' option that already existed for chan\_local. For example:

`exten => 5551212,1,Dial(Local/1234@somecontext/nj)`

`[somecontext] exten => 1234,1,MeetMe(1234)`

See the commit of the code [here](http://lists.digium.com/pipermail/asterisk-commits/2007-October/016710.html).

Download instructions for Asterisk 1.4: `$ svn co http://svncommunity.digium.com/svn/russell/asterisk-1.4 1.4-backports` There is a file in there called chan\_local\_jitterbuffer.patch.txt. To apply the patch: `$ cd asterisk-1.4 $ patch -p0 < /path/to/chan_local_jitterbuffer.patch.txt. $ make $ sudo make install`

Enjoy!
