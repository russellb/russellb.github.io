---
title: "Comparing OpenStack Neutron ML2+OVS and OVN - Control Plane"
date: "2016-12-19"
categories: 
  - "openstack"
  - "ovn"
  - "ovs"
---

We have done a lot of performance testing of OVN over time, but one major thing missing has been an apples-to-apples comparison with the current OVS-based OpenStack Neutron backend (ML2+OVS).  I've been working with a group of people to compare the two OpenStack Neutron backends.  This is the first piece of those results: the control plane.  Later posts will discuss data plane performance.

# Control Plane Differences

The ML2+OVS control plane is based on a pattern seen throughout OpenStack.  There is a series of agents written in Python.  The Neutron server communicates with these agents using an rpc mechanism built on top of AMQP (RabbitMQ in most deployments, including our tests).

OVN takes a distributed database-driven approach.  Configuration and state is managed through two databases: the OVN northbound and southbound databases.  These databases are currently based on OVSDB.  Instead of receiving updates via RPC, components are watching relevant portions of the database for changes and applying them locally.  More detail about these components can be found in my post about [the first release of OVN](https://blog.russellbryant.net/2016/09/29/ovs-2-6-and-the-first-release-of-ovn/), or even more detail is in the [ovn-architecture document](http://openvswitch.org/support/dist-docs/ovn-architecture.7.html).

OVN does not make use of any of the Neutron agents.  Instead, all required functionality is implemented by ovn-controller and OVS flows.  This includes things like security groups, DHCP, L3 routing, and NAT.

# Hardware and Software

Our testing was done in a lab using 13 machines which were allocated to the following functions:

- 1 OpenStack TripleO Undercloud for provisioning
- 3 Controllers (OpenStack and OVN control plane services)
- 9 Compute Nodes (Hypervisors)

The hardware had the following specs:

- - 2x E5-2620 v2 (12 total cores, 24 total threads)
    - 64GB RAM
    - 4 x 1TB SATA
    - 1 x Intel X520 Dual Port 10G
    

Software:

- CentOS 7.2
- OpenStack, OVS, and OVN from their master branches (early December, 2016)
- Neutron configuration notes
    - (OVN) 6 API workers, 1 RPC worker (since rpc is not used and neutron requires at least 1) for neutron-server on each controller (x3)
    - (ML2+OVS) 6 API workers, 6 RPC workers for neutron-server on each controller (x3)
    - (ML2+OVS) DVR was enabled

# Test Configuration

The tests were run using [OpenStack Rally](https://github.com/openstack/rally).  We used the [Browbeat](https://github.com/openstack/browbeat) project to easily set up, configure, and run the tests, as well as store, analyze, and compare results.  The rally portion of the browbeat configuration was:
```
rerun: 3
...
rally:
  enabled: true
  sleep_before: 5
  sleep_after: 5
  venv: /home/stack/rally-venv/bin/activate
  plugins:
    - netcreate-boot: rally/rally-plugins/netcreate-boot
    - subnet-router-create: rally/rally-plugins/subnet-router-create
    - neutron-securitygroup-port: rally/rally-plugins/neutron-securitygroup-port
  benchmarks:
    - name: neutron
      enabled: true
      concurrency:
        - 8
        - 16
        - 32 
      times: 500
      scenarios:
        - name: create-list-network
          enabled: true
          file: rally/neutron/neutron-create-list-network-cc.yml
        - name: create-list-port
          enabled: true
          file: rally/neutron/neutron-create-list-port-cc.yml
        - name: create-list-router
          enabled: true
          file: rally/neutron/neutron-create-list-router-cc.yml
        - name: create-list-security-group
          enabled: true
          file: rally/neutron/neutron-create-list-security-group-cc.yml
        - name: create-list-subnet
          enabled: true
          file: rally/neutron/neutron-create-list-subnet-cc.yml
    - name: plugins
      enabled: true
      concurrency:
        - 8
        - 16
        - 32 
      times: 500
      scenarios:
        - name: netcreate-boot
          enabled: true
          image_name: cirros
          flavor_name: m1.xtiny
          file: rally/rally-plugins/netcreate-boot/netcreate_boot.yml
        - name: subnet-router-create
          enabled: true
          num_networks:  10
          file: rally/rally-plugins/subnet-router-create/subnet-router-create.yml
        - name: neutron-securitygroup-port
          enabled: true
          file: rally/rally-plugins/neutron-securitygroup-port/neutron-securitygroup-port.yml

```
This configuration defines several scenarios to run.  Each one is set to run 500 times, at three different concurrency levels.  Finally, "rerun: 3" at the beginning says we run the entire configuration 3 times.  This is a bit confusing, so let's look at one example.

The "netcreate-boot" scenario is to create a network and boot a VM on that network.  The configuration results in the following execution:

- Run 1
    - Create 500 VMs, each on their own network, 8 at a time, and then clean up
    - Create 500 VMs, each on their own network, 16 at a time, and then clean up
    - Create 500 VMs, each on their own network, 32 at a time, and then clean up
- Run 2
    - Create 500 VMs, each on their own network, 8 at a time, and then clean up
    - Create 500 VMs, each on their own network, 16 at a time, and then clean up
    - Create 500 VMs, each on their own network, 32 at a time, and then clean up
- Run 3
    - Create 500 VMs, each on their own network, 8 at a time, and then clean up
    - Create 500 VMs, each on their own network, 16 at a time, and then clean up
    - Create 500 VMs, each on their own network, 32 at a time, and then clean up

In total, we will have created 4500 VMs.

# Results

Browbeat includes the ability to store all rally test results in elastic search and then display them using Kibana.  A live dashboard of these results is on [elk.browbeatproject.org](http://elk.browbeatproject.org:80/goto/0ce898c6624a88c0116f3764d26c6222).

The following tables show the results for the average times, 95th percentile, Maximum, and minimum times for all APIs executed throughout the test scenarios.

<table dir="ltr" border="1" cellspacing="0" cellpadding="0"><colgroup><col width="195"> <col width="100"> <col width="100"> <col width="103"></colgroup><tbody><tr><td>API</td><td>ML2+OVS Average</td><td>OVN Average</td><td>% improvement</td></tr><tr><td>nova.boot_server</td><td>80.672</td><td>23.45</td><td><strong>70.93%</strong></td></tr><tr><td>neutron.list_ports</td><td>6.296</td><td>6.478</td><td>-2.89%</td></tr><tr><td>neutron.list_subnets</td><td>5.129</td><td>3.826</td><td>25.40%</td></tr><tr><td>neutron.add_interface_router</td><td>4.156</td><td>3.509</td><td>15.57%</td></tr><tr><td>neutron.list_routers</td><td>4.292</td><td>3.089</td><td>28.03%</td></tr><tr><td>neutron.list_networks</td><td>2.596</td><td>2.628</td><td>-1.23%</td></tr><tr><td>neutron.list_security_groups</td><td>2.518</td><td>2.518</td><td>0.00%</td></tr><tr><td>neutron.remove_interface_router</td><td>3.679</td><td>2.353</td><td>36.04%</td></tr><tr><td>neutron.create_port</td><td>2.096</td><td>2.136</td><td>-1.91%</td></tr><tr><td>neutron.create_subnet</td><td>1.775</td><td>1.543</td><td>13.07%</td></tr><tr><td>neutron.delete_port</td><td>1.592</td><td>1.517</td><td>4.71%</td></tr><tr><td>neutron.create_security_group</td><td>1.287</td><td>1.372</td><td>-6.60%</td></tr><tr><td>neutron.create_network</td><td>1.352</td><td>1.285</td><td>4.96%</td></tr><tr><td>neutron.create_router</td><td>1.181</td><td>0.845</td><td>28.45%</td></tr><tr><td>neutron.delete_security_group</td><td>0.763</td><td>0.793</td><td>-3.93%</td></tr></tbody></table>

 

<table dir="ltr" border="1" cellspacing="0" cellpadding="0"><colgroup><col width="195"> <col width="100"> <col width="100"> <col width="103"></colgroup><tbody><tr><td>API</td><td>ML2+OVS 95%</td><td>OVN 95%</td><td>% improvement</td></tr><tr><td>nova.boot_server</td><td>163.2</td><td>35.336</td><td><strong>78.35%</strong></td></tr><tr><td>neutron.list_ports</td><td>11.038</td><td>11.401</td><td>-3.29%</td></tr><tr><td>neutron.list_subnets</td><td>10.064</td><td>6.886</td><td>31.58%</td></tr><tr><td>neutron.add_interface_router</td><td>7.908</td><td>6.367</td><td>19.49%</td></tr><tr><td>neutron.list_routers</td><td>8.374</td><td>5.321</td><td>36.46%</td></tr><tr><td>neutron.list_networks</td><td>5.343</td><td>5.171</td><td>3.22%</td></tr><tr><td>neutron.list_security_groups</td><td>5.648</td><td>5.556</td><td>1.63%</td></tr><tr><td>neutron.remove_interface_router</td><td>6.917</td><td>4.078</td><td>41.04%</td></tr><tr><td>neutron.create_port</td><td>5.521</td><td>4.968</td><td>10.02%</td></tr><tr><td>neutron.create_subnet</td><td>4.041</td><td>3.091</td><td>23.51%</td></tr><tr><td>neutron.delete_port</td><td>2.865</td><td>2.598</td><td>9.32%</td></tr><tr><td>neutron.create_security_group</td><td>3.245</td><td>3.547</td><td>-9.31%</td></tr><tr><td>neutron.create_network</td><td>3.089</td><td>2.917</td><td>5.57%</td></tr><tr><td>neutron.create_router</td><td>2.893</td><td>1.92</td><td>33.63%</td></tr><tr><td>neutron.delete_security_group</td><td>1.776</td><td>1.72</td><td>3.15%</td></tr></tbody></table>

 

<table dir="ltr" border="1" cellspacing="0" cellpadding="0"><colgroup><col width="195"> <col width="100"> <col width="100"> <col width="103"></colgroup><tbody><tr><td>API</td><td>ML2+OVS Maximum</td><td>OVN Maximum</td><td>% improvement</td></tr><tr><td>nova.boot_server</td><td>221.877</td><td>47.827</td><td><strong>78.44%</strong></td></tr><tr><td>neutron.list_ports</td><td>29.233</td><td>32.279</td><td>-10.42%</td></tr><tr><td>neutron.list_subnets</td><td>35.996</td><td>17.54</td><td>51.27%</td></tr><tr><td>neutron.add_interface_router</td><td>29.591</td><td>22.951</td><td>22.44%</td></tr><tr><td>neutron.list_routers</td><td>19.332</td><td>13.975</td><td>27.71%</td></tr><tr><td>neutron.list_networks</td><td>12.516</td><td>13.765</td><td>-9.98%</td></tr><tr><td>neutron.list_security_groups</td><td>14.577</td><td>13.092</td><td>10.19%</td></tr><tr><td>neutron.remove_interface_router</td><td>35.546</td><td>9.391</td><td>73.58%</td></tr><tr><td>neutron.create_port</td><td>53.663</td><td>40.059</td><td>25.35%</td></tr><tr><td>neutron.create_subnet</td><td>46.058</td><td>26.472</td><td>42.52%</td></tr><tr><td>neutron.delete_port</td><td>5.121</td><td>5.149</td><td>-0.55%</td></tr><tr><td>neutron.create_security_group</td><td>14.243</td><td>13.206</td><td>7.28%</td></tr><tr><td>neutron.create_network</td><td>32.804</td><td>32.566</td><td>0.73%</td></tr><tr><td>neutron.create_router</td><td>14.594</td><td>6.452</td><td>55.79%</td></tr><tr><td>neutron.delete_security_group</td><td>4.249</td><td>3.746</td><td>11.84%</td></tr></tbody></table>

 

<table dir="ltr" border="1" cellspacing="0" cellpadding="0"><colgroup><col width="195"> <col width="100"> <col width="100"> <col width="103"></colgroup><tbody><tr><td>API</td><td>ML2+OVS Minimum</td><td>OVN Minimum</td><td>% improvement</td></tr><tr><td>nova.boot_server</td><td>18.665</td><td>3.761</td><td><strong>79.85%</strong></td></tr><tr><td>neutron.list_ports</td><td>0.195</td><td>0.22</td><td>-12.82%</td></tr><tr><td>neutron.list_subnets</td><td>0.252</td><td>0.187</td><td>25.79%</td></tr><tr><td>neutron.add_interface_router</td><td>1.698</td><td>1.556</td><td>8.36%</td></tr><tr><td>neutron.list_routers</td><td>0.185</td><td>0.147</td><td>20.54%</td></tr><tr><td>neutron.list_networks</td><td>0.21</td><td>0.174</td><td>17.14%</td></tr><tr><td>neutron.list_security_groups</td><td>0.132</td><td>0.184</td><td>-39.39%</td></tr><tr><td>neutron.remove_interface_router</td><td>1.557</td><td>1.057</td><td>32.11%</td></tr><tr><td>neutron.create_port</td><td>0.58</td><td>0.614</td><td>-5.86%</td></tr><tr><td>neutron.create_subnet</td><td>0.42</td><td>0.416</td><td>0.95%</td></tr><tr><td>neutron.delete_port</td><td>0.464</td><td>0.46</td><td>0.86%</td></tr><tr><td>neutron.create_security_group</td><td>0.081</td><td>0.094</td><td>-16.05%</td></tr><tr><td>neutron.create_network</td><td>0.113</td><td>0.179</td><td>-58.41%</td></tr><tr><td>neutron.create_router</td><td>0.077</td><td>0.053</td><td>31.17%</td></tr><tr><td>neutron.delete_security_group</td><td>0.092</td><td>0.104</td><td>-13.04%</td></tr></tbody></table>

# Analysis

The most drastic difference in results is for "nova.boot\_server".  This is also the one piece of these tests that actually measures the time it takes to provision the network, and not just loading Neutron with configuration.

When Nova boots a server, it blocks waiting for an event from Neutron indicating that a port is ready before it sets the server state to ACTIVE and powers on the VM.  Both ML2+OVS and OVN implement this mechanism.  Our test scenario measured the time it took for servers to become ACTIVE.

Further tests were done on ML2+OVS and we were able to confirm that disabling this synchronization between Nova and Neutron brought the results back to being on par with the OVN results.  This confirmed that the extra time was indeed spent waiting for Neutron to report that ports were ready.

To be clear, you should not disable this synchronization.  The only reason you can disable it is because not all Neutron backends support it (ML2+OVS and OVN both do).  It was put in place to avoid a race condition.  It ensures that the network is actually ready for use before booting a VM.  The issue is how long it's taking Neutron to provision the network for use.  Further analysis is needed to break down where Neutron (ML2+OVS) is spending most of its time in the provisioning process.
