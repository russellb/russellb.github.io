---
title: "Astricon, Code Zone Adventures"
date: "2008-09-10"
categories: 
  - "asterisk"
  - "astricon"
---

[Allison Smith](http://www.digium.com/en/products/ivr/allisonsmith/) introduces us to the [Astricon Code Zone](http://astricon.net/2008/glendale/web/CodeZone.php):

\[audio:tt-codezone\_MIXDOWN.mp3\]

I am feeling especially motivated this year to make good use of the Astricon Code Zone. I seem to do well at working on development projects while I travel. Part of it is probably breaking out of my normal routine. Some of it is because I am stuck on an airplane for hours with nothing to distract me. This time, most of it is because I will be immersed in a very high concentration of Asterisk geeks for a week!

I have been trying to get some thoughts together about what things I would like to work on while I'm there. Here are some things on my mind.

- **Video on Hold** - Someone on the [asterisk-users mailing list](http://lists.digium.com/pipermail/asterisk-users/2008-September/218025.html) asked if this was supported in Asterisk. It currently is not. However, it occurred to me that it wouldn't be that hard to add. I have already written a rough draft. I plan to get it tested and finished up in the code zone.
- **SIP Performance** - One thing that could be done to improve the performance in chan\_sip is to make processing multi-threaded. I have stage 1 of this process completed, which is to run the SIP scheduler in its own thread. I would like to load test this code some more, as well as multi-thread packet processing. I have learned a \_lot\_ from the long and painful process of making chan\_iax2 multi-threaded, so I think this will go over much smoother. Early load testing of the scheduler optimization on a multi-core machine by [Joshua Colp](http://www.joshua-colp.com) showed a performance improvement of 300%!!! I was blown away. Hopefully we can put together some more details about these test cases soon. These results give me a lot of motivation to move forward with completing this work.
- **Distributed Call Queues** - We have been talking about this one for years. It keeps coming back up. Some of the needed prerequisites, like [distributed device state](http://www.russellbryant.net/blog/2008/06/10/asterisk-16-now-with-distributed-presence/), are now in place. I have talked to a number of people about what this functionality needs to look like, and I think we're ready to start developing the framework for it. Lately, Josh has been expressing interest in this, as well. So, we will likely work on this together on the trip.

If anyone has thoughts on things they would like to work on or talk about in the code zone, please share them!

I'll see you at Astricon.
