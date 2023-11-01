---
title: "OVN and OpenStack Status - 2015-04-21"
date: "2015-04-21"
categories: 
  - "openstack"
  - "ovn"
---

It has been a couple weeks since the [last OVN status update](http://blog.russellbryant.net/2015/04/08/ovn-and-openstack-integration-development-update/ "OVN and OpenStack Integration Development Update"). Here is a review of [what has happened](https://github.com/openvswitch/ovs/commits/ovn) since that time.

## ovn-nbd is now ovn-northd

Someone pointed out that the acronym "nbd" is used for ["Network Block Device"](https://github.com/qemu/qemu/blob/master/qemu-nbd.c) and may exist in the same deployment as OVN.  To avoid any possible confusion, we renamed [ovn-nbd to ovn-northd](https://github.com/openvswitch/ovs/commit/91ae206597a8944fe0d3a1d9ef1133f90f5e5c1c).

## ovn-controller now exists

`ovn-controller` is the daemon that runs on every hypervisor or gateway.  The initial version of this daemon [has been merged](https://github.com/openvswitch/ovs/commit/717c7fc508044d08210c686c1e8576c29a108f86).  The current version of `ovn-controller` performs two important functions.

First, `ovn-controller` populates the `Chassis` table of the [OVN\_Southbound database](https://github.com/openvswitch/ovs/blob/ovn/ovn/ovn-sb.xml).  Each row in the [Chassis table](https://github.com/openvswitch/ovs/blob/fa6aeaebdc033e204b88afb073732df7fc3ba587/ovn/ovn-sb.xml#L104) represents a hypervisor or gateway running `ovn-controller`.  It contains information that identifies the chassis and what encapsulation types it supports.  If you run `ovs-sandbox` with OVN support enabled, it will run the following commands to configure `ovn-controller`:
```
ovs-vsctl set open . external-ids:system-id=56b18105-5706-46ef-80c4-ff20979ab068
ovs-vsctl set open . external-ids:ovn-remote=unix:"$sandbox"/db.sock
ovs-vsctl set open . external-ids:ovn-encap-type=vxlan
ovs-vsctl set open . external-ids:ovn-encap-ip=127.0.0.1
ovs-vsctl add-br br-int

```
After setup is complete, we can check the `OVN_Southbound` table's contents and see the corresponding `Chassis` entry:
```
Chassis table
_uuid                                encaps                                 gateway_ports name                                  
------------------------------------ -------------------------------------- ------------- --------------------------------------
2852bf00-db63-4732-8b44-a3bc689ed1bc [e1c1f7fc-409d-4f74-923a-fc6de8409f82] {}            "56b18105-5706-46ef-80c4-ff20979ab068"

Encap table
_uuid                                ip          options type 
------------------------------------ ----------- ------- -----
e1c1f7fc-409d-4f74-923a-fc6de8409f82 "127.0.0.1" {}      vxlan

```
The other important task performed by the current version of `ovn-controller` is to monitor the local switch for ports being added that match up to logical ports created in OVN.  When a port is created on the local switch with an `iface-id` that matches the OVN logical port's name, `ovn-controller` will update the `Bindings` table to specify that the port exists on this chassis.  Once this is done, `ovn-northd` will report that the port is up to the `OVN_Northbound` database.
```
$ ovsdb-client dump OVN_Southbound
Bindings table
_uuid                                chassis                                logical_port                           mac parent_port tag
------------------------------------ -------------------------------------- -------------------------------------- --- ----------- ---
...
2dc299fa-835b-4e42-aa82-3d2da523b4d9 "81b0f716-c957-43cf-b34e-87ae193f617a" "d03aa502-0d76-4c1e-8877-43778088c55c" []  []          [] 
...

$ ovn-nbctl lport-get-up d03aa502-0d76-4c1e-8877-43778088c55c
up


```
The next steps for `ovn-controller` are to program the local switch to create tunnels and flows as appropriate based on the contents of the `OVN_Southbound` database.  This is currently being worked on.

## The Pipeline Table

The `OVN_Southbound` database has a table called [Pipeline](https://github.com/openvswitch/ovs/blob/fa6aeaebdc033e204b88afb073732df7fc3ba587/ovn/ovn-sb.xml#L205).  `ovn-northd` is responsible for translating the logical network elements defined in `OVN_Northbound` into entries in the `Pipeline` table of `OVN_Southbound`.  The first version of populating the `Pipeline` table [has been merged](https://github.com/openvswitch/ovs/commit/bd39395f46eda39854172507e56b61d2303758b7). One thing that is particularly interesting here is that `ovn-northd` defines _logical_ flows.  It does not have to figure out the detailed switch configuration for every chassis running `ovn-controller`.  `ovn-controller` is responsible for translating the logical flows into OpenFlow flows specific to the chassis.

The [OVN\_Southbound documentation](https://github.com/openvswitch/ovs/blob/fa6aeaebdc033e204b88afb073732df7fc3ba587/ovn/ovn-sb.xml#L205) has a good explanation of the contents of the `Pipeline` table.  If you're familiar with OpenFlow, the format will be very familiar.

As a simple example, let's just use `ovn-nbctl` to manually create a single logical switch that has 2 logical ports.
```
ovn-nbctl lswitch-add sw0
ovn-nbctl lport-add sw0 sw0-port1 
ovn-nbctl lport-add sw0 sw0-port2 
ovn-nbctl lport-set-macs sw0-port1 00:00:00:00:00:01
ovn-nbctl lport-set-macs sw0-port2 00:00:00:00:00:02

```
Now we can check out the resulting contents of the `Pipeline` table.  The output of `ovsdb-client` has been reordered to group the entries by `table_id` and `priority`. I've also cut off the `_uuid` column since it's not important for understanding here.
```
Pipeline table
match                          priority table_id actions                                                                 logical_datapath
------------------------------ -------- -------- ----------------------------------------------------------------------- ------------------------------------
"eth.src[40]"                  100      0        drop                                                                    843a9a4a-8afc-41e2-bea1-5fa58874e109
vlan.present                   100      0        drop                                                                    843a9a4a-8afc-41e2-bea1-5fa58874e109
"inport == \"sw0-port1\""      50       0        resubmit                                                                843a9a4a-8afc-41e2-bea1-5fa58874e109
"inport == \"sw0-port2\""      50       0        resubmit                                                                843a9a4a-8afc-41e2-bea1-5fa58874e109
"1"                            0        0        drop                                                                    843a9a4a-8afc-41e2-bea1-5fa58874e109

"eth.dst[40]"                  100      1        "outport = \"sw0-port2\"; resubmit; outport = \"sw0-port1\"; resubmit;" 843a9a4a-8afc-41e2-bea1-5fa58874e109
"eth.dst == 00:00:00:00:00:01" 50       1        "outport = \"sw0-port1\"; resubmit;"                                    843a9a4a-8afc-41e2-bea1-5fa58874e109
"eth.dst == 00:00:00:00:00:02" 50       1        "outport = \"sw0-port2\"; resubmit;"                                    843a9a4a-8afc-41e2-bea1-5fa58874e109

"1"                            0        2        resubmit                                                                843a9a4a-8afc-41e2-bea1-5fa58874e109

"outport == \"sw0-port1\""     50       3        "output(\"sw0-port1\")"                                                 843a9a4a-8afc-41e2-bea1-5fa58874e109
"outport == \"sw0-port2\""     50       3        "output(\"sw0-port2\")"                                                 843a9a4a-8afc-41e2-bea1-5fa58874e109

```
In table 0, we're dropping anything with a broadcast/multicast source MAC. We're also dropping anything with a logical VLAN tag, as that doesn't make sense. Next, if the packet comes from one of the ports connected to the logical switch, we will continue processing in table 1. Otherwise, we drop it.

In table 1, we will output the packet to all ports if the destination MAC is broadcast/multicast. Note that the output action to the source port is implicitly handled as a drop. Finally, we'll set the output variable based on destination MAC address and continue processing in table 2.

Table 2 does nothing but continue to table 3. In the ovn-northd code, table 2 is where entries for ACLs go. ovn-nbctl does not currently support adding ACLs. This table is where Neutron will program security groups, but that's not ready yet, either.

Table 3 handles sending the packet to the right output port based on the contents of the outport variable set back in table 1.

The `logical_datapath` column ties all of these rows together as implementing a single logical datapath, which in this case is an OVN logical switch.

There is one other item supported by `ovn-northd` that is not reflected in this example. The `OVN_Northbound` database has a [port\_security column for logical ports](https://github.com/openvswitch/ovs/blob/fa6aeaebdc033e204b88afb073732df7fc3ba587/ovn/ovn-nb.xml#L138). Its contents are defined as "A set of L2 (Ethernet) or L3 (IPv4 or IPv6) addresses or L2+L3 pairs from which the logical port is allowed to send packets and to which it is allowed to receive packets." If this were set here, table 0 would also handle ingress port security and table 3 would handle egress port security.

We will look at more detailed examples in future posts as both OVN and its Neutron integration progress further.

## Neutron Integration

There have also been [several changes](http://git.openstack.org/cgit/stackforge/networking-ovn/log/) to the Neutron integration for OVN in the last couple of weeks.  Since `ovn-northd` and `ovn-controller` are becoming more functional, the devstack integration [runs both of these daemons](http://git.openstack.org/cgit/stackforge/networking-ovn/commit/?id=7683ca800b1a5cc99904c37bae10faf030a3495d), along with `ovsdb-server` and `ovs-vswitchd`.  That means that as you create networks and ports via the Neutron API, they will be created in OVN and result in `Bindings` and `Pipeline` updates.

We now also have a devstack CI job that runs against every patch proposed to the OVN Neutron integration.  It installs and runs Neutron with OVN.  Devstack also creates some default networks.  We still have a bit more work to do in OVN before we can expand this to actually test network connectivity.

Also related to testing, Terry Wilson [submitted a patch to OVS](http://openvswitch.org/pipermail/dev/2015-April/053692.html) that will allow us to publish the OVS Python bindings to PyPI.  The patch has been merged and Terry will soon be publishing the code to PyPI.  This will allow us to install the library for unit test jobs.

The original Neutron ML2 driver implementation used `ovn-nbctl`.  It [has now been converted](http://git.openstack.org/cgit/stackforge/networking-ovn/commit/?id=25df316e3932e8e12158f4996c1109e646c72dd9) to use the Python ovsdb library, which should be much more efficient.  `neutron-server` will maintain an open connection to the `OVN_Northbound` database for all of its operations.

I've also been working on the necessary changes for creating a port in Neutron that is intended to be used by a container running inside a VM.  There is a [python-neutronclient change](https://review.openstack.org/#/c/174098/) and [two changes needed to networking-ovn](https://github.com/russellb/networking-ovn/commits/containers) that I'm still testing.

There are some edge cases where a resource can be created in Neutron but fail before we've created it in OVN.  Gal Sagie is [working on some code](https://review.openstack.org/#/c/174425/) to get them back in sync.

Gal Sagie also has a patch up for the first step toward security group support.  We have to [document how we will map](https://review.openstack.org/#/c/175149/) Neutron security groups to rules in the `OVN_Northbound ACL` table.

One piece of information that is communicated back up to the `OVN_Northbound` database by OVN is the `up` state of a logical port.  Terry Wilson is working on having our Neutron driver consume that so that we can emit a notification when a port that was created becomes ready for use.  This notification gets turned into a callback to Nova to tell it the VIF is ready for use so the corresponding VM can be started.
