---
title: "OVN and OpenStack Integration Development Update"
date: "2015-04-08"
categories: 
  - "openstack"
  - "ovn"
---

The Open vSwitch project [announced the OVN effort](http://networkheresy.com/2015/01/13/ovn-bringing-native-virtual-networking-to-ovs/) back in January.  After OVN was announced, I got very interested in its potential.  OVN is by no means tied to OpenStack, but the primary reason I'm interested is I see it as a promising open source backend for OpenStack Neutron.  To put it into context with existing Neutron code, it would replace the OVS agent in Neutron in the short term.  It would eventually also replace the L3 and DHCP agents once OVN gains the equivalent functionality.

Implementation has been coming along well in the last month, so I wanted to share an overview of what we have so far.  We're aiming to have a working implementation of L2 connectivity by the [OpenStack Vancouver Summit](https://www.openstack.org/summit/vancouver-2015/) next month.

## Design

The [initial design documentation](https://github.com/openvswitch/ovs/commit/fe36184b7a637839329e2b567659081fbe74ae5f) was merged at the end of February.  Here are the rendered versions of those docs: [ovn-architecture](http://benpfaff.org/~blp/dist-docs/ovn-architecture.7.html), [ovn-nb schema](http://benpfaff.org/~blp/dist-docs/ovn-nb.5.html), [ovn schema](http://benpfaff.org/~blp/dist-docs/ovn.5.html).

This initial design allows hooking up VMs or containers to OVN managed virtual networks.  There was an [update to the design](https://github.com/openvswitch/ovs/commit/9fb4636f6c587060713ea0abf60ed6bcbe4f11f4) merged that addresses the use case of running containers inside of VMs.  It seems like most existing work just creates another layer of overlay networks for containers.  What's interesting about this proposal is that it allows you to connect those containers directly to the OVN managed virtual networks.  In the OpenStack world, that means you could have your containers hooked up directly to virtual networks managed by Neutron.  Further, the container hosting VM and all of its containers do not have to be connected to the same network and this works without having to create an extra layer of overlay networks.

## OVN Implementation

For most of my OVN development and testing, I've been working straight from the ovs git tree. Building it is something like:
```
$ git clone http://github.com/openvswitch/ovs.git
$ cd ovs
```
Switch to the `ovn` branch, as that's where OVN development is happening for now:
```
$ git checkout ovn
```
You'll need automake, autoconf, libtool, make, patch, and gcc or clang installed, at least. For detailed instructions on building ovs, see [INSTALL.md](https://github.com/openvswitch/ovs/blob/master/INSTALL.md) in the ovs git tree.
```
$ ./boot.sh
$ ./configure
$ make
```
OVS includes a script called `ovs-sandbox` that I find **very** helpful for development. It sets up a dummy ovs environment that you can run the tools against, but it doesn't actually process real traffic. You can send some fake packets through to see how they would be processed if needed. I've been adding OVN support to `ovs-sandbox` along the way.

Here's a demonstration of `ovs-sandbox` with what is implemented in OVN so far.  Start by running `ovs-sandbox` with OVN support turned on:
```
$ make sandbox SANDBOXFLAGS="-o"
```
You'll get output like this:
```
----------------------------------------------------------------------
You are running in a dummy Open vSwitch environment. You can use
ovs-vsctl, ovs-ofctl, ovs-appctl, and other tools to work with the
dummy switch.

Log files, pidfiles, and the configuration database are in the
"sandbox" subdirectory.

Exit the shell to kill the running daemons.
```
Now everything is running:
```
$ ps ax | grep ov[sn]
 ...
 ... ovsdb-server --detach --no-chdir --pidfile -vconsole:off --log-file --remote=punix:/home/rbryant/src/ovs/tutorial/sandbox/db.sock ovn.db ovnnb.db conf.db
 ... ovs-vswitchd --detach --no-chdir --pidfile -vconsole:off --log-file --enable-dummy=override -vvconn -vnetdev_dummy
 ... ovn-nbd --detach --no-chdir --pidfile -vconsole:off --log-file
```
Note the `ovn-nbd` daemon. Soon there will also be an `ovn-controller` daemon running. Also note that `ovsdb-server` is serving up 3 databases (`ovn.db`, `ovnnb.db`, and `conf.db`).

You can run `ovn-nbctl` to create resources via the OVN public interface (the `OVN_Northbound` database). So, for example:
```
$ ovn-nbctl lswitch-add sw0
$ ovn-nbctl lswitch-add sw1
$ ovn-nbctl lswitch-list
4956f6b4-a1ba-49aa-86a6-134b9cfdfdf6 (sw1)
52858b33-995f-43fa-a1cf-445f16d2ab09 (sw0)
$ ovn-nbctl lport-add sw0-port0 sw0
$ ovn-nbctl lport-add sw0-port1 sw0
$ ovn-nbctl lport-list sw0
d4d78dc5-166d-4457-8bb0-1f6ed5f1ed91 (sw0-port1)
c2114eaa-2f75-443f-b23e-6dda664a979b (sw0-port0)
```
One of the things that `ovn-nbd` does is create entries in the `Bindings` table of the OVN database when logical ports are added to the `OVN_Northbound` database. The `Bindings` table is used to keep track of which hypervisor a port exists on after VIFs get created and plugged into the local ovs switch. After the commands above, there should be 2 entries in the `Bindings` table. We can dump the `OVN` db and see that they are there:
```
$ ovsdb-client dump OVN
Bindings table
_uuid chassis logical_port mac parent_port tag
------------------------------------ ------- ------------ --- ----------- ---
997e0c14-2fba-499d-b077-26ddfc87e935 "" "sw0-port0" [] [] []
f7b61ef1-01d5-42ab-b08e-176bf6f3eb4b "" "sw0-port1" [] [] []
```
Note that the `chassis` column is empty, meaning that the port hasn't been placed on a hypervisor yet.

We can also see that the state of the port is still down in the `OVN_Northbound` database since it hasn't been created on a hypervisor yet.
```
$ ovn-nbctl lport-get-up sw0-port0
down
```
One of the tasks of `ovn-controller` running on each hypervisor is to monitor the local switch and detect when a new port on the local switch corresponds with an OVN logical port. When that occurs, `ovn-controller` will update the `chassis` column. For now, we can simulate that with a manual ovsdb transaction:
```
$ ovsdb-client transact '["OVN",{"op":"update","table":"Bindings","where":[["_uuid","==",["uuid","997e0c14-2fba-499d-b077-26ddfc87e935"]]],"row":{"chassis":"hostname"}}]'
[{"count":1}]
```
```
$ ovsdb-client dump OVN
Bindings table
_uuid chassis logical_port mac parent_port tag
------------------------------------ -------- ------------ --- ----------- ---
f7b61ef1-01d5-42ab-b08e-176bf6f3eb4b "" "sw0-port1" [] [] []
997e0c14-2fba-499d-b077-26ddfc87e935 hostname "sw0-port0" [] [] []
```
Now that the `chassis` column has been populated, `ovn-nbd` should notice and set the port state to `up` in the `OVN_Northbound` db.
```
$ ovn-nbctl lport-get-up sw0-port0
up
```

## OpenStack Integration

Like with most OpenStack projects, you can try out the Neutron support for OVN using [devstack](http://git.openstack.org/cgit/openstack-dev/devstack/tree/README.md).  Instructions for using the OVN devstack plugin are in the [networking-ovn git repo](http://git.openstack.org/cgit/stackforge/networking-ovn/tree/devstack/README.rst).

You start by cloning both devstack and networking-ovn.
```
$ git clone http://git.openstack.org/openstack-dev/devstack.git
$ git clone http://git.openstack.org/openstack/networking-ovn.git
```
If you don't have any devstack configuration, you can use a sample `local.conf` from the networking-ovn repo:
```
$ cd devstack
$ cp ../networking-ovn/devstack/local.conf.sample local.conf
```
If you're new to using devstack, it is best if you use a throwaway VM for this.  You will also need to run devstack with a sudo enabled user.  Once your configuration that enables OVN support is in place, run devstack:
```
$ ./stack.sh
```
In my case, I'm running this on Fedora 21.  It has also been tested on Ubuntu. Once devstack finishes running successfully, you should get output that looks like this:
```
This is your host ip: 192.168.122.31
Keystone is serving at http://192.168.122.31:5000/
The default users are: admin and demo
The password: password
2015-04-08 14:31:10.242 | stack.sh completed in 165 seconds.
```
One bit of environment initialization that devstack does is create some initial Neutron networks.  You can see them using the `neutron` command, which talks to the Neutron REST API.
```
$ . openrc
$ neutron net-list
+--------------------------------------+---------+--------------------------------------------------+
| id | name | subnets |
+--------------------------------------+---------+--------------------------------------------------+
| a28b651e-5cb9-481b-9f9b-d5d57e55c6d0 | public | df0aee67-166c-4ad4-890c-bbf5d02ca3cf |
| 2637f01e-f41e-4d1b-865f-195253027031 | private | eac6621f-e8cc-4c94-84bf-e73dab610018 10.0.0.0/24 |
+--------------------------------------+---------+--------------------------------------------------+
```
Since OVN is the configured backend, we can use the `ovn-nbctl` utility to verify that these networks were created in OVN.
```
$ ovn-nbctl lswitch-list
480235d0-d1a5-43a9-821b-d32e109445fd (neutron-2637f01e-f41e-4d1b-865f-195253027031)
a60a2c16-cea7-4bdc-8082-b47745d016b3 (neutron-a28b651e-5cb9-481b-9f9b-d5d57e55c6d0)
```
```
$ ovn-nbctl lswitch-get-external-id 480235d0-d1a5-43a9-821b-d32e109445fd
neutron:network_name=private
```
```
$ ovn-nbctl lswitch-get-external-id a60a2c16-cea7-4bdc-8082-b47745d016b3
neutron:network_name=public
```
We can also create ports using the Neutron API and verify that they get created in OVN. To do that, we first create a port in Neutron:
```
$ neutron port-create private
Created a new port:
+-----------------------+---------------------------------------------------------------------------------+
| Field | Value |
+-----------------------+---------------------------------------------------------------------------------+
| admin_state_up | True |
| allowed_address_pairs | |
| binding:vnic_type | normal |
| device_id | |
| device_owner | |
| fixed_ips | {"subnet_id": "eac6621f-e8cc-4c94-84bf-e73dab610018", "ip_address": "10.0.0.3"} |
| id | ff07588c-4b11-4ec8-b7c5-1be64fc0ebac |
| mac_address | fa:16:3e:23:bd:f6 |
| name | |
| network_id | 2637f01e-f41e-4d1b-865f-195253027031 |
| security_groups | ab539a1c-c3d8-49f7-9ad1-3a8b451bce91 |
| status | DOWN |
| tenant_id | 64f29642350d4c978cf03a4917a35999 |
+-----------------------+---------------------------------------------------------------------------------+
```
Then we can list the logical ports in OVN for the logical switch associated with the Neutron network named private.  The output is the OVN UUID for the port followed by the port name in parentheses.  Neutron sets the port name equal to the UUID of the Neutron port.
```
$ ovn-nbctl lswitch-get-external-id 480235d0-d1a5-43a9-821b-d32e109445fd
neutron:network_name=private
$ ovn-nbctl lport-list 480235d0-d1a5-43a9-821b-d32e109445fd
...
fe959cfa-fd20-4129-9669-67af1fa6bbf7 (ff07588c-4b11-4ec8-b7c5-1be64fc0ebac)
```
We can also see that the port is `down` since it has not yet been plugged in to the local ovs switch on a hypervisor:
```
$ ovn-nbctl lport-get-up fe959cfa-fd20-4129-9669-67af1fa6bbf7
down
```

## Ongoing Work

All OVN development discussion, patch submission, and patch review happens on the [ovs-dev mailing list](http://mail.openvswitch.org/mailman/listinfo/dev).  Development is currently happening in [the ovn branch](https://github.com/openvswitch/ovs/commits/ovn) until things are further along.  Discussion about the OpenStack integration happens on the [openstack-dev mailing list](http://lists.openstack.org/cgi-bin/mailman/listinfo/openstack-dev), while patch submission and review happens in [OpenStack's gerrit](https://review.openstack.org/#/q/project:stackforge/networking-ovn+status:open,n,z).

As mentioned earlier, the `ovn-controller` daemon is not yet running in this development environment.  That will change shortly as Justin Pettit [posted it for review](http://openvswitch.org/pipermail/dev/2015-April/053408.html) earlier this week.

As you might have noticed, there's a lot of infrastructure in place, but the actual flows and tunnels necessary to implement these virtual networks are not yet in place.  There's been a lot of work in preparation for that, though.  Ben Pfaff has had [a patch series up for review](http://openvswitch.org/pipermail/dev/2015-March/053119.html) for expression matching needed for OVN.  It probably should have been merged by now, but the reviews have been a little slow.  (That's my guilt talking.)  Ben has also started working on making `ovn-nbd` populate the Pipeline table of the OVN database.

Finally, the proposed OVN design introduces some new demands on `ovsdb-server`.  In particular, there will easily be hundreds of instances of `ovn-controller` connected to `ovsdb-server`.  Andy Zhou has been doing some [very nice work around increasing performance](http://openvswitch.org/pipermail/dev/2015-March/052623.html) in anticipation of these new demands.
