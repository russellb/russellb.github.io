---
title: "Asterisk Clustering - Improvements for distributed services"
date: "2007-09-09"
categories: 
  - "asterisk"
---

One of the areas that I am very interested in right now is coming up with new ways to distribute Asterisk services across multiple servers, both locally with Asterisk clustering, or distributed with servers across the internet. Off and on this year I have been working on an event system. I want to have a framework that is easy to use such that when certain events occur on one server, they can be propogated to a federation of servers.

This effort is broken down to two main parts:

1. Internal Event API
2. Event Distribution Methods

**Internal Event API**

The first phase of this effort is to come up with a generic API for generating events inside of Asterisk. This is different than the Asterisk Manager Interface (AMI). The code for generating Asterisk manager events is very specific to the manager interface and does not lend itself toward using this information anywhere else in Asterisk.

I consider this part of the project to already be pretty mature. There is a generic, binary encoded event publish and subscribe API in Asterisk trunk. I have already converted some things to use it, such as Voicemail message waiting information (more on this later!), but a more aggressive effort to convert things to use this API will be done after phase 2 has matured.

Here is the [header file](http://svn.digium.com/view/asterisk/trunk/include/asterisk/event.h?view=markup) for this API if you are curious.

**Event Distribution Methods**

This part is a little bit more tricky than the first. The project here is to take the binary encoded events that are passed around using the internal event API and distribute them between a distributed network of Asterisk servers. This part is still very much in development.

I have completed a module to implement one method of distributing events. I set it up and verified that voicemail message waiting state was distributed and available between multiple servers. This approach works great for some network scenarios, but not for others.

**Details**

I have more to say on the specifics of what I'm working on. I will be making follow up posts on this topic to describe details of the event distribution methods I come up with as well as what features are being converted to take advantage of these new underlying features. I can say now that the areas that I have been working on so far are sharing voicemail and device state (presence) information.

I hope that people will find this work useful as it progresses.
