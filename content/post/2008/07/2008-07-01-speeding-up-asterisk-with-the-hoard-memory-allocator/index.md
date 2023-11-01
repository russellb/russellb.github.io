---
title: "Speeding up Asterisk with the Hoard Memory Allocator"
date: "2008-07-01"
categories: 
  - "asterisk"
tags: 
  - "asterisk-dev"
  - "hoard"
  - "performance"
---

A little over a year I ago, [I made a post](http://lists.digium.com/pipermail/asterisk-dev/2007-June/027937.html) to the Asterisk-dev mailing list looking for someone to do some load testing with the [Hoard Memory Allocator](http://prisms.cs.umass.edu/emery/index.php?page=hoard).

Chris Tooley just recently [gave it a test run](http://www.simplified.org/trac/blog/ctooley-2008/07/01) and saw a boost of about 10% in the number of call setups per second that his machine could handle.

I would encourage others to give this a try when load testing your system. There is virtually no setup other than installing hoard. There are some Asterisk functionality that I feel would see much greater performance improvements than have been demonstrated so far.
