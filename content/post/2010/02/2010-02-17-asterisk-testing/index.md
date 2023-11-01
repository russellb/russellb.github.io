---
title: "Asterisk Testing"
date: "2010-02-17"
categories: 
  - "asterisk"
---

See the [original post on the asterisk-dev list](http://lists.digium.com/pipermail/asterisk-dev/2010-February/042387.html).

Greetings,

I think we can do a much better job at testing within the Asterisk project and I suspect you do, too. The vast install base and wide variety of uses gets us very good feedback. However, it is clear that we can do better within the development community at improving the quality of the code before it makes it out in any type of release (beta, RC, or full release).

I would like to propose that we adopt an automated testing approach as a part of our development process. There are 4 pieces to this:

1. Testing with our eyes

3. Bottom-Up testing using unit tests within Asterisk

5. Top-Down testing using an external test suite

7. Tests running constantly using a continuous integration framework

While this isn't automated testing, it is too valuable not to mention. One of the most effective ways of finding bugs in code is by peer code review. We see this more and more as we continue to increase our usage of ReviewBoard for peer review before changes are merged. It's rare that a change goes in as the first proposed patch that goes on ReviewBoard. While it has never been a hard rule, it is strongly encouraged that every committer to seek a sign-off from at least one other committer via ReviewBoard for \_every\_ non-trivial change.

[https://reviewboard.asterisk.org/](https://reviewboard.asterisk.org)

For unit testing, we have a new API in Asterisk trunk for writing unit tests within the code. We're up to 20 unit tests. Writing these tests has already found multiple bugs and running them continuously has caught one regression before it made it into a release.

See [include/asterisk/test.h](http://svn.digium.com/view/asterisk/trunk/include/asterisk/test.h?view=markup)

The beginnings of an external test suite have been created. There is still design work to do, but hopefully it will begin to be populated with tests soon. In general, though, this will become a suite of tests that can use whatever external tools are necessary to verify Asterisk functionality. There is a README.txt that documents what is there so far.

```
$ svn co http://svn.digium.com/svn/testsuite/asterisk/trunk
```

On the continuous integration front, we have been using buildbot for a long time for doing compilation on various systems. It's time that we extend our continuous integration approach to not only compile Asterisk, but also run regression tests. After evaluating some different options, Bamboo has been installed and configured for our use. So far, it is compiling all release branches of Asterisk. For Asterisk trunk, it is running all of the unit tests after every change to the code. The results of the tests are integrated into the application.

[Bamboo - The Asterisk Project](http://bamboo.asterisk.org/)

The only build slave right now is Linux. This will be extended to other operating systems. If you would be interested in providing a build slave and assisting with its maintenance, please talk to me. We can configure up to 25 build slaves ("remote agents" in Bamboo terminology).

If you have commit access, you are strongly encouraged to sign up for an account on bamboo.asterisk.org and associate your account with your svn username. That way, bamboo can notify you by email and/or jabber if there any problems associated with changes that you make.

As you work on changes to the Asterisk code, you should consider this approach to testing along side your development. If there is a unit test you could write to help prove your code works or prevent a bug from showing up again, then please write one! If there is an external test you could write to help verify some piece of functionality you are using continues to work precisely as expected, then let's figure out how to get that into the test suite. This kind of testing not only prevents regressions but it also guarantees that expected behavior will remain intact across releases.

I am excited about what this effort can do for the quality of Asterisk in the long term, and I hope you are, too. I look forward to your feedback on the approach as well as participation in the development of our testing infrastructure.

Thanks,

\-- Russell Bryant Digium, Inc. | Engineering Manager, Open Source Software 445 Jan Davis Drive NW - Huntsville, AL 35806 - USA www.digium.com -=- www.asterisk.org -=- blogs.asterisk.org
