---
title: "Asterisk 1.2.28, 1.4.19.1, and 1.6.0-beta8 Released"
date: "2008-04-22"
categories: 
  - "asterisk"
---

The Asterisk development team has released versions 1.2.28, 1.4.19.1, and 1.6.0-beta8.

All of these releases contain a security patch for the vulnerability described in the AST-2008-006 security advisory. 1.6.0-beta8 is also a regular update to the 1.6.0 series with a number of bug fixes over the previous beta release.

Early last year, we made some modifications to the IAX2 channel driver to combat potential usage of IAX2 in traffic amplification attacks. Unfortunately, our fix was not complete and we were not notified of this until the original reporter of the issue decided to release information on how to exploit it to the public.

This issue affects all users of IAX2 that have allowed non-authenticated calls. For more information on the vulnerability, see the published security advisory.

- [http://downloads.digium.com/pub/security/AST-2008-006.pdf](http://downloads.digium.com/pub/security/AST-2008-006.pdf)

All releases are available for download from the following location:

- [http://downloads.digium.com/pub/telephony/asterisk/](http://downloads.digium.com/pub/telephony/asterisk/)

Thank you for your continued support of Asterisk!
