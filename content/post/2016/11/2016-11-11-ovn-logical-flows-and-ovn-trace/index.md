---
title: "OVN Logical Flows and ovn-trace"
date: "2016-11-11"
categories: 
  - "ovn"
  - "ovs"
---

One of the most satisfying feelings when working on new software is when you settle on a really great abstraction. When this goes well, things just fall into place. The design is easy to understand and modifying the system is an easy, pleasant experience.

This is how I've felt as I learned about the original proposed design for OVN and then contributed to OVN development over the last year and a half. In particular, I've been incredibly happy with the Logical Flows abstraction.

In this post I'll explain what Logical Flows are and how to use ovn-trace to understand them. I will also provide some examples where this abstraction made adding features far easier than I would have expected.

## But First, OpenFlow Basics

Before getting to Logical Flows, it is helpful to have a general understanding of OpenFlow. OpenFlow is the protocol used to program the packet processing pipeline of Open vSwitch. It lets you define a series of tables with rules (flows) that contain a priority, match, and a set of actions. For each table, the highest priority (larger number is higher priority) flow that matches is executed.

Let's imagine a trivial virtual switch with two ports, port1 and port2.
```
                        +--------+
            (1)         |        |          (2)
           port1 -------| br-int |-------- port2
                        |        |
                        +--------+

```
We can create a bridge with 2 ports using the following commands:
```
$ ovs-vsctl add-br br-int
$ ovs-vsctl add-port br-int port1
$ ovs-vsctl add-port br-int port2

$ ovs-vsctl show
3b1995d8-9683-45db-8929-36c62abdbd31
    Bridge br-int
        Port "port1"
            Interface "port1"
        Port br-int
            Interface br-int
                type: internal
        Port "port2"
            Interface "port2"

```
A trivial example is to define a single table where we forward all packets from port1 to port2, and all packets from port2 to port1.
```
    Table  Priority  Match      Actions
    ----- -------- ---------- -------
    0      0         in_port=1  output:2
    0      0         in_port=2  output:1

```
We can program this pipeline in OVS using the ovs-ofctl command:
```
$ ovs-ofctl del-flows br-int
$ ovs-ofctl add-flow br-int
$ ovs-ofctl add-flow br-int "table=0, priority=0, in_port=1,actions=output:2"
$ ovs-ofctl add-flow br-int "table=0, priority=0, in_port=2,actions=output:1"

$ ovs-ofctl dump-flows br-int
NXST_FLOW reply (xid=0x4):
  cookie=0x0, duration=9.679s, table=0, n_packets=0, n_bytes=0, idle_age=9, priority=0,in_port=1 actions=output:2
  cookie=0x0, duration=2.287s, table=0, n_packets=0, n_bytes=0, idle_age=2, priority=0,in_port=2 actions=output:1

```
We can extend this example to add a second table and demonstrate the use of different priorities. Let's only allow packets from port1 if the source MAC address is 00:00:00:00:01 and only allow packets from port2 with a source MAC address of 00:00:00:00:00:02 (basic source port security). We'll use table 0 to implement port security and then use table 1 to decide the packet's destination.

