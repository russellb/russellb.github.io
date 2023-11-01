---
title: "Asterisk 1.6, Now with Distributed Presence"
date: "2008-06-10"
categories: 
  - "asterisk"
tags: 
  - "asterisk-commits"
---

Have you ever wanted presence information across multiple Asterisk servers? Well, making that possible is something that I have worked on a bit here and there. I previously made [a post asking for some of that code to be tested](http://www.russellbryant.net/blog/index.php/2008/02/18/request-for-testing-distributed-device-state/). Now, this code has been merged in to Asterisk 1.6.

Asterisk 1.6.1 will have the ability to share device state between servers using a new module, res\_ais, which uses the [SAForum AIS](http://www.saforum.org/specification/AIS_Information/) to share events between Asterisk servers. It has been tested with and developed against the [openais](http://www.openais.org) implementation of AIS.

In addition to writing res\_ais, this effort consisted of a lot of improvements to the Asterisk core to understand distributed device state. My next step is to complete the changes to pbx\_dundi to allow Asterisk servers to use DUNDi to share events, as well. This should provide the best results for servers that want to share state information across the Internet, as opposed to servers in a local cluster on a high speed LAN.

Commits related to merging this code into Asterisk 1.6:

[http://lists.digium.com/pipermail/asterisk-commits/2008-June/023363.html](http://lists.digium.com/pipermail/asterisk-commits/2008-June/023363.html) [http://lists.digium.com/pipermail/asterisk-commits/2008-June/023371.html](http://lists.digium.com/pipermail/asterisk-commits/2008-June/023371.html) [http://lists.digium.com/pipermail/asterisk-commits/2008-June/023387.html](http://lists.digium.com/pipermail/asterisk-commits/2008-June/023387.html) [http://lists.digium.com/pipermail/asterisk-commits/2008-June/023396.html](http://lists.digium.com/pipermail/asterisk-commits/2008-June/023396.html) [http://lists.digium.com/pipermail/asterisk-commits/2008-June/023400.html](http://lists.digium.com/pipermail/asterisk-commits/2008-June/023400.html)
