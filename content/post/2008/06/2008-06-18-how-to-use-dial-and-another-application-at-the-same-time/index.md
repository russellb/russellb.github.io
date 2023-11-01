---
title: "How-to: Use Dial and Another Application at the Same Time"
date: "2008-06-18"
categories: 
  - "asterisk"
---

I spend a lot of time on IRC talking about Asterisk. A lot of questions get answered there but those answers are not archived as well as they are when questions are answered on mailing lists or forums. So, I'll start posting some questions and answers that come from IRC discussions.

This one is a simple example that shows the usefulness of Local channels in Asterisk.

In #asterisk:

Question: How can I run the Dial() and FollowMe() applications at the same time?

My Answer: It is quite simple. The Dial() application allows you to call multiple channels at the same time. If you use a Local channel, one of the things that you are dialing can be another chunk of dialplan. For example, call 1234:

`[default] exten => 1234,1,Dial(SIP/myphone&Local/followme@default) exten => followme,1,FollowMe(default)`

Here is another common question that can be answered by the use of Local channels:

Question: How do I call a second phone after a first phone has been ringing for some period of time, while still allowing the first phone to ring?

Answer: This can be accomplished by the use of Local channels. Let's say you want to ring phone2 after phone1 has been ringing for 10 seconds, while still allowing phone1 to ring, as well.

`[default] exten => 1234,1,Dial(SIP/phone1&Local/call_phone2@default) exten => call_phone2,1,Wait(10) exten => call_phone2,n,Dial(SIP/phone2)`
