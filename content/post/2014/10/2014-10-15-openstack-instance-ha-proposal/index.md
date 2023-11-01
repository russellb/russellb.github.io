---
title: "OpenStack Instance HA Proposal"
date: "2014-10-15"
categories: 
  - "openstack"
---

In a perfect world, every workload that runs on OpenStack would be a cloud native application that is horizontally scalable and fault tolerant to anything that may cause a VM to go down.  However, the reality is quite different.  We continue to see a high demand for support of traditional workloads running on top of OpenStack and the HA expectations that come with them.

Traditional applications run on top of OpenStack just fine for the most part.  Some applications come up with availability requirements that a typical OpenStack deployment will not provide automatically.  If a hypervisor goes down, there is nothing in place that tries to rescue VMs that were running there.  There are some features in place that allow manual rescue, but it requires manual intervention from a cloud operator or an external orchestration tool.

This proposal discusses what it would take to provide automated detection of a failed hypervisor and the recovery of the VMs that were running there.  There are some differences to the solution based on what hypervisor you’re using.  I’m primarily concerned with libvirt/KVM, so I assume that for the rest of this post.  Except where libvirt is specifically mentioned, I think everything applies just as well to the use of the xenserver driver.

This topic is raised on a regular basis in the OpenStack community.  There has been pushback against putting this functionality directly in OpenStack.  Regardless of what components are used, I think we need to provide an answer to the question of how this problem should be approached.  I think this is quite achievable today using existing software.

# Scope

This proposal is specific to recovery from infrastructure failures.  There are other types of failures that can affect application availability.  The guest operating system or the application itself could fail.  Recovery from these types of failures is primarily left up to the application developer and/or deployer.

