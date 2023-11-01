---
title: "Juno Preview for OpenStack Compute (Nova)"
date: "2014-07-07"
categories: 
  - "openstack"
---

We're now well into the Juno release cycle. Here's my take on a preview of some of what you can expect in Juno for Nova.

# NFV

One area receiving a lot of focus this cycle is NFV. We've started an upstream [NFV sub-team for OpenStack](https://wiki.openstack.org/wiki/Teams/NFV) that is tracking and helping to drive requirements and development efforts in support of NFV use cases. If you're not familiar with NFV, here's a quick overview that was put together by the NFV sub-team:

> NFV stands for Network Functions Virtualization. It defines the replacement of usually stand alone appliances used for high and low level network functions, such as firewalls, network address translation, intrusion detection, caching, gateways, accelerators, etc, into virtual instance or set of virtual instances, which are called Virtual Network Functions (VNF). In other words, it could be seen as replacing some of the hardware network appliances with high-performance software taking advantage of high performance para-virtual devices, other acceleration mechanisms, and smart placement of instances. The origin of NFV comes from a working group from the European Telecommunications Standards Institute (ETSI) whose work is the basis of most current implementations. The main consumers of NFV are Service providers (telecommunication providers and the like) who are looking to accelerate the deployment of new network services, and to do that, need to eliminate the constraint of slow renewal cycle of hardware appliances, which do not autoscale and limit their innovation.
> 
> NFV support for OpenStack aims to provide the best possible infrastructure for such workloads to be deployed in, while respecting the design principles of a IaaS cloud. In order for VNF to perform correctly in a cloud world, the underlying infrastructure needs to provide a certain number of functionalities which range from scheduling to networking and from orchestration to monitoring capacities. This means that to correctly support NFV use cases in OpenStack, implementations may be required across most, if not all, main OpenStack projects, starting with Neutron and Nova.

The opportunities for OpenStack in the NFV space are huge. The work being tracked by the NFV sub-team spans more than just Nova, but here are some of the NFV related projects for Nova:

- [Virt driver guest vCPU topology configuration](http://git.openstack.org/cgit/openstack/nova-specs/tree/specs/juno/virt-driver-vcpu-topology.rst)
- [Virt driver guest NUMA node placement & topology](http://git.openstack.org/cgit/openstack/nova-specs/tree/specs/juno/virt-driver-numa-placement.rst)
- [I/O (PCIe) based NUMA scheduling](http://git.openstack.org/cgit/openstack/nova-specs/tree/specs/juno/input-output-based-numa-scheduling.rst)
- [Virt driver large page allocation for guest RAM](https://review.openstack.org/#/c/93653/)
- [Virt driver pinning guest vCPUs to host pCPUs](https://review.openstack.org/#/c/93652/)
- [PCI SR-IOV passthrough support for networking](http://git.openstack.org/cgit/openstack/nova-specs/tree/specs/juno/pci-passthrough-sriov.rst)

# Upgrades

The road to live upgrades has been a long one. Progress has been made over the last several releases. The Icehouse release was the first release that supported live upgrades in some form. From Havana to Icehouse, you can do a control plane upgrade with some API downtime without having to upgrade your compute nodes at the same time. You can roll through upgrading the compute nodes with the control plane already upgraded to Icehouse.

For Juno we are continuing to improve on this in several areas. Since Nova is a highly distributed system, one of the biggest requirements for doing this is versioning everything about the interactions between components. First we went through and versioned all of the interfaces between components. Next we have been versioning all of the data passed between components. This versioning of the data is part of what Nova Objects provide. Nova Objects are an internal implementation detail, but are critical to upgrade support. The entire code base had not been converted as of the Icehouse release. For Juno we continue to do conversions over to this new object model.

The other major improvement being looked at this release cycle is how we can reduce the downtime needed on the control plane by reducing how long it takes for database schema migrations to run. This is largely about developing new best practices about how migrations need to work going forward.

Finally, for the Icehouse release we added some basic testing of the live upgrade sceanario to the OpenStack CI system. This testing runs OpenStack using the previous release and then upgrades everything except the nova-compute service. At that point, everything should continue to work. One goal for the Juno cycle is to improve this testing to verify that we can also run an older instance of the nova-network service with an upgraded control plane. This is critical for deployments that use nova-network in multi-host mode. In that case, you have nova-network running on each compute node, so we need to support a mixed version environment for nova-network, as well as nova-compute.

# Scheduler

There's always a lot of interest in improving the way host scheduling works in Nova. In the Icehouse cycle we identified that we wanted to split the scheduler out into a new project (codenamed Gantt). Doing so requires decoupling Nova's scheduler as much as possible from the rest of Nova. This decoupling effort is the primary goal for the Juno cycle. Once the scheduler is independent of Nova, we can investigate ways to integrate other projects so that scheduling can use information that currently only lives in other projects such as Neutron or Cinder.

# Docker

The Docker driver for Nova was moved to Stackforge during the Icehouse development cycle. The primary reason was the lack of CI running for the driver. However, there were a number of feature gaps that made getting CI with tempest working as it needed to. Moving to stackforge gave an opportunity for the team working on this driver to iterate quicker and fill these gaps.

There has been a lot of progress on the Docker driver in the Juno cycle. Some of the feature gap work has resulting in improvements to Docker itself, which is really great to see. For example, Docker now supports pause and unpause, which is a feature of the Nova API that the Docker driver is now able to support. Another area that has seen some focus is Cinder support. To make this work, we have to be able to support exposing block devices to Docker containers at creation time, as well as later on after they are already running. There has been work on Docker itself in this area, which will eventually lead to support in the Nova Docker driver.

Finally, there has been ongoing work to get CI with tempest running. It's now running in OpenStack's CI infrastructure. The progress is great to see, but it also seems most likely that the driver will return to Nova in the K release cycle instead of Juno.

# Ironic

Nova introduced the baremetal driver [in the Grizzly release](https://blueprints.launchpad.net/nova/+spec/general-bare-metal-provisioning-framework).  This driver allows you to use Nova's API to do provisioning of bare metal instead of virtual machines.  There was immediately a lot of interest in this functionality for OpenStack.  Soon after this driver was introduced, it was decided that we should start a new project dedicated to bare metal management.  That project is Ironic.

Ironic has come a long way since then.  The project is currently incubated and could potentially graduate for the K release.  One of the major tasks in moving towards graduation is getting the Ironic driver for Nova merged.  The [spec has been approved](http://git.openstack.org/cgit/openstack/nova-specs/tree/specs/juno/add-ironic-driver.rst) and the code seems to be in good shape.  I'm very hopeful that we will have this step completed in the Juno release.

# Database Integration

OpenStack has been a long time user of the SQLAlchemy library for its integration with relational databases.  More recently, some OpenStack projects have begun using Alembic for managing database schema migrations.  [Michael Bayer](http://techspot.zzzeek.org/), author of SQLAlchemy and Alembic, recently joined Red Hat to help with OpenStack, as well as continue to maintain SQLAlchemy and Alembic.  He has been surveying OpenStack's current usage of SQLAlchemy and identifying areas where we can improve.  He has written up a [fascinating wiki page](https://wiki.openstack.org/wiki/Openstack_and_SQLAlchemy) with his findings.  I expect this to result in some very nice improvements to many OpenStack projects, including Nova.

# Other

There are many other features being worked on right now for Nova. The best place to get an idea of what's going on is to look at either the [list of approved design specs](http://git.openstack.org/cgit/openstack/nova-specs/tree/specs/juno) or the [list of specs under review](https://review.openstack.org/#/q/project:openstack/nova-specs+status:open,n,z).
