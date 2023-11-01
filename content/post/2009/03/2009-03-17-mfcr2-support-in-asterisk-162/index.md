---
title: "MFC/R2 support in Asterisk 1.6.2"
date: "2009-03-17"
categories: 
  - "asterisk"
---

Yesterday, official support for MFC/R2 signaling in chan\_dahdi was merged, and will be included in Asterisk 1.6.2. Here is [the commit](http://lists.digium.com/pipermail/asterisk-commits/2009-March/031735.html).

To get a head start on testing Asterisk 1.6.2 with R2 support, check out the code from the 1.6.2 branch: `$ svn co http://svn.digium.com/svn/asterisk/branches/1.6.2 asterisk-1.6.2`

Before compiling Asterisk, you will need to compile and install [LibOpenR2](http://www.libopenr2.org).

Thank you very much to Moises Silva (Moy), who worked very hard to write LibOpenR2 and bring this functionality to Asterisk!