It’s worth noting that the libvirt/KVM driver in OpenStack does contain one feature related to guest operating system failure.  The [libvirt-watchdog](https://blueprints.launchpad.net/nova/+spec/libvirt-watchdog) blueprint was implemented in the Icehouse release of Nova.  This feature allows you to set the **hw\_watchdog\_action** property on either the image or flavor.  Valid values include **poweroff**, **reset**, **pause**, and **none**.  When this is enabled, libvirt will enable the **i6300esb** watchdog device for the guest and will perform the requested action if the watchdog is triggered.  This may be a helpful component of your strategy for recovery from guest failures.

# Architecture

A solution to this problem requires a few key components:

1. **Monitoring** - A system to detect that a hypervisor has failed.
2. **Fencing** \- A system to fence failed compute nodes.
3. **Recovery -** A system to orchestrate the rescue of VMs from the failed hypervisor.

## **Monitoring**

There are a two main requirements for the monitoring component of this solution.

1. Detect that a host has failed.
2. Trigger an automatic response to the failure (**Fencing** and **Recovery**).

It’s often suggested that the solution for this problem should be a part of OpenStack.  Many people have suggested that all of this functionality should be built into Nova.  The problem with putting it in Nova is that it assumes that Nova has proper visibility into the health of the infrastructure that Nova itself is running on.  There is a servicegroup API that does very basic group membership.  In particular, it keeps track of active compute nodes.  However, at best this can only tell you that the nova-compute service is not currently checking in.  There are several potential causes for this that would still leave the guest VMs running just fine.  Getting proper infrastructure visibility into Nova is really a layering violation.  Regardless, it would be a significant scope increase for Nova, and I really don’t expect the Nova team to agree to it.

It has also been proposed that this functionality be added to Heat.  The most fundamental problem with that is that a cloud user should not be required to use Heat to get their VM restarted if something fails.  There have been other proposals to use other (potentially new) OpenStack components for this.  I don’t like that for many of the same reasons I don’t think it should be in Nova.  I think it’s a job for the infrastructure supporting the OpenStack deployment, not OpenStack itself.

Instead of trying to figure out which OpenStack component to put it in, I think we should consider this a feature provided by the infrastructure supporting an OpenStack deployment.  Many OpenStack deployments already use [Pacemaker](http://clusterlabs.org/) to provide HA for portions of the deployment.  Historically, there have been scaling limits in the cluster stack that made Pacemaker not an option for use with compute nodes since there’s far too many of them.  This limitation is actually in Corosync and not Pacemaker itself.  More recently, Pacemaker has added a new feature called [pacemaker\_remote](http://clusterlabs.org/doc/en-US/Pacemaker/1.1/html-single/Pacemaker_Remote/), which allows a host to be a part of a Pacemaker cluster, without having to be a part of a Corosync cluster.  It seems like this may be a suitable solution for OpenStack compute nodes.

Many OpenStack deployments may already be using a monitoring solution like Nagios for their compute nodes.  That seems reasonable, as well.

## **Fencing**

To recap, fencing is an operation that completely isolates a failed node.  It could be IPMI based where it ensures that the failed node is powered off, for example.  Fencing is important for several reasons.  There are many ways a node can fail, and we must be sure that the node is completely gone before starting the same VM somewhere else.  We don’t want the same VM running twice.  That is certainly not what a user expects.  Worse, since an OpenStack deployment doing automatic evacuation is probably using shared storage, running the same VM twice can result in data corruption, as two VMs will be trying to use the same disks.  Another problem would be having the same IPs on the network twice.

A huge benefit of using Pacemaker for this is that it has built-in integration with fencing, since it’s a key component of any proper HA solution.  If you went with Nagios, fencing integration may be left up to you to figure out.

## **Recovery**

Once a failure has been detected and the compute node has been fenced, the evacuation needs to be triggered.  To recap, evacuation is restarting an instance that was running on a failed host by moving it to another host.  Nova provides an API call to evacuate a single instance.  For this to work properly, instance disks should be on shared storage.  Alternatively, they could all be booted from Cinder volumes.  Interestingly, the evacuate API will still run even without either of these things.  The result is just a new VM from the same base image but without any data from the old one.  The only benefit then is that you get a VM back up and running under the same instance UUID.

A common use case with evacuation is “evacuate all instances from a given host”.  Since this is common enough, it was scripted as a feature in the novaclient library.  So, the monitoring tool can trigger this feature provided by novaclient.

If you want this functionality for all VMs in your OpenStack deployment, then we’re in good shape.  Many people have made the additional request that users should be able to request this behavior on a per-instance basis.  This does indeed seem reasonable, but poses an additional question.  How should we let a user indicate to the OpenStack deployment that it would like its instance automatically recovered?

The typical knobs used are image properties and flavor extra-specs.  That would certainly work, but it doesn’t seem quite flexible enough to me.  I don’t think a user should have to create a new image to mark it as “keep this running”.  Flavor extra-specs are fine if you want this for all VMs of a particular flavor or class of flavors.  In either case, the novaclient “evacuate a host” feature would have to be updated to optionally support it.

Another potential solution to this is by using a special tag that would be specified by the user.  There is a [proposal up for review](https://review.openstack.org/#/c/127281/) right now to provide a simple tagging API for instances in Nova.  For this discussion, let’s say the tag would be **automatic-recovery**.  We could also update the novaclient feature we’re using with support for “evacuate all instances on this host that have a given tag”.  The monitoring tool would trigger this feature and ask novaclient to evacuate a host of all VMs that were tagged with **automatic-recovery**.

# Conclusions and Next Steps

Instance HA is clearly something that many deployments would like to provide.  I believe that this could be put together for a deployment today using existing software, Pacemaker in particular.  A next step here is to provide detailed information on how to set this up and also do some testing.

I expect that some people might say, “but I’m already using system Foo (Nagios or whatever) for monitoring my compute nodes”.  You could go this route, as well.  I’m not sure about fencing integration with something like Nagios.  If you skip the use of fencing in this solution, you get to keep the pieces when it breaks.  Aside from that, your monitoring system could trigger the evacuation functionality of novaclient just like Pacemaker would.

Some really nice future development around this would be integration into an OpenStack management UI.  I’d like to have a dashboard of my deployment that shows me any failures that have occurred and what responses have been triggered.  This should be possible since pcsd offers a REST API (WIP) that could export this information.

Lastly, it’s worth thinking about this problem specifically in the context of TripleO.  If you’re deploying OpenStack with OpenStack, should the solution be different?  In that world, all of your baremetal nodes are OpenStack resources via Ironic.  Ceilometer could be used to monitor the status of those resources.  At that point, OpenStack itself does have enough information about the supporting infrastructure to perform this functionality.  Then again, instead of trying to reinvent all of this in OpenStack, we could just use the more general Pacemaker based solution there, as well.
