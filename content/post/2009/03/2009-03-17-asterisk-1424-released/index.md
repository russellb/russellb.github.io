---
title: "Asterisk 1.4.24 Released"
date: "2009-03-17"
categories: 
  - "asterisk"
  - "zaptel"
---

The Asterisk Development Team is proud to announce release of Asterisk 1.4.24, and is available for immediate download at [http://downloads.digium.com/](http://downloads.digium.com/)

In addition to other bug fixes, this release candidate fixes several crash issues, and resolved some remaining issues related to call pickup and call parking that were discovered after the release of Asterisk 1.4.23. In addition, issues related to chan\_iax2, and regressions introduced to the 'h' extension have been resolved.

This release marks the first inclusion of the release summary files which will be included in all future releases. The purpose is to give a clearer overview of the changes that have taken place between the current and previous release, which issues have been closed, and which community members were involved with issue submission, code commits, and issue testing. Additionally, a diffstat at the end of the file shows at a brief glance the number of changes made to files between the previous and current releases.

For a summary of the changes in this release, please see the release summary:

[http://svn.digium.com/view/asterisk/tags/1.4.24/asterisk-1.4.24-summary.html?view=co](http://svn.digium.com/view/asterisk/tags/1.4.24/asterisk-1.4.24-summary.html?view=co)

For a full list of changes in this release, please see the ChangeLog:

[http://svn.digium.com/view/asterisk/tags/1.4.24/ChangeLog?view=co](http://svn.digium.com/view/asterisk/tags/1.4.24/ChangeLog?view=co)

The following list of bugs were resolved with the participation of the community, and this release would not have been possible without your help!

\* Paging application crashes asterisk - Closes issue #14308. Submitted by bluefox. Tested by kc0bvu. Patched by seanbright. \* Crash in VoiceMailMain if hangup occurs before a valid mailbox number is entered (IMAP only) - Closes issue #14473. Submitted by, and patch provided by dwpaul.

\* Incoming Gtalk calls fail - Closes issue #13984. Submitted by, tested, and patched by jcovert.

\* Realtime peers are never qualified after 'sip reload' - Closes issue #14196. Submitted by, tested, and patched by pdf.

\* SIP Attended Transfer fails - Closes issue 14611. Submitted by, tested, and patched by klaus3000.

Thank you for your continued support of Asterisk!
