---
title: "Request for Testing: Distributed Device State"
date: "2008-02-19"
categories: 
  - "asterisk"
---

Greetings,

One of the things that I have been working with off and on is a new event API for Asterisk. Most of the current infrastructure was written almost a year ago. However, just recently, I extended this system to be able to allow device states to be shared within a cluster of Asterisk servers.

The current system for distributed events is designed for a high speed LAN. However, I also have plans to extend this to DUNDi so that it can be used withina DUNDi network, as well.

If you are interested in this work, I would appreciate any help with testing. I have documented how to set it up and how I used some custom states to verify the functionality.

To download the code: `$ svn co http://svn.digium.com/svn/asterisk/team/russell/events asterisk-events`

For information on setup and testing, check out [this document](http://svn.digium.com/view/asterisk/team/russell/events/doc/distributed_devstate.txt?view=markup).

**UPDATE**

This functionality has been merged into Asterisk and will be available in Asterisk 1.6.1. Instead of testing this branch, please test 1.6.1.

To download the code: `$ svn co http://svn.digium.com/svn/asterisk/branches/1.6.1 asterisk-1.6.1`

For information on setup and testing, check out [this document](http://svn.digium.com/view/asterisk/branches/1.6.1/doc/distributed_devstate.txt?view=markup).
