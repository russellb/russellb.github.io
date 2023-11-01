---
title: "New Versions of Asterisk, Asterisk-addons, Zaptel, and DAHDI"
date: "2008-09-03"
categories: 
  - "asterisk"
  - "dahdi"
  - "digium"
  - "zaptel"
---

The Asterisk development team is pleased to announce new releases of Asterisk, Asterisk-Addons, Zaptel, and for the first time, DAHDI.

This is a set of coordinated releases intended to begin the transition from Zaptel to DAHDI; for the reasons why this is being done, please see:

[http://blogs.digium.com/2008/05/19/zaptel-project-being-renamed-to-dahdi/](http://blogs.digium.com/2008/05/19/zaptel-project-being-renamed-to-dahdi/)

The list of packages released today includes:

- zaptel 1.2.27
- zaptel 1.4.12
- dahdi-linux 2.0.0-rc3
- dahdi-tools 2.0.0-rc2
- dahdi-linux-complete 2.0.0-rc3+2.0.0-rc2
- asterisk 1.4.22-rc3
- asterisk 1.6.0-rc4
- asterisk-addons 1.6.0-rc1

All of these packages are available on http://downloads.digium.com in their respective directories. Detailed information about each package release is included below.

\=== zaptel-1.2.27 ===

This will be the final release of Zaptel made from the 1.2 release branch. This release includes a number of bug fixes and other improvements, most notably compatibility with the 2.6.26 and 2.6.27 kernels and a fix for wctdm driver (for TDM400P cards) that corrects a problem introduced in 1.2.26 that caused FXO ports to not properly recognize incoming calls.

The change log for this release is here:

[http://downloads.digium.com/pub/telephony/zaptel/releases/ChangeLog-1.2.27](http://downloads.digium.com/pub/telephony/zaptel/releases/ChangeLog-1.2.27)

\=== zaptel-1.4.12 ===

This will be the final release of Zaptel made from the 1.4 release branch. This release includes a number of bugs fixes and other improvements, including the changes that are listed above for the Zaptel 1.2.27 release.

In addition, there are two major changes in this release:

\- Support for the Digium TC400B transcoder card has been completely rewritten, and the API used by Asterisk (or any other application) to the card has been changed significantly. The result of this work is that the transcoder interface is more reliable and stable, and it supports variable-sized G.729 frames (including G.729B silence detection frames). However, that means that any Asterisk user using a TC400B who wants to upgrade to this version of Zaptel must \*also\* upgrade their version of Asterisk to one of the releases included in this announcement; older versions of Asterisk will not be able to use the transcoder support in this release of Zaptel.

\- To help users make the transition to DAHDI (and especially if they need to move back to Zaptel for some reason), this version of Zaptel contains installation steps that will \*uninstall\* the important parts of DAHDI if they are present on the system during the 'make install' step. This will allow a user to 'switch back' to Zaptel from DAHDI without having to manually uninstall any portions of DAHDI.

The change log for this release is here:

[http://downloads.digium.com/pub/telephony/zaptel/releases/ChangeLog-1.4.12](http://downloads.digium.com/pub/telephony/zaptel/releases/ChangeLog-1.4.12)

\=== dahdi-linux-complete-2.0.0-rc3+2.0.0-rc2 ===

This release combines dahdi-linux-2.0.0-rc3 and dahdi-tools-2.0.0-rc2 into a single download, one-package installation process, so that users who are installing DAHDI for the first time don't have to download and install the dahdi-linux and dahdi-tools packages separately.

\=== dahdi-linux-2.0.0-rc3 ===

This is the first release candidate of the DAHDI Linux kernel modules package, which replaces the kernel modules components of Zaptel. It contains all the functionality of Zaptel 1.4 plus many improvements, but also has some old (generally unsupported) functionality from Zaptel removed, including (but not limited to):

- Support for Linux 2.4.x kernels
- Support for devfs dynamic device filesystems
- The 'torisa' and 'wcusb' drivers

Information on upgrading from Zaptel to DAHDI can be found in the included UPGRADE.txt file, which can also be read here:

[http://svn.digium.com/view/dahdi/linux/tags/2.0.0-rc3/UPGRADE.txt?view=co](http://svn.digium.com/view/dahdi/linux/tags/2.0.0-rc3/UPGRADE.txt?view=co)

The change log for this release is here:

[http://downloads.digium.com/pub/telephony/dahdi-linux/releases/ChangeLog-2.0.0-rc3](http://downloads.digium.com/pub/telephony/dahdi-linux/releases/ChangeLog-2.0.0-rc3)

\=== dahdi-tools-2.0.0-rc2 ===

This is the first release candidate of the DAHDI userspace tools package, which replaces the userspace components of Zaptel. It contains all the functionality of Zaptel 1.4 plus many improvements.

Information on upgrading from Zaptel to DAHDI can be found in the included UPGRADE.txt file, which can also be read here:

[http://svn.digium.com/view/dahdi/tools/tags/2.0.0-rc2/UPGRADE.txt?view=co](http://svn.digium.com/view/dahdi/tools/tags/2.0.0-rc2/UPGRADE.txt?view=co)

The change log for this release is here:

[http://downloads.digium.com/pub/telephony/dahdi-tools/releases/ChangeLog-2.0.0-rc2](http://downloads.digium.com/pub/telephony/dahdi-tools/releases/ChangeLog-2.0.0-rc2)

\=== asterisk-1.4.22-rc3 ===

This release candidate includes a large number of bug fixes and also is the first release of Asterisk 1.4 that includes support for DAHDI, the package that is replacing Zaptel. This version of Asterisk can be built against \*either\* Zaptel or DAHDI, but since Zaptel 1.4.12 is the last release of Zaptel 1.4, users are encouraged to transition to DAHDI as soon as they can, so that they will be able to continue to receive bug fixes and other improvements.

Information on how the transition from Zaptel to DAHDI affects this Asterisk release can be found in the included Zaptel-to-DAHDI.txt file, which can also be read here:

[http://svn.digium.com/view/asterisk/tags/1.4.22-rc3/Zaptel-to-DAHDI.txt?view=co](http://svn.digium.com/view/asterisk/tags/1.4.22-rc3/Zaptel-to-DAHDI.txt?view=co)

The change log for this release is here:

[http://downloads.digium.com/pub/telephony/asterisk/releases/ChangeLog-1.4.22-rc3](http://downloads.digium.com/pub/telephony/asterisk/releases/ChangeLog-1.4.22-rc3)

\=== asterisk-1.6.0-rc4 ===

This release candidate marks the transition of the Asterisk 1.6 release branch from 'beta' status to 'release candidate' status, signifying that it is nearing final release. This version of Asterisk contains a large number of enhancements and improvements over Asterisk 1.4, which can be found in the included CHANGES file, and read here:

[http://svn.digium.com/view/asterisk/tags/1.6.0-rc4/CHANGES?view=co](http://svn.digium.com/view/asterisk/tags/1.6.0-rc4/CHANGES?view=co)

\*All\* users who install this version of Asterisk are strongly encouraged to read the UPGRADE.txt file to learn about changes that affect compatibility with previous versions of Asterisk; this file can also be read here:

[http://svn.digium.com/view/asterisk/tags/1.6.0-rc4/UPGRADE.txt?view=co](http://svn.digium.com/view/asterisk/tags/1.6.0-rc4/UPGRADE.txt?view=co)

Information on how the transition from Zaptel to DAHDI affects this Asterisk release can be found in the included Zaptel-to-DAHDI.txt file, which can also be read here:

[http://svn.digium.com/view/asterisk/tags/1.6.0-rc4/Zaptel-to-DAHDI.txt?view=co](http://svn.digium.com/view/asterisk/tags/1.6.0-rc4/Zaptel-to-DAHDI.txt?view=co)

The change log for this release is here:

[http://downloads.digium.com/pub/telephony/asterisk/releases/ChangeLog-1.6.0-rc4](http://downloads.digium.com/pub/telephony/asterisk/releases/ChangeLog-1.6.0-rc4)

\=== asterisk-addons-1.6.0-rc1 ===

This release candidate marks the transition of the Asterisk-Addons 1.6 release branch from 'beta' status to 'release candidate' status, signifying that it is nearing final release. This version of Asterisk-Addons contains a Bluetooth device channel driver called chan\_mobile, as well as a number of smaller improvements to existing modules.

\*All\* users who install this version of Asterisk-Addons are strongly encouraged to read the UPGRADE.txt file to learn about changes that affect compatibility with previous versions of Asterisk-Addons; this file can also be read here:

[http://svn.digium.com/view/asterisk-addons/tags/1.6.0-rc1/UPGRADE.txt?view=co](http://svn.digium.com/view/asterisk-addons/tags/1.6.0-rc1/UPGRADE.txt?view=co)

The change log for this release is here:

[http://downloads.digium.com/pub/telephony/asterisk/releases/ChangeLog-addons-1.6.0-rc1](http://downloads.digium.com/pub/telephony/asterisk/releases/ChangeLog-addons-1.6.0-rc1)

\=== ===

Thank you for your continued support of the Asterisk project!
