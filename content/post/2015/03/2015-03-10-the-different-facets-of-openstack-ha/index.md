---
title: "The Different Facets of OpenStack HA"
date: "2015-03-10"
categories: 
  - "openstack"
---

Last October, I wrote about [a particular aspect of providing HA](http://blog.russellbryant.net/2014/10/15/openstack-instance-ha-proposal/) for workloads running on OpenStack. The HA problem space for OpenStack is much more broad than what was addressed there. There has been a lot of work around HA for the OpenStack services themselves. The problems for OpenStack services seem to be pretty well understood. There are [reference architectures](https://openstack.redhat.com/HA_Architecture) with [detailed instructions](https://github.com/beekhof/osp-ha-deploy/blob/master/ha-openstack.md) available from vendors and distributions that have been integrated into deployment tools. The upstream OpenStack project also has an [HA guide](http://docs.openstack.org/high-availability-guide/content/) that covers this topic.

Another major area that has received less attention and effort is HA for compute workloads running on top of OpenStack. Requirements are very diverse in this area, so I'd like to provide some broader context around this topic and expand on my thoughts about useful work that can be done either in or around OpenStack to better support legacy workloads.

## Approaches to Recovery

My original thinking around all of this was that we should be building infrastructure that can automatically handle the recovery in response to failures.  It turns out that there is also a lot of demand for the ability to completely custom manage the recovery process.  This was particularly made clear to me while discussing availability requirements for [NFV](https://wiki.openstack.org/wiki/TelcoWorkingGroup#What_is_NFV.3F) workloads on OpenStack with participants in the [OPNFV project](https://www.opnfv.org/).  With that said, we need to think about all failure types in two ways.

1. **Completely Automated Recovery** - In this case, I envision a use wanting to enable an option that says "keep my VM running".  The rest would be automatically handled by the cloud infrastructure.
2. **Custom Recovery** - In this case, a set of applications has more strict requirements around failure detection (or even prediction) and availability.  They want to be notified of failures and be given the APIs necessary to implement their own recovery.

## Types of Failures

We can break down the types of failures into 3 major categories that require different work for an OpenStack deployment.

1. **Failure of Infrastructure** - This type of failure is when the hardware infrastructure support OpenStack workloads fails. A hardware failure on a given hypervisor is a prime example.
2. **Failure of the Guest Operating System** - In this case, the base operating system in the VM fails for some reason.  Imagine a kernel panic in your VM that causes the VM to stop running, even though the hypervisor is still operating just fine.
3. **Failure of the Application** - Something at the application layer may also fail.

Now let's look at each failure type and consider the work that could be done to support each approach to recovery.

### Infrastructure Failure

Failure of the infrastructure is what I wrote about last October.  I discussed the use of some recent enhancements to Pacemaker that would allow Pacemaker to monitor compute nodes.  This makes a lot of sense as Pacemaker is often already included in the HA architecture for the underlying OpenStack services.  There has since been work in cooperation with the Pacemaker team to build a proof-of-concept implementation of this approach.  Details for review and experimentation will be released soon.

The focus so far has been on completely automating the recovery process.  In particular that means issuing a nova evacuate API call for all of the instances that were running on the dead node.  However, this same architecture can be adapted to support the custom recovery approach.

Instead of fencing the node and issuing an evacuate, Pacemaker could optionally fence the node and emit a notification about the failure.  This notification could be put on the existing OpenStack notification message bus.  A deployment could choose to write something that consumes these notifications.  Another option could be to enhance Ceilometer to consume these notifications and turn it into an alarm that notifies an API consumer that the host that was running a VM has failed.

### Guest Operating System Failure

The libvirt/KVM driver in OpenStack already has great support for handling this type of failure and providing automated recovery.  In particular, you can set the "hw:watchdog\_action" property in either the extra\_specs field of a flavor or as an image property.  When this property is set, a watchdog device will be configured for the VM. If the value for this property is set to "reset", the VM will automatically be rebooted if the guest operating system crashes, triggering the watchdog.

To support the custom recovery case, we could add a new watchdog\_action called "notify".  In this case, Nova could simply emit a notification to the OpenStack notification message bus.  Again, a deployment could have a custom application that consumes these notifications or a service like Ceilometer could turn it into something consumable by a public REST API.

### Application Failure

In my opinion, providing an answer for application failures is the least important part of HA from an OpenStack perspective.  Dealing with this is not new for legacy applications.  It was necessary before running on OpenStack and there are a lot of solutions available.  For example, it's possible to run a virtual Pacemaker cluster inside a set of VMs just like you would on a physical deployment.  This has not always been the case, though.  When we looked at this in much earlier days of OpenStack (pre-Neutron), it was much more difficult to accomplish.  Neutron gives you enough control so that IP addresses can be predictable before creating the VMs, making it easy to get all of the addresses needed for Pacemaker configuration into the VM at the time it's created.

There is still some discussion about coming up with a cloud native approach to application monitoring and recovery handling.  Heat is often brought up in this context.  There was a [thread on the openstack-dev](http://lists.openstack.org/pipermail/openstack-dev/2014-December/053447.html) list this past December about this.  It seems there is potential there, but it's unclear if it's really high enough on anyone's priority list to work on.

## Pulling Things Together

A major point with all of this is that there are different layers failures can happen and there is not one solution for handling them all.  Breaking the problem space into pieces helps make it more manageable.  Once broken down, it seems clear to me that there is very reasonable work that can be done to enhance the ability of OpenStack to run traditional "pet" workloads.  Once these solutions are further along, it will be important to figure out how they integrate with deployment and management solutions, but let's start with just making it work.
