---
title: "Asterisk 1.10 Update"
date: "2011-02-17"
categories: 
  - "asterisk"
---

I just posted [an update on the development of Asterisk 1.10](http://lists.digium.com/pipermail/asterisk-dev/2011-February/048028.html) to the asterisk-dev mailing list. Here is the content:

> Greetings,
> 
> Shortly after the release of Asterisk 1.8, we had a developer meeting and discussed some of the projects that people would like to see in Asterisk 1.10 \[1\]. We discussed the schedule there a bit, as well. Now that Asterisk 1.8 has settled down and we are well into the development cycle for Asterisk 1.10, it is a good time to revisit the plans for the next release.
> 
> At Digium, the biggest thing we have been working on for 1.10 so far is replacing the media infrastructure in Asterisk. Most of the critical and invasive plumbing work is done and has been merged into trunk. Next we're looking at building up some features on top of that, such as adding more codecs, enhancing ConfBridge() to support additional sampling rates (HD conferencing), adding features that exist in MeetMe() but not ConfBridge(), and enhancing codec negotiation.
> 
> Of course, many others have been working on new developments as well. I would encourage you to respond if you'd like to provide an update on some new things that you're working on.
> 
> We would like to release Asterisk 1.10 roughly a year after Asterisk 1.8. This will be a standard release, not LTS \[2\]. To have the release out in the October time frame, we need to branch off 1.10 (feature freeze) at the end of June. At that point we will begin the beta and RC process. If you're working on new development projects that you would like to get into Asterisk 1.10, please keep this timeline in mind.
> 
> As always, comments and questions are welcome.
> 
> Thanks,
> 
> \[1\] https://wiki.asterisk.org/wiki/display/AST/AstriDevCon+2010 \[2\] https://wiki.asterisk.org/wiki/display/AST/Asterisk+Versions
> 
> \-- Russell Bryant
