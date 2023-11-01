---
title: "Asterisk 1.6.0-beta4 Released"
date: "2008-02-21"
categories: 
  - "asterisk"
---

The Asterisk.org development team has released version 1.6.0-beta4.

Here are some highlights from the changes, with the associated issue numbers from bugs.digium.com if an issue was associated with the change.

This release contains the following improvements:

- 12020, a CLI formatting improvement
- 11964, added the ability to get the original called number on SS7 calls
- 11873, Added core API changes to handle T.38 origination and termination (The version of app\_fax in Asterisk-addons now supports this.)
- 11553, Added a status variable to the ChannelRedirect() application

The changes in this release include fixes for the following issues (trivial and minor issues not included):

- 11960, a crash in chan\_sip
- 12021, a crash related to invalid formats being specified for voicemail
- 11779, fix enabling echo cancellation for incoming SS7 calls
- 11740, DTMF handling fixes
- 11864, Fixed device state reporting on incoming calls on FXO
- 12012, a crash in chan\_local
- Fix a regression in codec handling that was introduced in 1.6.0-beta3

A full list of changes can be found in the ChangeLog. This release is available for immediate download from http://downloads.digium.com/.

Thank you for your support!
