---
title: "Release Summary for Asterisk Releases"
date: "2009-02-26"
categories: 
  - "asterisk"
---

I just posted some information on a script I have been working on to generate a release summary to the [asterisk-dev list](http://lists.digium.com/pipermail/asterisk-dev/2009-February/036900.html).

\-------------------- Greetings,

In the past, when we made a new release of Asterisk, the only file that you had to get an idea of what changed in that release was the ChangeLog, which includes the developer oriented commit message for every change. There is also the CHANGES file which lists new features included in a new major release. Also, the release announcements generally contain a high level overview of some of the important changes.

In order to help improve visibility into what changes are being made, David Vossel and I have been working on a script that automatically generates a summary for a release. The intent would be to start generating these as a part of the release process and include them with every release. The intent is to provide something a bit more user friendly than the ChangeLog.

Currently, the script generates a document that includes the following information:

- The type of release (bug fix update, security update, etc.)
- People involved in making the release by coding, testing, or reporting issues.
- All issues closed from bugs.digium.com
- A list of "other changes", that didn't close out a bug report, and which bug reports these changes were related to, if any.
- A "diffstat", which gives an overview of which files were changed, and by how much.

The script is pretty much ready to be integrated into the release process, but I wanted to present example output to everyone for feedback first. So, take a look at these examples and let me know what you think. The script generates both an HTML version, and a plain text version.

Thanks!

Examples:

A bug-fix release: - [http://www.russellbryant.net/asterisk/asterisk-1.4.23-summary.html](http://www.russellbryant.net/asterisk/asterisk-1.4.23-summary.html) - [http://www.russellbryant.net/asterisk/asterisk-1.4.23-summary.txt](http://www.russellbryant.net/asterisk/asterisk-1.4.23-summary.txt)

A security release: - [http://www.russellbryant.net/asterisk/asterisk-1.4.23.1-summary.html](http://www.russellbryant.net/asterisk/asterisk-1.4.23.1-summary.html) - [http://www.russellbryant.net/asterisk/asterisk-1.4.23.1-summary.txt](http://www.russellbryant.net/asterisk/asterisk-1.4.23.1-summary.txt)

A feature release: - [http://www.russellbryant.net/asterisk/asterisk-1.6.1-summary.html](http://www.russellbryant.net/asterisk/asterisk-1.6.1-summary.html) - [http://www.russellbryant.net/asterisk/asterisk-1.6.1-summary.txt](http://www.russellbryant.net/asterisk/asterisk-1.6.1-summary.txt)

\-- Russell Bryant Digium, Inc. | Senior Software Engineer, Open Source Team Lead 445 Jan Davis Drive NW - Huntsville, AL 35806 - USA Check us out at: www.digium.com & www.asterisk.org
