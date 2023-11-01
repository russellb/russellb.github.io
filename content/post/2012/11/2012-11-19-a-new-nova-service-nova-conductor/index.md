---
title: "A new Nova service: nova-conductor"
date: "2012-11-19"
categories: 
  - "openstack"
---

The Grizzly release of OpenStack Nova will have a new service, nova-conductor. The service was discussed on the [openstack-dev list](http://lists.openstack.org/pipermail/openstack-dev/2012-November/002573.html) and it was [merged today](https://review.openstack.org/#/c/16135/11). There is currently a configuration option that can be turned on to make it optional, but it is possible that by the time Grizzly is released, this service will be required.

One of the efforts that started during Folsom development and is scheduled to be completed in Grizzly is [no-db-compute](https://blueprints.launchpad.net/nova/+spec/no-db-compute). In short, this effort is to remove direct database access from the nova-compute service. There are two main reasons we are doing this. Compute nodes are the least trusted part of a nova deployment, so removing direct database access is a step toward reducing the potential impact of a compromised compute node. The other benefit of no-db-compute is for upgrades. Direct database access complicates the ability to do live rolling upgrades. We're working toward eventually making that possible, and this is a part of that.

All of the nova services use a messaging system (usually AMQP based) to communicate with each other. Many of the database accesses in nova-compute can be (and have been) removed by just sending more data in the initial message sent to nova-compute. However, that doesn't apply to everything. That's where the new service, nova-conductor, comes in.

The nova-conductor service is key to completing no-db-compute. Conceptually, it implements a new layer on top of nova-compute. It should \*not\* be deployed on compute nodes, or else the security benefits of removing database access from nova-compute will be negated. Just like other nova services such as nova-api or nova-scheduler, it can be scaled horizontally. You can run multiple instances of nova-conductor on different machines as needed for scaling purposes.

The methods exposed by nova-conductor will initially be relatively simple methods used by nova-compute to offload its database operations. Places where nova-compute previously did database access will now be talking to nova-conductor. However, we have plans in the medium to long term to move more and more of what is currently in nova-compute up to the nova-conductor layer. The compute service will start to look like a less intelligent slave service to nova-conductor. The conductor service will implement long running complex operations, ensuring forward progress and graceful error handling. This will be especially beneficial for operations that cross multiple compute nodes, such as migrations or resizes.

If you have any comments, questions, or suggestions for nova-conductor or the no-db-compute effort in general, please feel free to bring it up on the [openstack-dev list](http://lists.openstack.org/cgi-bin/mailman/listinfo/openstack-dev).
