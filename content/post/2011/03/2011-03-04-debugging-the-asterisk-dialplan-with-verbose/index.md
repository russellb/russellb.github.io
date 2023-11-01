---
title: "Debugging the Asterisk Dialplan with Verbose()"
date: "2011-03-04"
categories: 
  - "asterisk"
---

[Leif Madsen](http://leifmadsen.wordpress.com/) and I are working on a new book, the [Asterisk Cookbook](http://oreilly.com/catalog/0636920018551/). One of the recipes that I am working on this morning is a method of adding debug statements into the Asterisk dialplan. I came up with a GoSub() routine that can log messages based on log level settings that are global, per-device, or per-channel. Here's a preview. I hope you find it useful!

### Channel logging GoSub() routine.

- **ARG1** - Log level.
- **ARG2** - The log message.

Channel logging using this routine will be sent to the Asterisk console at verbose level 0, meaning that they will show up when you want them to regardless of the current "core set verbose" setting. This routine uses a different method, values in AstDB, to control what messages show up.

**AstDB entries:**

- Family: ChanLog/ Key: all

- If the log level is less than or equal to this value the message will be printed.

- Family: ChanLog/ Key: channels/

- This routine will also check for a channel specific debug setting. It will actually check for both the full channel name as well as just the part of the channel name before '-'. This allows setting a debug level for all calls from a particular device. For example, a SIP channel may be "SIP/myphone-0011223344". This routine will check:

- Family: ChanLog/ Key: channels/SIP/myphone
- Family: ChanLog/ Key: channels/SIP/myphone-0011223344

**Example Dialplan Usage:**
```
exten => 7201,1,GoSub(chanlog,s,1(1,${CHANNEL} has called ${EXTEN}))

```

**Example of enabling debugging for a device from the Asterisk CLI:**
```
*CLI> database put ChanLog SIP/myphone 3 

```

**chanlog routine implementation:**
```
[chanlog]

exten => s,1,GotoIf($[${DB_EXISTS(ChanLog/all)} = 0]?checkchan1)
    same => n,GotoIf($[${ARG1}  n(checkchan1),Set(KEY=ChanLog/channel/${CHANNEL})
    same => n,GotoIf($[${DB_EXISTS(${KEY})} = 0]?checkchan2)
    same => n,GotoIf($[${ARG1}  n(checkchan2),Set(KEY=ChanLog/channel/${CUT(CHANNEL,-,1)})
    same => n,GotoIf($[${DB_EXISTS(${KEY})} = 0]?return)
    same => n,GotoIf($[${ARG1}  n(return),Return() ; Return without logging
    same => n(log),Verbose(0,${ARG2})
    same => n,Return()

```
