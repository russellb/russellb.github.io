---
title: "New Versions of Asterisk and DAHDI"
date: "2008-09-08"
categories: 
  - "asterisk"
  - "dahdi"
---

The Asterisk development team is pleased to announce new releases of Asterisk and DAHDI.

For more information on the reasoning behind the transition to DAHDI, please see the following post:

[http://blogs.digium.com/2008/05/19/zaptel-project-being-renamed-to-dahdi/](http://blogs.digium.com/2008/05/19/zaptel-project-being-renamed-to-dahdi/)

The list of packages released today includes:

- dahdi-linux 2.0.0-rc4
- dahdi-linux-complete 2.0.0-rc4+2.0.0-rc2
- asterisk 1.4.22-rc4
- asterisk 1.6.0-rc5

All of these packages are available on [http://downloads.digium.com](http://downloads.digium.com) in their respective directories. Detailed information about each package release is included below.

\=== dahdi-linux-complete-2.0.0-rc3+2.0.0-rc2 ===

This release combines dahdi-linux-2.0.0-rc4 and dahdi-tools-2.0.0-rc2 into a single download, one-package installation process, so that users who are installing DAHDI for the first time don't have to download and install the dahdi-linux and dahdi-tools packages separately.

\=== dahdi-linux-2.0.0-rc4 ===

This is a release candidate of the DAHDI Linux kernel modules package, which replaces the kernel modules components of Zaptel. It contains all the functionality of Zaptel 1.4 plus many improvements, but also has some old (generally unsupported) functionality from Zaptel removed, including (but not limited to):

- Support for Linux 2.4.x kernels
- Support for devfs dynamic device filesystems
- The 'torisa' and 'wcusb' drivers

Information on upgrading from Zaptel to DAHDI can be found in the included UPGRADE.txt file, which can also be read here:

[http://svn.digium.com/view/dahdi/linux/tags/2.0.0-rc4/UPGRADE.txt?view=co](http://svn.digium.com/view/dahdi/linux/tags/2.0.0-rc4/UPGRADE.txt?view=co)

The change log for this release is here:

[http://downloads.digium.com/pub/telephony/dahdi-linux/releases/ChangeLog-2.0.0-rc4](http://downloads.digium.com/pub/telephony/dahdi-linux/releases/ChangeLog-2.0.0-rc4)

\=== asterisk-1.4.22-rc4 ===

This release candidate includes a large number of bug fixes and also is the first release of Asterisk 1.4 that includes support for DAHDI, the package that is replacing Zaptel. This version of Asterisk can be built against \*either\* Zaptel or DAHDI, but since Zaptel 1.4.12 is the last release of Zaptel 1.4, users are encouraged to transition to DAHDI as soon as they can, so that they will be able to continue to receive bug fixes and other improvements.

Information on how the transition from Zaptel to DAHDI affects this Asterisk release can be found in the included Zaptel-to-DAHDI.txt file, which can also be read here:

[http://svn.digium.com/view/asterisk/tags/1.4.22-rc4/Zaptel-to-DAHDI.txt?view=co](http://svn.digium.com/view/asterisk/tags/1.4.22-rc4/Zaptel-to-DAHDI.txt?view=co)

The change log for this release is here:

[http://downloads.digium.com/pub/telephony/asterisk/releases/ChangeLog-1.4.22-rc4](http://downloads.digium.com/pub/telephony/asterisk/releases/ChangeLog-1.4.22-rc4)

\=== asterisk-1.6.0-rc5 ===

This version of Asterisk contains a large number of enhancements and improvements over Asterisk 1.4, which can be found in the included CHANGES file, and read here:

[http://svn.digium.com/view/asterisk/tags/1.6.0-rc5/CHANGES?view=co](http://svn.digium.com/view/asterisk/tags/1.6.0-rc5/CHANGES?view=co)

\*All\* users who install this version of Asterisk are strongly encouraged to read the UPGRADE.txt file to learn about changes that affect compatibility with previous versions of Asterisk; this file can also be read here:

[http://svn.digium.com/view/asterisk/tags/1.6.0-rc5/UPGRADE.txt?view=co](http://svn.digium.com/view/asterisk/tags/1.6.0-rc5/UPGRADE.txt?view=co)

Information on how the transition from Zaptel to DAHDI affects this Asterisk release can be found in the included Zaptel-to-DAHDI.txt file, which can also be read here:

[http://svn.digium.com/view/asterisk/tags/1.6.0-rc5/Zaptel-to-DAHDI.txt?view=co](http://svn.digium.com/view/asterisk/tags/1.6.0-rc5/Zaptel-to-DAHDI.txt?view=co)

The change log for this release is here:

[http://downloads.digium.com/pub/telephony/asterisk/releases/ChangeLog-1.6.0-rc5](http://downloads.digium.com/pub/telephony/asterisk/releases/ChangeLog-1.6.0-rc5)

\=== ===

Thank you for your continued support of the Asterisk project!
