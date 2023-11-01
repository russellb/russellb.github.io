---
title: "chan_iax2 stability and performance improvements"
date: "2007-08-23"
categories: 
  - "asterisk"
---

Today, I merged a large set of changes to the IAX2 channel driver in Asterisk into the 1.4 branch and trunk. These changes address both stability and performance issues. Normally, changes such as performance enhancements are not merged into the release branch. However, they just happened to be a side benefit of the work which resolved ways that the module could crash Asterisk.

[View Changes](http://svn.digium.com/view/asterisk?view=rev&revision=80362)

The issue was related to the handling of IAX2 peers and users in memory. It was possible for these objects to get destroyed while they were still in use. The issue could be triggered by reloading the configuration. Using realtime configuration made the problem much worse, as using realtime causes these objects to come and go as they are needed.

The important thing here is that the handling of these objects now uses a common technique called [reference counting](http://en.wikipedia.org/wiki/Reference_counting). Now, it is no longer possible for the objects to disappear while there are still any references to them that could be used.

The changes to the handling of these objects were done by coverting chan\_iax2 to use a new object model called [astobj2](http://svn.digium.com/view/asterisk/trunk/include/asterisk/astobj2.h?view=markup). This object model makes reference counting objects very easy. The use of astobj2 is what brings in the side benefit of performance improvements.

The astobj2 API provides handling of objects, containers, and iterators. The containers that it supports today are a hash table, and a single linked list. The users and peers are now stored in a hash table instead of what it used previously, which was a single linked lists. This makes looking up peers and users **much** more efficient. In a quick test, I hammered my machine with 10 to 20 thousand IAX2 registrations while listening to music on hold on a phone and didn't hear a blip in the audio!

Another performance improvement that the module gets from using astobj2 is how iterating the container is done. Previously, code that needed to look at all of the users or peers had to lock the container during the entire process. This is no longer necessary. The container lock is only held for a short time while getting the next item in the iteration.

I'd like to get some more solid numbers on the performance improvements of using the hash table in this module, but that will have to wait for another day.
