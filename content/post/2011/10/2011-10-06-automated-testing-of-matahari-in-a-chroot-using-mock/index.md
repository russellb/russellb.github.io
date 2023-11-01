---
title: "Automated Testing of Matahari in a chroot Using Mock"
date: "2011-10-06"
---

While I was at Digium, I helped build out some automated testing for Asterisk (posts on that [here](http://www.russellbryant.net/blog/2010/02/16/asterisk-testing/) and [here](http://www.russellbryant.net/blog/2010/07/29/automated-testing-update/)). We had a continuous integration [server](http://bamboo.asterisk.org) that did builds, ran C API unit tests, and ran some functional tests written in Python.

One of the things that we wanted to do with the Asterisk tests is to sandbox each instance of Asterisk. All of this is handled by an [Asterisk Python class](http://svnview.digium.com/svn/testsuite/asterisk/trunk/lib/python/asterisk/asterisk.py?view=markup) which creates a new set of directories for each Asterisk instance to store its files. This seems to have worked pretty well. One down side is that there is still the potential for Asterisk instances to step on each others files. All instances of Asterisk for all runs of the tests run within the same OS installation. One way to improve that that is a bit less heavy handed than starting up a bunch of new VMs all the time is to use a chroot.

Over the last week or so, I have been working on a similar setup for [Matahari](http://www.matahariproject.org) and wanted to share some information on how it works and in particular, some aspects that are different from what I've done before, including running all of the tests in a chroot.

**What was already in place**

Matahari uses [Test-AutoBuild](http://autobuild.org/) as its continuous integration server. The results of the latest build for Fedora can be found [here](http://et3.redhat.com/hosted/matahari/).

When autobuild runs each hour, it clones the Matahari git repo and runs the [autobuild.sh](https://github.com/matahari/matahari/blob/master/autobuild.sh) script in the top level directory. This script uses [mock](http://fedoraproject.org/wiki/Projects/Mock) to build both matahari and mingw32-matahari packages. Mock handles setting up a chroot for the distribution and version you want to build a package for so you can easily do a lot of different builds on one machine. It also does a lot of caching to make this process much faster if you run it multiple times.

To install mock on Fedora, install the `mock` package. You will also need to add the user that will be running mock to the mock group.

To do a build in mock, you first need an srpm. The autobuild.sh script in Matahari has a make\_srpm function that does this. Once you have an srpm, you can do a build with a single call to mock. The --root option specifies the type of chroot you want mock to use.

`$ mock --root=fedora-16-x86_64 --rebuild *.src.rpm`

The root, fedora-16-x86\_64, is defined by a mock profile. When mock gets installed, a set of profiles gets installed, which can be found in /etc/mock/.

**What's new**

While doing continuous builds is useful (for example, breaking the Windows build is not terribly uncommon), doing tests against these builds adds an enormous amount of additional value to the setup. Similar to what we have for Asterisk, we have two sets of tests for Matahari. We have a suite of C API unit tests, and we have a suite of functional tests written in Python.</P.

The first thing I did was update the setup to run the unit tests. I decided to modify the RPM spec file to optionally run the unit tests as a part of the build process. That patch is [here](https://github.com/matahari/matahari/commit/c9024b70c4af25730c330fa5c16a1349c14a14e9). If the `run_unit_tests` variable is set, the spec file runs `ctest` after compilation is complete. The other aspect of this is getting this variable defined, which is pretty easy to do with mock.

`$ mock --root=fedora-16-x86_64 _--define "run_unit_tests 1"_ --rebuild *.src.rpm`

Getting the Python-based functional tests running within mock is a bit more tricky, but not too bad. With the mock commands presented so far, a number of things are happening automatically, including setting up the chroot and cleaning up the chroot. To get these other tests running, we have to break up the processes into smaller steps. The first step is to initialize a chroot. We will also be using another option for all of the mock commands, --resultdir, which lets you specify where logs and the resulting RPMs from --rebuild are stored.

`$ mock --root=fedora-16-x86_64 --resultdir=mockresults --init`

The next step is to build the RPMs like before. In this case, we already have an initialized chroot, so we need to tell mock not to create a new one. We also need to tell mock not to clean up the chroot after the RPM builds are complete, because we want to perform more operations in there.

`$ mock --root=fedora-16-x86_64 --resultdir=mockresults --define="run_unit_tests 1" --no-clean --no-cleanup-after --rebuild *.src.rpm`

At this point, we have compiled the source, run the unit tests, and built RPMs. The chroot used to do all of this is still there. We can take the RPMs we just built and install them into the chroot.

`$ mock --root=fedora-16-x86_64 --resultdir=mockresults --install mockresults/*.rpm`

Now it's time to set up the functional tests. We need to install some dependencies for the tests and then copy the tests themselves into the chroot.

`$ . src/tests/deps.sh $ mock --root=fedora-16-x86_64 --resultdir=mockresults --install ${MH_TESTS_DEPS}

$ mock --root=fedora-16-x86_64 --resultdir=mockresults --copyin src/tests /matahari-tests`

The dependencies of the tests are installed in the chroot and the tests themselves have been copied in. Now mock can be used to execute each set of tests.

`$ mock --root=fedora-16-x86_64 --resultdir=mockresults --shell "nosetests -v /matahari-tests/test_host_api.py"

$ mock --root=fedora-16-x86_64 --resultdir=mockresults --shell "nosetests -v /matahari-tests/test_sysconfig_api.py"

$ mock --root=fedora-16-x86_64 --resultdir=mockresults --shell "nosetests -v /matahari-tests/test_resource_api.py"

$ mock --root=fedora-16-x86_64 --resultdir=mockresults --shell "nosetests -v /matahari-tests/test_service_api_minimal.py"

$ mock --root=fedora-16-x86_64 --resultdir=mockresults --shell "nosetests -v /matahari-tests/test_network_api_minimal.py"`

That's it! The autobuild setup now tests compilation, unit tests, building RPMs, installing RPMs, and running and exercising all of the installed applications. It's fast, too.

**Final Thoughts**

As with most things, there are some areas for improvement. For example, one glaring issue is that the entire setup is Fedora specific, or at least specific to a distribution that can use mock. However, I have at least heard of a similar tool to mock called [pbuilder](https://wiki.ubuntu.com/PbuilderHowto) for dpkg based distributions, which could potentially be used in a similar way. I'm not sure.

There are also some issues with this approach specific to the Matahari project. Matahari includes a set of agents that provide systems management APIs. Testing of some of these APIs isn't necessarily something you want to do on the same machine running the actual tests. To expand to much more complete coverage of these APIs, we're going to have to break down and adopt an approach of spinning up VMs to run the Matahari agents. At that point, we may not run any of the functional tests from within mock anymore.

Mock is a very handy tool and it helped me to expand the automated build and test setup to get a lot more coverage in a shorter amount of time in a way that I was happy with. I hope this writeup helps you think about some other things that you could do with it.
