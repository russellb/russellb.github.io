---
title: "(Critical Updates) Asterisk 1.2.27, 1.4.18.1, 1.4.19-rc3, 1.6.0-beta6 Released"
date: "2008-03-19"
categories: 
  - "asterisk"
---

The Asterisk.org development team has released four new versions of Asterisk to address critical security vulnerabilities.

AST-2008-002 details two buffer overflows that were discovered in RTP codec payload type handling.

- [http://downloads.digium.com/pub/security/AST-2008-002.pdf](http://downloads.digium.com/pub/security/AST-2008-002.pdf)
- All users of SIP in Asterisk 1.4 and 1.6 are affected.

AST-2008-003 details a vulnerability which allows an attacker to bypass SIP authentication and to make a call into the context specified in the general section of sip.conf.

- [http://downloads.digium.com/pub/security/AST-2008-003.pdf](http://downloads.digium.com/pub/security/AST-2008-003.pdf)
- All users of SIP in Asterisk 1.0, 1.2, 1.4, or 1.6 are affected.

AST-2008-004 details some format string vulnerabilities that were found in the code handling the Asterisk logger and the Asterisk manager interface.

- [http://downloads.digium.com/pub/security/AST-2008-004.pdf](http://downloads.digium.com/pub/security/AST-2008-004.pdf)
- All users of Asterisk 1.6 are affected.

Asterisk 1.2.27 and 1.4.18.1 are releases that only contain changes to fix these security vulnerabilities.

In addition to fixes for these security issues, 1.4.19-rc3 and 1.6.0-beta6 contain a number of other bug fixes over the previous release candidates and beta releases for the upcoming 1.4.19 and 1.6.0 releases.

We encourage all affected users of these security vulnerabilities to upgrade their installations as time permits.

Thank you for your continued support of Asterisk!
