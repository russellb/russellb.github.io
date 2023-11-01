---
title: "Asterisk-commits: Inband DTMF Detector Improvements"
date: "2007-08-25"
categories: 
  - "asterisk"
tags: 
  - "asterisk-commits"
---

We got a very interesting patch to improve the reliability of the Inband DTMF detector. Some poor debouncing logic was making it very possible to miss a digit when it didn't start very clean.

Here is the [commit](http://lists.digium.com/pipermail/asterisk-commits/2007-August/015753.html) and also [the related discussion](http://lists.digium.com/pipermail/asterisk-dev/2007-August/029149.html) on the asterisk-dev mailing list.

Many thanks to Tony Mountifield for the patch!
