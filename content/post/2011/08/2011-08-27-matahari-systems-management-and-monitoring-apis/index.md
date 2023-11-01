---
title: "Matahari: Systems Management and Monitoring APIs"
date: "2011-08-27"
---

I have been working at Red Hat for a few weeks now and have started getting some real work done. I wanted to share what I'm currently working on, and that is [Matahari](http://www.matahariproject.org/). Matahari is a cross-platform (Linux and Windows so far) collection of APIs accessible over local and remote interfaces for systems management and monitoring. What the heck does that mean? Read on, dear friends.

**Architecture**

I mentioned that Matahari is a collection of APIs. These APIs are accessible via a few different methods. The core of the functionality that we are implementing is done as C libraries. These can be used directly. However, we expect and intend for most users to access the functionality via one of the agents we provide. A Matahari Agent is an application that provides access to the features implemented in a Matahari Library via some transport. We are currently providing agents for [D-Bus](http://www.freedesktop.org/wiki/Software/dbus) and [QMF](http://qpid.apache.org/).

D-Bus is used quite heavily as a communications mechanism between applications on a single system. QMF, or the Qpid Management Framework, is used as a remote interface. QMF is a framework for building remote APIs on top of [AMQP](http://www.amqp.org/), an open protocol for messaging.

The agents are generally thin wrappers around a core library, so other transports could be added in the future if the need presents itself.

**Current Features**

So, what can you do with Matahari?

Matahari is still under heavy development, but there is already a decent amount of usable functionality.

- _Host_ - An agent for viewing and controlling hosts

- View basic host information such as OS, hostname, CPU, RAM, load average, and more.
- Host control (shutdown, reboot)

- _Networking_ - An agent for viewing and controlling network devices

- Get a list of available network interfaces and information about them, such as IP and MAC addresses
- Start and stop network interfaces

- _Services_ - An agent for viewing and controlling system services

- List configured services
- Start and stop services
- Monitor the status of services

- _Sysconfig_ - Modify system configuration

- Modify system configuration files (Linux)
- Modify system registry (Windows)

More things that are in the works can be found on the project [backlog](https://github.com/matahari/matahari/wiki/Backlog).

**Use Cases**

An example of a project that already utilizes Matahari is [Pacemaker-cloud](http://pacemaker-cloud.org/), which is also under heavy development. Pacemaker-cloud utilizes both the Host and Services agents of Matahari. Being able to actively monitor and control services on remote hosts is a key element of being able to provide HA in a cloud deployment.

In addition to providing ready-to-use agents, we also provide some code that makes it easier to write a QMF agent so that third-parties can write their own Matahari agents. One example of this that already exists is [libvirt-qmf](http://libvirt.org/), which is a Matahari agent that exposes libvirt functionality over QMF.

**Join Us**

If Matahari interests you, follow us on [github](https://github.com/matahari/matahari) and join our [mailing list](https://fedorahosted.org/mailman/listinfo/matahari).  Thanks!
