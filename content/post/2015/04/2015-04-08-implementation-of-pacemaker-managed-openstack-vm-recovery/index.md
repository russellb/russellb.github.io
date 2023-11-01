---
title: "Implementation of Pacemaker Managed OpenStack VM Recovery"
date: "2015-04-08"
categories: 
  - "openstack"
---

I've discussed the use of Pacemaker as a method to detect compute node failures and recover the VMs that were running there.  The implementation of this is ready for testing.  Details can be found in [this post to rdo-list](https://www.redhat.com/archives/rdo-list/2015-April/msg00008.html).

The post mentions one pending enhancement to Nova that would improve things further:

> Currently fence\_compute loops, waiting for nova to recognise that the failed host is down, before we make a host-evacuate call which triggers nova to restart the VMs on another host. The discussed nova API extensions will speed up recovery times by allowing fence\_compute to proactively push that information into nova instead.

The issue here is that the default backend for Nova's servicegroup API relies on the nova-compute service to periodically check in to the Nova database to indicate that it is still running.  The delay in the recovery process is caused by Nova waiting on a configured timeout since the last time the service checked in.  Pacemaker is going to know about the failure much sooner, so it would be helpful if there was an API to tell Nova "trust me, this node is gone".  [This proposed spec](https://review.openstack.org/#/c/169836/) intends to provide such an API.
