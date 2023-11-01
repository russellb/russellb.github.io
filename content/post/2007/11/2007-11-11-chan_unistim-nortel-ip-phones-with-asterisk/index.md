---
title: "chan_unistim: Nortel IP Phones with Asterisk"
date: "2007-11-11"
categories: 
  - "asterisk"
tags: 
  - "asterisk-commits"
  - "voip"
---

A new channel driver has been committed to Asterisk trunk which allows you to use Nortel IP phones with Asterisk. It was submitted by Cedric Hans. See the [commit](http://svn.digium.com/view/asterisk?view=rev&revision=88368) and the original [mantis issue](http://bugs.digium.com/view.php?id=8864). This module will be available in Asterisk 1.6.

There is some [basic documentation](http://svn.digium.com/view/asterisk/trunk/doc/unistim.txt?view=markup) and a [sample configuration file](http://svn.digium.com/view/asterisk/trunk/configs/unistim.conf.sample?view=markup) available.

Nortel phones which have been verified to work with this module are the Nortel i2002, i2004 and i2050.

One of the biggest problems with going with a proprietary phone system is vendor lock-in across all aspects of the system. The amazing thing about Asterisk is the amount of choice that is available. Making it possible to use these handsets with Asterisk will allow those that have previously been locked in to switch to a more powerful and more flexible system without having to invest in new handsets.

That brings the number of VoIP protocols natively supported in Asterisk up to 7. There are:

- SIP
- IAX2
- H.323
- MGCP
- Cisco Skinny / SCCP
- Jingle / GoogleTalk
- UNISTIM

What's next?
