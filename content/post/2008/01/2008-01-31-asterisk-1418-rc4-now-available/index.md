---
title: "Asterisk 1.4.18-rc4 Now Available"
date: "2008-01-31"
categories: 
  - "asterisk"
---

Asterisk 1.4.18-rc4 is now available.

This release candidate includes an important fix for a regression related to the use of codec\_g729 that caused decoders to not get properly released. Additional fixes added today that are included in this release candidate include:

- fixes for some locking errors in chan\_agent
- a memory leak related to the use of AMI redirect
- Solaris compatibility fixes
- a fix related to call recordings from Monitor getting deleted before being mixed if a blind transfer is done from a Queue.

Thanks to everyone that has jumped on to help out with testing of release candidates! It has already been extremely helpful.

This release candidate is published for anyone that is interested in helping to test it for a couple of days before it is officially released. To download the release candidate, use the following svn command: `$ svn co http://svn.digium.com/svn/asterisk/tags/1.4.18 asterisk-1.4.18-rc4` If you would like it in tarball format, use the following commands: `$ svn export http://svn.digium.com/svn/asterisk/tags/1.4.18 asterisk-1.4.18-rc4 $ tar -czvf asterisk-1.4.18-rc4.tar.gz asterisk-1.4.18-rc4/` Thanks!
