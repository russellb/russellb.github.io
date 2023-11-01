---
title: "Automated Testing Update"
date: "2010-07-30"
categories: 
  - "asterisk"
---

Greetings,

A while back, I posted [a message](http://lists.digium.com/pipermail/asterisk-dev/2010-February/042387.html) about an effort to improve automated testing in the Asterisk project. I wanted to give an update on how that project has progressed for those that have not been following along very closely.

We started using Bamboo as a continuous integration tool, which you can find running at [http://bamboo.asterisk.org](http://bamboo.asterisk.org)/. Note that some of the pass/fail statistics on there are a bit skewed, as the Bamboo server was just rebuilt and things were failing as everything was put back together.

A lot of really good automated test cases have been developed, and more are constantly being added. There are currently 85 test cases that run against Asterisk trunk after every change to the code. While some tests are small in scope, many of them cover significant call scenarios, such as various methods of doing transfers and call parking.

I apologize for the previous flood of Bamboo emails to the -dev list. :-) I now have a new mailing list created for those that would like to subscribe to those messages.

[http://lists.digium.com/mailman/listinfo/test-results](http://lists.digium.com/mailman/listinfo/test-results)

Additionally, one of the latest updates to our Bamboo setup is automated testing code coverage analysis. It will tell us exactly what code ran as a result of our automated test cases. It provides a good metric to start using to help identify areas of Asterisk that are in need of more test cases. You can find the code coverage reports for the latest builds of Asterisk trunk and 1.8 on Linux in the artifacts tab when viewing the details of a build.

[http://bamboo.asterisk.org/browse/AST-TRUNK/latest](http://bamboo.asterisk.org/browse/AST-TRUNK/latest)

I'm proud of the progress we have made so far and am excited to continue aggressive development of automated test cases for Asterisk. The tests we have are already catching problems on a regular basis. The resulting quality improvements make the job of the development team easier, as well as result in a better experience for end users.

If you're looking for a way to contribute to Asterisk and you are more comfortable writing scripts instead of C code, then the external test suite is a great way to get involved and help out.

Thank you all for your continued support of Asterisk!

Best Regards,

\-- Russell Bryant Digium, Inc. | Engineering Manager, Open Source Software 445 Jan Davis Drive NW - Huntsville, AL 35806 - USA jabber: rbryant@digium.com -=- skype: russell-bryant www.digium.com -=- www.asterisk.org -=- blogs.asterisk.org
