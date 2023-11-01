---
title: "Asterisk 1.6.0-beta5 Now Available"
date: "2008-03-05"
categories: 
  - "asterisk"
---

The Asterisk.org development team has released Asterisk 1.6.0-beta5. As of this beta of 1.6.0, 1.6.0 is now feature frozen. In addition to a number of bug fixes, the following new features have been added since beta4:

- The SMDI interface in Asterisk has been reworked to fix a number of issues as well as add some new features. SMDI message information is now accessed in the dialplan using some new dialplan functions. New options have been added to map Asterisk voicemail boxes to SMDI station IDs. Also, MWI will now properly be sent for systems that have some external interface modifying voicemail boxes, such as a web interface, or with an email client in the case of IMAP storage.
    
- The Postgres CDR module now supports some of the features of cdr\_adaptive\_odbc. Specifically, you may add additional columns into the table and they will be set, if you set the corresponding CDR variable name. Also, if you omit columns in your database table, those fields will be silently skipped when inserting the record.
    
- The ResetCDR application now has an 'e' option that re-enables the CDR if it has been disabled using the NoCDR option.
    
- A new CLI command, "devstate change", has been added which allows you to change the state of a Custom device. Custom device states were previously only settable by using the DEVICE\_STATE() dialplan function.
    
- The Originate manager action now has its own permission level called originate. Also, if you want this action to be able to execute applications that call out to a subshell, it requires the system privilege, as well. These changes were made to enhance the security of the manager interface.

For a full list of features that have been introduced from Asterisk 1.4 to Asterisk 1.6.0, see the following file:

- [http://svn.digium.com/view/asterisk/branches/1.6.0/CHANGES?view=markup](http://svn.digium.com/view/asterisk/branches/1.6.0/CHANGES?view=markup)

For a full list of changes to Asterisk 1.6.0 from beta4 to beta5, see the ChangeLog:

- [http://svn.digium.com/view/asterisk/tags/1.6.0-beta5/ChangeLog?view=markup](http://svn.digium.com/view/asterisk/tags/1.6.0-beta5/ChangeLog?view=markup)

There are a few more issues to resolve in 1.6.0 before it can enter release candidate status, but we expect that to happen relatively soon.

Thank you for your continued support of Asterisk!
