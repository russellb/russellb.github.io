---
title: "JACK interfaces for Asterisk"
date: "2008-01-14"
categories: 
  - "asterisk"
tags: 
  - "jack"
  - "puredata"
---

I just merged a new module that I have been working on into trunk. The module is app\_jack. It provides interfaces between Asterisk and [JACK](http://www.jackaudio.org) (Jack Audio Connection Kit). A description of JACK from their website:

> Have you ever wanted to take the audio output of one piece of software and send it to another? How about taking the output of that same program and send it to two others, then record the result in the first program? If so, JACK may be what you've been looking for.
> 
> JACK is a low-latency audio server, written for POSIX conformant operating systems such as GNU/Linux and Apple's OS X. It can connect a number of different applications to an audio device, as well as allowing them to share audio between themselves. Its clients can run in their own processes (ie. as normal applications), or can they can run within the JACK server (ie. as a "plugin").

I recently learned about [PureData](http://puredata.info), so very quickly I wanted to be able to get phone calls in and out of Pd (PureData). After exploring various options, JACK turned out to be the obvious choice for providing audio transport between Asterisk and Pd. I have been wanting to build some interesting applications in Pd that do audio analysis and manipulation, but I had some infrastructure work to complete first.

So, I wrote app\_jack. This module provides two interfaces between Asterisk and JACK. The first is an Asterisk application, JACK(). The second interface is a dialplan function, JACK\_HOOK().

**JACK() Application**

This is the simpler of the two interfaces. When the JACK() application is executed in the Asterisk dialplan, two JACK ports get created. There is an input and output port that acts as the endpoint of a phone call. The audio from the channel goes out of the output port that gets created. Whatever audio that comes in on the input port is what gets sent back to the caller. This would allow for some advanced voice applications that interact with the audio of the call and run as a separate application.

Example: `exten => 1234,1,Answer exten => 1234,n,JACK()`

For more information, see the built in documentation for the application from the Asterisk CLI: `*CLI> core show application JACK`

**JACK\_HOOK() Function**

This interface is a little bit more complex, but it is the much more interesting one, in my opinion. The JACK\_HOOK function creates an [audiohook](http://russellbryant.net/blog/?p=12) and attaches it to the channel. In this case, instead of the JACK interface being the endpoint of the phone call, it is simply hooked in to the audio path for a phone call to something else. Audio that comes from a caller gets sent out the JACK output port, and whatever audio that comes back in on the input port gets sent on as the caller's audio. This allows for cool applications that do analysis and manipulation of the audio in a phone call. One example is that I can now write custom vocoders in Pd.

For more information about the syntax for using JACK\_HOOK, see the built in documentation from the Asterisk CLI: `*CLI> core show function JACK_HOOK`

Here is a simple example for enabling a JACK\_HOOK for a call: `exten => _1NXXNXXXXXX,1,Answer exten => _1NXXNXXXXXX,n,Set(JACK_HOOK(manipulate)=on) exten => _1NXXNXXXXXX,n,Dial(IAX2/myprovider/${EXTEN})`

This also inspired me to write a new CLI command, "core set chanvar", which lets you set a channel variable or function from the Asterisk CLI. You can use it to turn on a JACK\_HOOK for an existing call from the CLI. `*CLI> core set chanvar SIP/poly1-ab23jadf234 JACK_HOOK(manipulate) on`

Another way to enable a JACK\_HOOK for an existing call would be by using the manager interface. For example: `Action: SetVar Channel: SIP/123-f234kjsdfz Variable: JACK_HOOK(manipulate) Value: on`

Enjoy! Please let me know if you find this interesting or useful. I would also be happy to consider any suggestions for enhancements.
