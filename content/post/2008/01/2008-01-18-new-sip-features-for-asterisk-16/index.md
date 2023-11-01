---
title: "New SIP features for Asterisk 1.6"
date: "2008-01-18"
categories: 
  - "asterisk"
tags: 
  - "asterisk-commits"
---

In the past week, I have merged patches into Asterisk trunk that provide new features for chan\_sip. These features will be available in Asterisk 1.6.

**TCP and TLS support**

[The Commit](http://lists.digium.com/pipermail/asterisk-commits/2008-January/019255.html)

In the past, Asterisk has only had support for UDP as a transport for SIP signaling. Asterisk 1.6 will have support for both TCP and TLS, as well.

This work was done by Brett Bryant and James Golovich, who were both funded by Digium for their work.

**SIP Session Timers (RFC 4028)**

[The Commit](http://lists.digium.com/pipermail/asterisk-commits/2008-January/019192.html)

Asterisk 1.6 will also have support for SIP Session Timers, as defined by RFC 4028. These changes prevent stuck SIP sessions that were not properly torn down due to network or endpoint failures during an established SIP session.

This work was done by Raj Jain. It was funded by John Todd from TalkPlus, Inc., as well as JR Richardson of Ntegrated Solutions.

Both of these features are major steps forward for SIP support in Asterisk!
