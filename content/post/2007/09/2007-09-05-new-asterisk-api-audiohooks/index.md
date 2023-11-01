---
title: "New Asterisk API: Audiohooks"
date: "2007-09-05"
categories: 
  - "asterisk"
---

There is a new Asterisk API in town, and it is called Audiohooks. It was developed by Joshua Colp, a fellow Asterisk developer and friend. As you may have inferred from the name, the audiohooks API lets you add hooks into the audio passing through an Asterisk channel. There are a lot of exciting things that can be done with this, including places where it is already in use.

First, let me explain how this came about. Since Asterisk 1.2, there have been a couple of applications called ChanSpy and MixMonitor that allow you to listen in on any call on the system or record any call, respectively. _(Even earlier than 1.2, there was the Monitor application, but MixMonitor is more efficient when it comes to mixing the Tx and Rx audio for a channel.)_ This is especially useful in call centers ... or for Big Brother. However, the implementation has been problematic. It has been a long road of crashes and deadlocks that has caused the underlying code to be re-written multiple times. The Audiohooks is the latest chapter in the saga of ChanSpy and MixMonitor.

If you're hardcore, here is the [header file](http://svn.digium.com/view/asterisk/trunk/include/asterisk/audiohook.h?view=markup "header file") that defines the API. However, I'll continue here and give an explanation of the cool things this will let us do.

The Audiohooks API can be used in one of 3 modes.

1. Spy
2. Whisper
3. Manipulate

The spy mode allows the code to receive all audio coming in and out of a channel. This is used for both ChanSpy and MixMonitor today. The Whisper mode allows code to provide audio that will be mixed with the audio coming in and/or out of a channel. This is in use today for the whisper mode of ChanSpy.

The Manipulate mode is perhaps the most exciting mode. It allows code to modify the audio coming in and out of a channel. This is being used today in the func\_volume module. This module provides the VOLUME() dialplan function which provides channel technology independent gain control. You can use it and press digits on your phone to increase or decrease the volume of the audio coming in and out of your channel.

Here are some more ideas for things that this API would make fairly straight forward to implement:

1. [Application independent announcements](http://russellbryant.net/blog/?p=11) - This API would make it pretty easy to implement a generic mechanism for scheduled and periodic announcements that would just be mixed with whatever other audio is going to the channel from whatever application it is currently executing.
2. Cool Audio Effects. People have written modules before, such as [this one](http://www.lobstertech.com/code/voicechanger//), that allow you to add fancy effects to your voice in Asterisk. However, none of them have ever made it into the tree because we wanted to agree on a solid API for doing this type of thing first.
3. Fake static. For years, developers have been joking around about adding the ability to introduce fake static into the audio of a call. You know, let's say you've been on the phone for 30 minutes and you really just want to hang up, but you'd like to blame it on something else. Well, just start pressing the magic key on your phone to start increasing the amount of static introduced into the audio! _"I'm losing you! Bad connection! I'll talk to you later!"._

There are plenty more cool things that could be done. If you have an idea, I'd love to hear it. Most of all, thanks to Josh for the cool code!