(Yes, this could be done in a single table, but then it wouldn't be demonstrating tables and priorities, which is the main point here.)
```
    Table  Priority  Match                               Actions
    ----- -------- ----------------------------------- ------------
    0      10        in_port=1,dl_src=00:00:00:00:00:01  resubmit(,1)
    0      10        in_port=2,dl_src=00:00:00:00:00:02  resubmit(,1)
    0      0                                             drop
    1      0         in_port=1                           output:2
    1      0         in_port=2                           output:1

```
Again, we can program this pipeline using the ovs-ofctl command line utility.
```
$ ovs-ofctl del-flows br-int
$ ovs-ofctl add-flow br-int "table=0, priority=10, in_port=1,dl_src=00:00:00:00:00:01,actions=resubmit(,1)"
$ ovs-ofctl add-flow br-int "table=0, priority=10, in_port=2,dl_src=00:00:00:00:00:02,actions=resubmit(,1)"
$ ovs-ofctl add-flow br-int "table=0, priority=0, actions=drop"
$ ovs-ofctl add-flow br-int "table=1, priority=0, in_port=1,actions=output:2"
$ ovs-ofctl add-flow br-int "table=1, priority=0, in_port=2,actions=output:1"

$ ovs-ofctl dump-flows br-int
NXST_FLOW reply (xid=0x4):
 cookie=0x0, duration=72.132s, table=0, n_packets=0, n_bytes=0, idle_age=72, priority=10,in_port=1,dl_src=00:00:00:00:00:01 actions=resubmit(,1)
 cookie=0x0, duration=60.565s, table=0, n_packets=0, n_bytes=0, idle_age=60, priority=10,in_port=2,dl_src=00:00:00:00:00:02 actions=resubmit(,1)
 cookie=0x0, duration=28.127s, table=0, n_packets=0, n_bytes=0, idle_age=28, priority=0 actions=drop
 cookie=0x0, duration=13.887s, table=1, n_packets=0, n_bytes=0, idle_age=13, priority=0,in_port=1 actions=output:2
 cookie=0x0, duration=4.023s, table=1, n_packets=0, n_bytes=0, idle_age=4, priority=0,in_port=2 actions=output:1

```
Open vSwitch also provides a mechanism to trace a sample packet through a configured pipeline. Here we will trace a packet from port1 with an expected source MAC address. The output of this trace shows that the packet is resubmitted to table 1 and is then output to port 2.   The output is a bit verbose.  Just look at the "Rule" and "OpenFlow actions" lines to see which flows were executed.
```
$ ovs-appctl ofproto/trace br-int in_port=1,dl_src=00:00:00:00:00:01,dl_dst=00:00:00:00:00:02 -generate
Bridge: br-int
Flow: in_port=1,vlan_tci=0x0000,dl_src=00:00:00:00:00:01,dl_dst=00:00:00:00:00:02,dl_type=0x0000

Rule: table=0 cookie=0 priority=10,in_port=1,dl_src=00:00:00:00:00:01
OpenFlow actions=resubmit(,1)

    Resubmitted flow: in_port=1,vlan_tci=0x0000,dl_src=00:00:00:00:00:01,dl_dst=00:00:00:00:00:02,dl_type=0x0000
    Resubmitted regs: reg0=0x0 reg1=0x0 reg2=0x0 reg3=0x0 reg4=0x0 reg5=0x0 reg6=0x0 reg7=0x0 reg8=0x0 reg9=0x0 reg10=0x0 reg11=0x0 reg12=0x0 reg13=0x0 reg14=0x0 reg15=0x0
    Resubmitted  odp: drop
    Resubmitted megaflow: recirc_id=0,in_port=1,dl_src=00:00:00:00:00:01,dl_type=0x0000
    Rule: table=1 cookie=0 priority=0,in_port=1
    OpenFlow actions=output:2

Final flow: unchanged
Megaflow: recirc_id=0,in_port=1,dl_src=00:00:00:00:00:01,dl_type=0x0000
Datapath actions: 3

```
OpenFlow can be used to build up much more complex pipelines, as well. See the [ovs-ofctl(8) man page](http://openvswitch.org/support/dist-docs/ovs-ofctl.8.html) for a lot more detail.

## OVN Logical Flows

The previous section recapped the basics of OpenFlow. It showed how OpenFlow can be used to build packet processing pipelines of a single switch. Manually programming these pipelines on one host, much less hundreds or thousands of hosts, can be tedious. That's where an SDN controller that programs flows across many switches to accomplish a task is helpful. That's the role that OVN plays for the Open vSwitch project. OVN does all of the OpenFlow programming necessary to implement the network topologies and security policies you define using its high level configuration interface.

How does OVN determine the flows required on each host? The central abstraction to solving this problem in OVN is Logical Flows. Logical Flows are conceptually similar to OpenFlow in that they are made up of tables of flows with a priority, match, and actions. The major difference is that logical flows describe the detailed behavior of an entire network that can span any number of hosts. It provides us with separation between defining detailed network behavior and having to worry about the actual physical layout of the environment (how many hosts exist and which hosts ports reside on).

OVN centrally programs networks in logical flows. These logical flows are distributed throughout the whole environment to ovn-controller running on each host. ovn-controller then knows how to compile logical flows into OpenFlow using the current state of the physical environment (what ports reside locally, and how to reach other hosts).

Let's create an example OVN configuration similar to the one in the section on OpenFlow basics. We will create a single OVN logical switch with two logical ports.
```
$ ovn-nbctl ls-add sw0

$ ovn-nbctl lsp-add sw0 sw0-port1
$ ovn-nbctl lsp-set-addresses sw0-port1 00:00:00:00:00:01
$ ovn-nbctl lsp-set-port-security sw0-port1 00:00:00:00:00:01

$ ovn-nbctl lsp-add sw0 sw0-port2
$ ovn-nbctl lsp-set-addresses sw0-port2 00:00:00:00:00:02
$ ovn-nbctl lsp-set-port-security sw0-port2 00:00:00:00:00:02

$ ovn-nbctl show sw0
    switch 48d5f699-7ffe-4627-a369-2fc905e44b32 (sw0)
        port sw0-port1
            addresses: ["00:00:00:00:00:01"]
        port sw0-port2
            addresses: ["00:00:00:00:00:02"]

```
OVN defines the logical switch, sw0, using two pipelines: an ingress pipeline and an egress pipeline. When a packet enters the network, the ingress pipeline is executed on the host where the packet originated. If the destination is on the same host, the egress pipeline will be executed as well.
```
sw0-port1 and sw0-port2 on the same host:

    +--------------------------------------------------------------------------+
    |                                                                          |
    |                               Host A                                     |
    |                                                                          |
    |   +---------+                                              +---------+   |
    |   |sw0-port1| --> ingress pipeline --> egress pipeline --> |sw0-port2|   |
    |   +---------+                                              +---------+   |
    |                                                                          |
    +--------------------------------------------------------------------------+

```
If the destination is remote, the packet will be sent over a tunnel before executing the egress pipeline on the remote host.
```
sw0-port1 and sw0-port2 on separate hosts:

    +--------------------------------------+
    |                                      |
    |             Host A                   |
    |                                      |
    |   +---------+                        |
    |   |sw0-port1| --> ingress pipeline   |
    |   +---------+           ||           |
    |                         ||           |
    +-------------------------||-----------+
                              ||
                              \/
                         geneve tunnel
                              ||
                              ||
    +-------------------------||-----------+
    |                         ||           |
    |             Host B      ||           |
    |                         ||           |
    |   +---------+           \/           |
    |   |sw0-port2| < -- egress pipeline   |
    |   +---------+                        |
    |                                      |
    +--------------------------------------+

```
You can use the "ovn-sbctl lflow-list" command to view the full set of logical flows. The structure will feel somewhat familiar to OpenFlow, but there are some key differences:

1. Ports are logical entities that reside somewhere on a network, not physical ports on a single switch.
2. Each table in the pipeline is given a name in addition to its number. The name describes the purpose of that stage in the pipeline.
3. The match syntax is far more flexible. It supports complex boolean expressions and will feel very familiar to programmers.
4. The actions supported in OVN logical flows extend beyond what you would expect from OpenFlow. We are able to implement higher level features, such as DHCP, in the logical flow syntax. See the documentation for the Logical\_Flow table in [ovn-sb(5)](http://openvswitch.org/support/dist-docs/ovn-sb.5.html) for details on match and action syntax.

There are several additional stages in the pipeline reserved for features not being used in this example, so the flows in many of the tables are not doing anything interesting.
```
    $ ovn-sbctl lflow-list
    Datapath: "sw0" (d7bf4a7b-e915-4502-8f9d-5995d33f5d10)  Pipeline: ingress
      table=0 (ls_in_port_sec_l2  ), priority=100  , match=(eth.src[40]), action=(drop;)
      table=0 (ls_in_port_sec_l2  ), priority=100  , match=(vlan.present), action=(drop;)
      table=0 (ls_in_port_sec_l2  ), priority=50   , match=(inport == "sw0-port1" && eth.src == {00:00:00:00:00:01}), action=(next;)
      table=0 (ls_in_port_sec_l2  ), priority=50   , match=(inport == "sw0-port2" && eth.src == {00:00:00:00:00:02}), action=(next;)
      table=1 (ls_in_port_sec_ip  ), priority=0    , match=(1), action=(next;)
      table=2 (ls_in_port_sec_nd  ), priority=90   , match=(inport == "sw0-port1" && eth.src == 00:00:00:00:00:01 && arp.sha == 00:00:00:00:00:01), action=(next;)
      table=2 (ls_in_port_sec_nd  ), priority=90   , match=(inport == "sw0-port1" && eth.src == 00:00:00:00:00:01 && ip6 && nd && ((nd.sll == 00:00:00:00:00:00 || nd.sll == 00:00:00:00:00:01) || ((nd.tll == 00:00:00:00:00:00 || nd.tll == 00:00:00:00:00:01)))), action=(next;)
      table=2 (ls_in_port_sec_nd  ), priority=90   , match=(inport == "sw0-port2" && eth.src == 00:00:00:00:00:02 && arp.sha == 00:00:00:00:00:02), action=(next;)
      table=2 (ls_in_port_sec_nd  ), priority=90   , match=(inport == "sw0-port2" && eth.src == 00:00:00:00:00:02 && ip6 && nd && ((nd.sll == 00:00:00:00:00:00 || nd.sll == 00:00:00:00:00:02) || ((nd.tll == 00:00:00:00:00:00 || nd.tll == 00:00:00:00:00:02)))), action=(next;)
      table=2 (ls_in_port_sec_nd  ), priority=80   , match=(inport == "sw0-port1" && (arp || nd)), action=(drop;)
      table=2 (ls_in_port_sec_nd  ), priority=80   , match=(inport == "sw0-port2" && (arp || nd)), action=(drop;)
      table=2 (ls_in_port_sec_nd  ), priority=0    , match=(1), action=(next;)
      table=3 (ls_in_pre_acl      ), priority=0    , match=(1), action=(next;)
      table=4 (ls_in_pre_lb       ), priority=0    , match=(1), action=(next;)
      table=5 (ls_in_pre_stateful ), priority=100  , match=(reg0[0] == 1), action=(ct_next;)
      table=5 (ls_in_pre_stateful ), priority=0    , match=(1), action=(next;)
      table=6 (ls_in_acl          ), priority=0    , match=(1), action=(next;)
      table=7 (ls_in_qos_mark     ), priority=0    , match=(1), action=(next;)
      table=8 (ls_in_lb           ), priority=0    , match=(1), action=(next;)
      table=9 (ls_in_stateful     ), priority=100  , match=(reg0[1] == 1), action=(ct_commit(ct_label=0/1); next;)
      table=9 (ls_in_stateful     ), priority=100  , match=(reg0[2] == 1), action=(ct_lb;)
      table=9 (ls_in_stateful     ), priority=0    , match=(1), action=(next;)
      table=10(ls_in_arp_rsp      ), priority=0    , match=(1), action=(next;)
      table=11(ls_in_dhcp_options ), priority=0    , match=(1), action=(next;)
      table=12(ls_in_dhcp_response), priority=0    , match=(1), action=(next;)
      table=13(ls_in_l2_lkup      ), priority=100  , match=(eth.mcast), action=(outport = "_MC_flood"; output;)
      table=13(ls_in_l2_lkup      ), priority=50   , match=(eth.dst == 00:00:00:00:00:01), action=(outport = "sw0-port1"; output;)
      table=13(ls_in_l2_lkup      ), priority=50   , match=(eth.dst == 00:00:00:00:00:02), action=(outport = "sw0-port2"; output;)
    Datapath: "sw0" (d7bf4a7b-e915-4502-8f9d-5995d33f5d10)  Pipeline: egress
      table=0 (ls_out_pre_lb      ), priority=0    , match=(1), action=(next;)
      table=1 (ls_out_pre_acl     ), priority=0    , match=(1), action=(next;)
      table=2 (ls_out_pre_stateful), priority=100  , match=(reg0[0] == 1), action=(ct_next;)
      table=2 (ls_out_pre_stateful), priority=0    , match=(1), action=(next;)
      table=3 (ls_out_lb          ), priority=0    , match=(1), action=(next;)
      table=4 (ls_out_acl         ), priority=0    , match=(1), action=(next;)
      table=5 (ls_out_qos_mark    ), priority=0    , match=(1), action=(next;)
      table=6 (ls_out_stateful    ), priority=100  , match=(reg0[1] == 1), action=(ct_commit(ct_label=0/1); next;)
      table=6 (ls_out_stateful    ), priority=100  , match=(reg0[2] == 1), action=(ct_lb;)
      table=6 (ls_out_stateful    ), priority=0    , match=(1), action=(next;)
      table=7 (ls_out_port_sec_ip ), priority=0    , match=(1), action=(next;)
      table=8 (ls_out_port_sec_l2 ), priority=100  , match=(eth.mcast), action=(output;)
      table=8 (ls_out_port_sec_l2 ), priority=50   , match=(outport == "sw0-port1" && eth.dst == {00:00:00:00:00:01}), action=(output;)
      table=8 (ls_out_port_sec_l2 ), priority=50   , match=(outport == "sw0-port2" && eth.dst == {00:00:00:00:00:02}), action=(output;)

```
The easiest way to understand logical flows is to use the ovn-trace command. ovn-trace allows you to see how OVN would process a sample packet.

ovn-trace has two required arguments:
```
    $ ovn-trace DATAPATH MICROFLOW

```
DATAPATH identifies the logical datapath (a logical switch or a logical router) where the sample packet will begin. MICROFLOW describes the sample packet to be simulated. Much more detail can be found in the [ovn-trace(8) man page](http://openvswitch.org/support/dist-docs/ovn-trace.8.html).

Given our sample OVN configuration, let's see how OVN would process a packet from sw0-port1 that is intended for sw0-port2. ovn-trace has a few different levels of detail to choose from. The first is --minimal, which tells you what happens to a packet, but omits a lot of unnecessary detail. In this case, we see that the final result is that the packet will be delivered to sw0-port2, as expected.
```
    $ ovn-trace --minimal sw0 'inport == "sw0-port1" && eth.src == 00:00:00:00:00:01 && eth.dst == 00:00:00:00:00:02'
    # reg14=0x1,vlan_tci=0x0000,dl_src=00:00:00:00:00:01,dl_dst=00:00:00:00:00:02,dl_type=0x0000
    output("sw0-port2");

```
The next level of detail is given if you use the --summary option. In this mode, we get more detail about packet processing, including which pipeline is being executed. If we run ovn-trace with the same sample packet, we get a better idea of how the packet is processed. We see that:

1. The packet enters the network (sw0) from port sw0-port1 and runs the ingress pipeline.
2. We can see the value "sw0-port2" set to the "outport" variable, indicating that the intended destination for this packet is "sw0-port2".
3. The packet is output from the ingress pipeline, which brings it to the egress pipeline for "sw0" with the outport variable set to "sw0-port2".
4. The output action is executed in the egress pipeline, which outputs the packet to the current value of the "outport" variable, which is "sw0-port2".

```
    $ ovn-trace --summary sw0 'inport == "sw0-port1" && eth.src == 00:00:00:00:00:01 && eth.dst == 00:00:00:00:00:02'
    # reg14=0x1,vlan_tci=0x0000,dl_src=00:00:00:00:00:01,dl_dst=00:00:00:00:00:02,dl_type=0x0000
    ingress(dp="sw0", inport="sw0-port1") {
        outport = "sw0-port2";
        output;
        egress(dp="sw0", inport="sw0-port1", outport="sw0-port2") {
            output;
            /* output to "sw0-port2", type "" */;
        };
    };

```
While debugging a problem or modifying the code, you may want even more detailed output. ovn-trace has a --detailed option. In this case you get more details about each meaningful logical flow encountered. You see the table number, pipeline stage name, full match, and priority number from the flow. You also get a reference to the location in the OVN source code that is responsible for the creation of that logical flow.
```
    $ ovn-trace --detailed sw0 'inport == "sw0-port1" && eth.src == 00:00:00:00:00:01 && eth.dst == 00:00:00:00:00:02'
    # reg14=0x1,vlan_tci=0x0000,dl_src=00:00:00:00:00:01,dl_dst=00:00:00:00:00:02,dl_type=0x0000

    ingress(dp="sw0", inport="sw0-port1")
    -------------------------------------
     0. ls_in_port_sec_l2 (ovn-northd.c:2827): inport == "sw0-port1" && eth.src == {00:00:00:00:00:01}, priority 50
        next(1);
    13. ls_in_l2_lkup (ovn-northd.c:3095): eth.dst == 00:00:00:00:00:02, priority 50
        outport = "sw0-port2";
        output;

    egress(dp="sw0", inport="sw0-port1", outport="sw0-port2")
    ---------------------------------------------------------
     8. ls_out_port_sec_l2 (ovn-northd.c:3170): outport == "sw0-port2" && eth.dst == {00:00:00:00:00:02}, priority 50
        output;
        /* output to "sw0-port2", type "" */
```
Another good example of using ovn-trace would be to see why a packet is getting dropped.  We've enabled port security, so let's get a detailed trace of what would happen to a packet sent from sw0-port1 that contained an unexpected source MAC address.  The output will show us that the packet entered sw0 and failed to match any flow in table 0, meaning the packet is dropped.  We also see that table 0 is named "ls\_in\_port\_sec\_l2", short for "Logical Switch ingress L2 port security".
```
    $ ovn-trace --detailed sw0 'inport == "sw0-port1" && eth.src == 00:00:00:00:00:ff && eth.dst == 00:00:00:00:00:02'
# reg14=0x1,vlan_tci=0x0000,dl_src=00:00:00:00:00:ff,dl_dst=00:00:00:00:00:02,dl_type=0x0000

    ingress(dp="sw0", inport="sw0-port1")
    -------------------------------------
    0. ls_in_port_sec_l2: no match (implicit drop)
```
A similar example would be if a packet contained an unknown destination MAC address.  In this case, we'll see that the packet successfully passed table 0, but failed to match in table 13, "ls\_in\_l2\_lkup", short for "Logical Switch ingress L2 lookup".
```
    $ ovn-trace --detailed sw0 'inport == "sw0-port1" && eth.src == 00:00:00:00:00:01 && eth.dst == 00:00:00:00:00:ff'
    # reg14=0x1,vlan_tci=0x0000,dl_src=00:00:00:00:00:01,dl_dst=00:00:00:00:00:ff,dl_type=0x0000

    ingress(dp="sw0", inport="sw0-port1")
    -------------------------------------
     0. ls_in_port_sec_l2 (ovn-northd.c:2827): inport == "sw0-port1" && eth.src == {00:00:00:00:00:01}, priority 50
        next(1);
    13. ls_in_l2_lkup: no match (implicit drop)

```
So far, we have only looked at examples of a single L2 logical network.  Let's create a new environment that shows how ovn-trace works across multiple networks.  We will create 2 networks, each with 2 ports, and connect them with a logical router.
```
#!/bin/bash

# Create the first logical switch and its two ports.
ovn-nbctl ls-add sw0

ovn-nbctl lsp-add sw0 sw0-port1
ovn-nbctl lsp-set-addresses sw0-port1 "00:00:00:00:00:01 10.0.0.51"
ovn-nbctl lsp-set-port-security sw0-port1 "00:00:00:00:00:01 10.0.0.51"

ovn-nbctl lsp-add sw0 sw0-port2
ovn-nbctl lsp-set-addresses sw0-port2 "00:00:00:00:00:02 10.0.0.52"
ovn-nbctl lsp-set-port-security sw0-port2 "00:00:00:00:00:02 10.0.0.52"

# Create the second logical switch and its two ports.
ovn-nbctl ls-add sw1

ovn-nbctl lsp-add sw1 sw1-port1
ovn-nbctl lsp-set-addresses sw1-port1 "00:00:00:00:00:03 192.168.1.51"
ovn-nbctl lsp-set-port-security sw1-port1 "00:00:00:00:00:03 192.168.1.51"

ovn-nbctl lsp-add sw1 sw1-port2
ovn-nbctl lsp-set-addresses sw1-port2 "00:00:00:00:00:04 192.168.1.52"
ovn-nbctl lsp-set-port-security sw1-port2 "00:00:00:00:00:04 192.168.1.52"

# Create a logical router between sw0 and sw1.
ovn-nbctl create Logical_Router name=lr0

ovn-nbctl lrp-add lr0 lrp0 00:00:00:00:ff:01 10.0.0.1/24
ovn-nbctl lsp-add sw0 sw0-lrp0 \
    -- set Logical_Switch_Port sw0-lrp0 type=router \
    options:router-port=lrp0 addresses='"00:00:00:00:ff:01"'

ovn-nbctl lrp-add lr0 lrp1 00:00:00:00:ff:02 192.168.1.1/24
ovn-nbctl lsp-add sw1 sw1-lrp1 \
    -- set Logical_Switch_Port sw1-lrp1 type=router \
    options:router-port=lrp1 addresses='"00:00:00:00:ff:02"'

```
We can then use "ovn-nbctl show" to view the resulting logical network configuration.
```
$ ovn-nbctl show
    switch bf4ba6c6-91c5-4f56-9981-72643816f923 (sw1)
        port sw1-lrp1
            addresses: ["00:00:00:00:ff:02"]
        port sw1-port2
            addresses: ["00:00:00:00:00:04 192.168.1.52"]
        port sw1-port1
            addresses: ["00:00:00:00:00:03 192.168.1.51"]
    switch 13b80127-4b36-46ea-816a-1ba4ffd6ac57 (sw0)
        port sw0-port1
            addresses: ["00:00:00:00:00:01 10.0.0.51"]
        port sw0-lrp0
            addresses: ["00:00:00:00:ff:01"]
        port sw0-port2
            addresses: ["00:00:00:00:00:02 10.0.0.52"]
    router 68935017-967a-4c4a-9dad-5d325a9f203a (lr0)
        port lrp0
            mac: "00:00:00:00:ff:01"
            networks: ["10.0.0.1/24"]
        port lrp1
            mac: "00:00:00:00:ff:02"
            networks: ["192.168.1.1/24"]

```
We should be able to trace a packet sent from sw0-port1 destined for sw1-port2, which requires going through the router. The minimal output will confirm that the end result is that the packet should be output to sw1-port2. We will also see what modifications the packet received along the way. As the packet traversed the logical router, the TTL was decremented and then the source and destination MAC addresses were updated for the next hop.
```
$ ovn-trace --minimal sw0 'inport == "sw0-port1" && \
                           eth.src == 00:00:00:00:00:01 && \
                           ip4.src == 10.0.0.51 && \
                           eth.dst == 00:00:00:00:ff:01 && \
                           ip4.dst == 192.168.1.52 && \
                           ip.ttl == 32'
# ip,reg14=0x1,vlan_tci=0x0000,dl_src=00:00:00:00:00:01,dl_dst=00:00:00:00:ff:01,nw_src=10.0.0.51,nw_dst=192.168.1.52,nw_proto=0,nw_tos=0,nw_ecn=0,nw_ttl=32
ip.ttl--;
eth.src = 00:00:00:00:ff:02;
eth.dst = 00:00:00:00:00:04;
output("sw1-port2");

```
If you'd like to take an even closer look, you can experiment with the previous ovn-trace command by changing the verbosity to --summary or --detailed. You could also start to make changes to the sample packet to see what would happen.

## A Powerful Abstraction

I mentioned at the very beginning of this post that I find OVN Logical Flows to be a powerful abstraction that has made adding features to OVN much easier than I anticipated. Now that we've gone through logical flows in some detail, I'd like to point to a couple of recent feature developments that help demonstrate how easy logical flows make adding features to OVN.

### Source Based Routing

OVN has support for L3 gateways that can be used to provide connectivity between OVN logical networks and physical networks. A typical OVN network might have an L3 gateway that resides on a single host. The downside to using a single L3 gateway is that all traffic destined for that physical network must go through the single host where the L3 gateway resides.

Recently, [Gurucharan Shetty added support for multiple L3 gateways on an OVN logical network](https://github.com/openvswitch/ovs/commit/440a9f4b32bead4fa82d93b1fdceed3e55c20b4b). The method supported for distributing traffic among the gateways is based on source IP address.

It's not important to understand all of the details in this change. I mainly want to draw attention to how little of a code change was required. Let's take a look at the diffstat, organizied by the type of change.
```
Documentation:
 NEWS                          |    1 
 ovn/ovn-nb.xml                |   28 +++++
 ovn/utilities/ovn-nbctl.8.xml |    8 +

Database schema update:
 ovn/ovn-nb.ovsschema          |    8 +

Command line utility support for new db schema additions:
 ovn/utilities/ovn-nbctl.c     |   43 ++++++--

Changes to how OVN builds logical flows to add support for this feature:
 ovn/northd/ovn-northd.c       |   24 +++-

Test code:
 tests/ovn-nbctl.at            |   42 ++++----
 tests/ovn.at                  |  219 ++++++++++++++++++++++++++++++++++++++++++

 8 files changed, 334 insertions(+), 39 deletions(-)

```
At the very core of this feature is the 24 lines of code changed in ovn-northd.c. This is where the code that generates logical flows was updated for this feature. That is amazing to me. This feature has a significant impact on network behavior, yet was accomplished in very few lines of C code.

### DSCP

Another example of adding a feature using logical flows is [this patch that adds support for setting the DSCP field of IP packets based on an arbitrary traffic classifier](https://github.com/openvswitch/ovs/commit/1a03fc7da32e74aacbf638d3617a290ddffaa069).

The patch added a new QoS table to the OVN northbound database. In this table you define a match (or traffic classifier) and the corresponding DSCP value to set on packets that match this classifier.

The key changes to the code are the 80 lines changed in ovn-northd.c. The patch looks a bit bigger than it really is because it created a new pipeline stage for QoS and had to renumber the stages that followed it.

Implementing this feature using logical flows just requires inserting flows matching the configured match (traffic classifier), and then using the OVN logical flow actions "ip.dscp = VALUE; next;".

### DHCP

OVN supports DHCPv4 and DHCPv6, but all of the details are controlled through logical flows. In most cases, if behavior needs to be changed, it's only a change to the code that generates the DHCP related logical flows.

A recent example was [this patch which added stateless DHCPv6 support](https://github.com/openvswitch/ovs/commit/40df456680fc1e760d9cc666fef1761b5234cf00). With stateless DHCPv6, we want to provide some configuration entries (such as a DNS address), but not assign an IPv6 address. Implementing this was just a small tweak to the DHCPv6 logical flows to optionally not include the IPv6 address option in the response generated by OVN.

## Conclusion

I hope you now have a better understanding of OVN Logical Flows, a match-action pipeline for defining the behavior of logical networks that can span many hosts.

Thanks again to Ben Pfaff for more recently writing ovn-trace, which makes it easier to read, understand, and modify OVN logical flows. Ben talked about ovn-trace as a part of our OVN talk at the OpenStack Summit in Barcelona. You can find a video of that talk here:

\[youtube https://www.youtube.com/watch?v=q3cJ6ezPnCU\]
