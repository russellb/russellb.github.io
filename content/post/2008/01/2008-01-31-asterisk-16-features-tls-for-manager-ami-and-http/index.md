---
title: "Asterisk 1.6 Features: TLS for Manager (AMI) and HTTP"
date: "2008-01-31"
categories: 
  - "asterisk"
---

I have pointed out this file before, but I'd like to point it out again. If you're curious what new features have been added for Asterisk 1.6 since 1.4 was released, then check out the CHANGES file. The current version of it can be found [here](http://svn.digium.com/view/asterisk/trunk/CHANGES?view=markup).

There are a lot of cool features in there, so I'll pull some out from time to time to highlight them.

One of the nice features in this version is native TLS support for the Asterisk manager interface as well as the built-in HTTP server. This was developed by one of our outstanding community developers, [Luigi Rizzo](http://info.iet.unipi.it/~luigi/). This means that it is easier to make applications that use these interfaces more secure. For example, users of the AsteriskGUI that is bundled with [AsteriskNOW](http://www.asterisknow.org/) will have an easier time setting up secure access to configure Asterisk from a web browser. Previously, the way to do this was to proxy communication with Asterisk through a web server such as Apache or lighttpd.

Turning on this feature is easy. For the manager interface, only two options are required in the \[general\] section of manager.conf: `sslenable = yes sslcert = /var/lib/asterisk/asterisk.pem`

For the HTTP interface, the exact same two options are required, but for http.conf.

We take security very seriously in the Asterisk project. This has really been demonstrated through our current process for handling security issues that get reported to us. Now, in Asterisk 1.6, with TLS support for SIP signalling, The Asterisk Manager Interface (AMI), as well as the HTTP interface, we're making it even easier to secure communications with your Asterisk server.
