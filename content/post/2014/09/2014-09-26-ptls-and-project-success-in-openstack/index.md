---
title: "PTLs and Project Success in OpenStack"
date: "2014-09-26"
categories: 
  - "openstack"
---

We're in the middle of another PTL change cycle.  [Nominations](http://lists.openstack.org/pipermail/openstack-dev/2014-September/046559.html) have occurred over the last week.  We've also seen several people step down this cycle ([Keystone](http://lists.openstack.org/pipermail/openstack-dev/2014-September/046679.html), [TripleO](http://lists.openstack.org/pipermail/openstack-dev/2014-September/046630.html), [Cinder](http://lists.openstack.org/pipermail/openstack-dev/2014-September/046840.html), [Heat](http://lists.openstack.org/pipermail/openstack-dev/2014-September/046663.html), [Glance](http://lists.openstack.org/pipermail/openstack-dev/2014-September/046713.html)).  This is becoming a regular thing in OpenStack.  The PTL position for most projects has changed hands over time.  The Heat project changes every cycle.  Nova has its 3rd PTL from a 3rd company (about to enter his 2nd cycle).  With all of this change, some people express some amount of discomfort and concern.

I think the change that we see is quite healthy.  This is our open governance model working well.  We should be thrilled that OpenStack is healthy enough that we don't rely on any one person to move forward.

I'd like to thank everyone who steps up to fill a PTL position.  It is a **huge** commitment.  The recognition you get is great, but it's really hard work.  It’s quite a bit more than just technical leadership.  It also involves project management and community management.  It's a position subject to a lot of criticism and a lot of the work is thankless.  So, thank you very much to those that are serving as a PTL or have served as one in the past.

It's quite important that everyone also realize that it takes a lot more than a PTL to make a project successful.  Our project governance and culture includes checks and balances.  There is a [Technical Committee](https://wiki.openstack.org/wiki/Governance/TechnicalCommittee) (TC) that is responsible for ensuring that OpenStack development overall remains healthy.  Some good examples of TC influence on projects would be the project reviews the TC has been doing over the last cycle, working with projects to apply course corrections ([Neutron](https://wiki.openstack.org/wiki/Governance/TechnicalCommittee/Neutron_Gap_Coverage), [Trove](https://wiki.openstack.org/wiki/Governance/TechnicalCommittee/Trove_Gap_Coverage), [Ceilometer](https://wiki.openstack.org/wiki/Governance/TechnicalCommittee/Ceilometer_Gap_Coverage), [Horizon](https://wiki.openstack.org/wiki/Governance/TechnicalCommittee/Horizon_Gap_Coverage), [Heat](https://wiki.openstack.org/wiki/Governance/TechnicalCommittee/Heat_Gap_Coverage), [Glance](https://wiki.openstack.org/wiki/Governance/TechnicalCommittee/Glance_Gap_Coverage)).  Most importantly, the PTL still must work for consensus of the project’s members (though is empowered to make the final call if necessary).

As a contributor to an OpenStack project, there is quite a bit you can do to help ensure project success beyond just writing code.  Here are some of those things:

# Help the PTL

While the PTL is held accountable for the project, they do not have to be responsible for all of the work that needs to get done.  The larger projects have started officially delegating responsibilities.  There is talk about [formalizing aspects of this](http://lists.openstack.org/pipermail/openstack-dev/2014-August/043812.html), so take a look at that thread for examples.

If you're involved in a project, you should work to understand these different aspects of keeping the project running.  See if there's an area you can help out with.  This stuff is critically important.

If you aspire to be a PTL at some point in the future, I would say getting involved in these areas is the best way you can grow your skills, influence, and visibility in the project to make yourself a strong candidate in the future.

# Participate in Project Discussions

The direction of a project is a result of many discussions.  You should be aware of these and participate in them as much as you can.  Watch the [openstack-dev mailing list](http://lists.openstack.org/cgi-bin/mailman/listinfo/openstack-dev) for discussions affecting your project.  It’s quite surprising how many people may be a core reviewer for a project but rarely participate in discussions on the mailing list.

Most projects also have an [IRC channel](https://wiki.openstack.org/wiki/IRC).  Quite a bit of day-to-day discussion happens there.  This can be difficult due to time zones or other commitments.  Just join when you can.  If it’s time zone compatible, you should definitely make it a priority to join [weekly project IRC meetings](https://wiki.openstack.org/wiki/Meetings).  This is an important time to discuss current happenings in the project.

Finally, attend and participate in the [design summit](https://wiki.openstack.org/wiki/Summit).  This is the time that projects try to sync up on goals for the cycle.  If you really want to play a role in affecting project direction and ensuring success, it’s important that you attend if possible.  Of course, there are many legitimate reasons some people may not be able to travel and those should be understood and respected by fellow project members.

Also keep in mind that project discussions span more than the project’s technical issues.  There are often project process and structure issues to work out.  Try to raise your awareness of these issues, provide input, and propose new ideas if you have them.  Some good recent examples of contributors doing this would be Daniel Berrange putting forth [a detailed proposal](http://lists.openstack.org/pipermail/openstack-dev/2014-September/044872.html) to split out the virt drivers from nova, or [Joe Gordon](https://review.openstack.org/#/c/116699/) and [John Garbutt](http://lists.openstack.org/pipermail/openstack-dev/2014-September/047120.html) pushing forward on evolving the blueprint handling process.

# Do the Dirty Work

Most contributors to OpenStack are contributing on behalf of an employer.  Those employers have customers (which may be internal or external) and those customers have requirements.  It’s understandable that some amount of development time goes toward implementing features or fixing problems that are important to those customers.

It’s also critical that everyone understands that there is a good bit of common work that must get done.  If you want to build goodwill in a project while also helping its success, help in these areas.  Some of those include:

- Help with [bug triage](https://wiki.openstack.org/wiki/BugTriage). (or specific page for [Nova](https://wiki.openstack.org/wiki/Nova/BugTriage))
- Help fix bugs in the project’s bug list.
- Watch what [bugs are affecting gate throughput](http://status.openstack.org/elastic-recheck/) and help debug and fix them.
- Work on some coding projects that clean up code and pay back technical debt.
- Do [code reviews](http://review.openstack.org).

# See You in Paris!

I hope this has helped some contributors think of new ways of helping ensuring the success of OpenStack projects.  I hope to see you on the mailing list, IRC, and in Paris!
