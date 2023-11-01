---
title: "Asterisk 1.4.19 and Asterisk-addons 1.6.0-beta3 Released"
date: "2008-04-02"
categories: 
  - "asterisk"
---

The Asterisk development team has released version 1.4.19 of Asterisk and 1.6.0-beta3 of Asterisk-addons.

The new Asterisk-addons release contains a few bug fixes over the previous version.

[http://svn.digium.com/view/asterisk-addons/tags/1.6.0-beta3/ChangeLog?view=markup](http://svn.digium.com/view/asterisk-addons/tags/1.6.0-beta3/ChangeLog?view=markup)

Asterisk 1.4.19 contains a large number of fixes over the previous release,1.4.18. For a full list of changes, see the ChangeLog that is included in the release.

[http://svn.digium.com/view/asterisk/tags/1.4.19/ChangeLog?view=markup](http://svn.digium.com/view/asterisk/tags/1.4.19/ChangeLog?view=markup)

One change that requires specific attention is a change to iLBC support. Due to problems with the licensing of the iLBC source code, the implementation of the codec has been removed from the Asterisk source tree. To get the codec\_ilbc module to compile, you will have to retrieve the iLBC source code. A script has been provided which does this for you. Simply run the contrib/scripts/get\_ilbc\_source.sh script from the root directory of the Asterisk source tree.

All users of the iLBC source code should review the license agreement and take whatever actions may be necessary to comply with its terms before continuing to use codec\_ilbc with Asterisk.

Thank you for your support!
