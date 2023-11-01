---
title: "IAX2 Performance"
date: "2008-05-05"
categories: 
  - "asterisk"
tags: 
  - "iax2"
---

As a part of the latest Asterisk security release, the IAX2 channel driver in Asterisk got various changes to make it more difficult to abuse IAX2 in Asterisk in a traffic amplification attack. IAX2 uses call numbers to specify which packets are associated with which call. One of the changes that I made for the security issue was to have Asterisk randomly choose call numbers where it previously chose the lowest one that it could.

This change had an unfortunate side effect. It highlighted a part of chan\_iax2 that was extremely inefficient. The higher the call number, the worse this piece of code performed. By choosing a very high call number, the performance of the server would plummet. On a small embedded platform, this may have meant that it couldn't handle a single call. On my Intel Core 2 Duo machine @ 2.33 GHz, this meant that I couldn't handle much more than about 16 IAX2 channels. That is _terrible_.

So, this got me motivated to fix up this code once and for all. I reworked some of the IAX2 code in Asterisk to index calls in a hash table in such a way that the lookup that previously had such terrible performance could be performed very quickly.

Testing my new code against chan\_iax2 with my security fixes yielded a 3600% concurrent IAX2 channel increase! However, even when compared with the code before the security fixes, the IAX2 channel driver is still able to handle a lot more calls. On my machine, the most recent code can handle 55% more concurrent IAX2 channels than the code could handle before the security changes.

Asterisk 1.4.20 will contain these changes. Normally I would not put enhancements like this in the 1.4 release version of Asterisk. However, this was a special situation. So, enjoy!

There has been criticism of IAX2 performance in the past. However, during the 1.4 release series, I have put a lot of work into re-doing a lot of data structure management. Most of the changes were inspired by bugs that came up, but had nice performance impacts as a beneficial side effect.

At this point, I can not see any reason that the IAX2 channel driver in Asterisk would perform any worse than SIP. It would be interesting to do some more IAX2 versus SIP comparisons with the most recent code. In Asterisk trunk, a lot of work has gone into the data structures supporting both protocols. All critical data structures have been carefully converted to be indexed in hash tables using the astobj2 object model. Not only are lookups much faster with the hash tables, but the astobj2 object model has a lot of very nice locking features which provide even more performance improvements.

Anyway, thank you all for your continued support of Asterisk!
