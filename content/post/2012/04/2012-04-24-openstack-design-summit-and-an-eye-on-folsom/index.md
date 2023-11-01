---
title: "OpenStack Design Summit and an Eye on Folsom"
date: "2012-04-24"
categories: 
  - "openstack"
---

I just spent a week in San Francisco at the OpenStack design summit and conference. It was quite an amazing week and I'm really looking forward to the Folsom development cycle.  You can find a notes from various sessions held at the design summit [on the OpenStack wiki](http://wiki.openstack.org/FolsomSummitEtherpads).

Essex was the first release that I contributed to.  One thing I did was add [Qpid](http://qpid.apache.org) support to both [Nova](https://github.com/openstack/nova/blob/master/nova/rpc/impl_qpid.py) and [Glance](https://github.com/openstack/glance/blob/master/glance/notifier/notify_qpid.py) as an alternative to using RabbitMQ.  Beyond that, I primarily worked on [vulnerability management](http://openstack.org/projects/openstack-security/) and [other bug fixing](https://launchpad.net/nova/essex/2012.1).  For Folsom, I'm planning on working on some more improvements involving the inter-service messaging layer, also referred to as the rpc API, in Nova.

## 1) Moving the rpc API to openstack-common

The rpc API in Nova is used for private communication between nova services. As an example, when a tenant requests that a new virtual machine instance be created (either via the EC2 API or the OpenStack compute REST API), the nova-api service sends a message to the nova-scheduler service via the rpc API.  The nova-scheduler service decides where the instance is going to live and then sends a message to that compute node's nova-compute service via the rpc API.

The other usage of the rpc API in Nova has been for notifications.  Notifications are asynchronous messages about events that happen within the system.  They can be used for monitoring and billing, among other things.  Strictly speaking, notifications aren't directly tied to rpc.  A Notifier is an abstraction, of which using rpc is one of the implementations.  Glance also has notifications, including a set of Notifier implementations.  The code was the same at one point but has diverged quite a bit since.

We would like to move the notifiers into openstack-common.  Moving rpc into openstack-common is a prerequisite for that, so I'm going to knock that part out.  I've already written a few patches in that direction.  Once the rpc API is in openstack-common, other projects will be able to make use of it.  There was discussion of Quantum using rpc at the design summit, so this will be needed for that, too.  Another benefit is that the [Heat](http://heat-api.org) project is using a copy of Nova's rpc API right now, but will be able to migrate over to using the version from openstack-common.

## 2) Versioning the rpc API interfaces

The [existing rpc API](https://github.com/openstack/nova/blob/master/nova/rpc/__init__.py) is pretty lightweight and seems to work quite well.  One limitation is that there is nothing provided to help with different versions of services talking to each other.  It may work ... or it may not.  If it doesn't, the failure you get could be something obvious, or it could be something really bad and bizarre where an operation fails half-way through, leaving things in a bad state.  I'd like to clean this up.

The end goal with this effort will be to make sure that as you upgrade from Essex to Folsom, any messages originating from an Essex service can and will be correctly processed by a Folsom service.  If that fails, then the failure should be immediate and obvious that a message was rejected due to a version issue.

## 3) Removing database access from nova-compute

This is by far the biggest effort of the 3 described here, and I won't be tackling this one alone.  I want to help drive it, though.  This discussion came up in the design summit session about enhancements to Nova security.  By removing direct database access from the nova-compute service, we can help reduce the potential impact if a compute node were to be compromised.  There are two main parts to this effort.

The first part is to make more efficient use of messaging by sending full objects through the rpc API instead of IDs. For example, there are many cases where the nova-api service gets an instance object from the database, does its thing, and then just sends the instance ID in rpc message.  On the other side it has to go pull that same object out of the database.  We have to go through and change all cases like this to include the full object in the message.  In addition to the security benefit, it should be more efficient, as well. This doesn't sound too complicated, and it isn't really, but it's quite a bit of work as there is a lot of code that needs to be changed. There will be some snags to deal with along the way, such as dealing with making sure all of the objects can be serialized properly.

Including full objects in messages is only part of the battle here.  The nova-compute service also does database updates.  We will have to come up with a new approach for this.  It will most likely end up being a new service of some sort, that handles state changes coming from compute nodes and makes the necessary database updates according to those state changes. I haven't fully thought through the solution to this part yet. For example, ensuring that this service maintains proper order of messages without turning into a bottleneck in the overall system will be important.

## Onward!

I'm sure I'll work on other things in Folsom, as well, but those are three that I have on my mind right now.  OpenStack is a great project and I'm excited to be a part of it!
