---
title: "Asterisk 1.4.22 and 1.6.0 Released"
date: "2008-10-02"
categories: 
  - "asterisk"
---

The Asterisk.org development team is proud to announce the releases of Asterisk 1.4.22 and 1.6.0.

\================================================================= === Asterisk 1.4.22 ============================================= =================================================================

Asterisk 1.4.22 includes a large number of bug fixes for the 1.4 release series of Asterisk. 1.4.22 also includes support for DAHDI. For more information about the transition from Zaptel to DAHDI, please see the following help file:

[http://svn.digium.com/view/asterisk/tags/1.4.22/Zaptel-to-DAHDI.txt?view=markup](http://svn.digium.com/view/asterisk/tags/1.4.22/Zaptel-to-DAHDI.txt?view=markup)

For a full listing of changes in this release, see the ChangeLog:

[http://svn.digium.com/view/asterisk/tags/1.4.22/ChangeLog?view=markup](http://svn.digium.com/view/asterisk/tags/1.4.22/ChangeLog?view=markup)

Asterisk 1.4.22 is available for immediate download from the Digium downloads site:

[http://downloads.digium.com/pub/telephony/asterisk/asterisk-1.4.22.tar.gz](http://downloads.digium.com/pub/telephony/asterisk/asterisk-1.4.22.tar.gz)

\================================================================= =================================================================

\================================================================= === Asterisk 1.6.0 ============================================== =================================================================

Asterisk 1.6.0 is the first official release of Asterisk 1.6.

\----------------------------------------------------------------- --- Upgrade Information ----------------------------------------- -----------------------------------------------------------------

Asterisk 1.6 no longer supports Zaptel. It only contains support for DAHDI. For more information on this transition, please see the following help file:

[http://svn.digium.com/view/asterisk/tags/1.6.0/Zaptel-to-DAHDI.txt?view=markup](http://svn.digium.com/view/asterisk/tags/1.6.0/Zaptel-to-DAHDI.txt?view=markup)

There are a number of other important changes to be aware of when upgrading to Asterisk 1.6.0 from previous versions of Asterisk. For a listing of those things, please see UPGRADE.txt:

[http://svn.digium.com/view/asterisk/tags/1.6.0/UPGRADE.txt?view=markup](http://svn.digium.com/view/asterisk/tags/1.6.0/UPGRADE.txt?view=markup)

\----------------------------------------------------------------- --- New Features ------------------------------------------------ -----------------------------------------------------------------

Asterisk 1.6.0 contains new features that were not previously available in an Asterisk release. For a full listing of the features that are included in Asterisk 1.6.0, please see the CHANGES file:

[http://svn.digium.com/view/asterisk/tags/1.6.0/CHANGES?view=markup](http://svn.digium.com/view/asterisk/tags/1.6.0/CHANGES?view=markup)

A verbose listing of each individual change that was made in the development of Asterisk 1.6.0 is also available:

[http://svn.digium.com/view/asterisk/tags/1.6.0/ChangeLog?view=markup](http://svn.digium.com/view/asterisk/tags/1.6.0/ChangeLog?view=markup)

\----------------------------------------------------------------- --- Release Management ------------------------------------------ -----------------------------------------------------------------

The Asterisk.org development team has decided on a new release management style for Asterisk 1.6. Previously, a release series was strictly feature frozen for its entire lifetime. The release management guidelines for Asterisk 1.6 were inspired by the Linux Kernel, among a number of other projects.

Asterisk 1.6 is not feature frozen. Features will be added in point releases. However, effort will be made to ensure that the number of large changes is minimized in a single point release. Even as 1.6.0 is being released, the Asterisk development team is already working on what new things will be in 1.6.1 and beyond. To take a look ahead to see what features have been added for releases that have not yet been made, take a look at the trunk version of the CHANGES file:

[http://svn.digium.com/view/asterisk/trunk/CHANGES?view=markup](http://svn.digium.com/view/asterisk/trunk/CHANGES?view=markup)

Even though new features will be added in point releases of Asterisk 1.6, that does not mean that any deprecated functionality will be removed as has been done between major releases in the past. In fact, we have decided that maintaining backwards compatibility is of the utmost importance for configuration and external interfaces. C API and ABI compatibility is not guaranteed between point releases. However, things like dialplan applications, functions, and AGI commands will not disappear just because there is a new and better way to accomplish the same thing.

With Asterisk 1.4, once Asterisk 1.4.N is released, Asterisk 1.4.X is no longer supported, where X < N. With Asterisk 1.6, the development team plans to maintain a total of 3 releases at a time. For example, the development team will support Asterisk 1.6.0, 1.6.1, and 1.6.2 until 1.6.3 is released. This means that for the time that 1.6.0 is supported, there may be 1.6.0.1, 1.6.0.2, etc. releases that include fixes for regressions found in 1.6.0.

With Asterisk 1.4, the goal has been to make releases every 4 to 6 weeks. With Asterisk 1.6, we aim to release updates in a similar time frame, but it is likely that it will be closer to 6 to 8 weeks for Asterisk 1.6 due to a more strict beta and release candidate testing cycle for each point release.

The number of releases supported and release time frames are not yet set in stone. The development team is interested in adjusting these policies to help meet the desires of the Asterisk community.

\----------------------------------------------------------------- --- Download ---------------------------------------------------- -----------------------------------------------------------------

Asterisk 1.6.0 is available for immediate download from the Digium downloads site:

[http://downloads.digium.com/pub/telephony/asterisk/asterisk-1.6.0.tar.gz](http://downloads.digium.com/pub/telephony/asterisk/asterisk-1.6.0.tar.gz)

\----------------------------------------------------------------- --- Reporting Issues -------------------------------------------- -----------------------------------------------------------------

The bug tracking process is one of the best places to get started in the Asterisk development community. Please report all issues to [http://bugs.digium.com/](http://bugs.digium.com/). Live discussion about current issues is done on the Freenode IRC network, in the #asterisk-bugs or #asterisk-dev channels.

\================================================================= =================================================================

Thank you very much for your continued support of the Asterisk project!
