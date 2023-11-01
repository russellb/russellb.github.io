---
title: "Deployment Considerations for nova-conductor Service in OpenStack Grizzly"
date: "2013-02-19"
categories: 
  - "openstack"
---

The Grizzly release of OpenStack Nova includes a new service, nova-conductor. Some previous posts about this service can be found [here](http://russellbryantnet.wordpress.com/2012/11/19/a-new-nova-service-nova-conductor/) and [here](http://www.danplanet.com/blog/2013/02/07/all-your-db-are-belong-to-conductor/). This post is intended to provide some additional insight into how this service should be deployed and how the service should be scaled as load increases.

Smaller OpenStack Compute deployments typically consist of a single controller node and one or more compute (hypervisor) nodes. The nova-conductor service fits in the category of controller services. In this style of deployment you would run nova-conductor on the controller node and not on the compute nodes. Note that most of what nova-conductor does in the Grizzly release is doing database operations on behalf of compute nodes. This means that the controller node will have more work to do than in previous releases. Load should be monitored and the controller services should be scaled out if necessary.

Here is a model of what this size of deployment might look like:

[![nova-simple](http://russellbryantnet.files.wordpress.com/2013/02/nova-simple.png?w=300)](http://russellbryantnet.files.wordpress.com/2013/02/nova-simple.png)

As Compute deployments get larger, the controller services are scaled horizontally. For example, there would be multiple instances of the nova-api service on multiple nodes sitting behind a load balancer. You may also be running multiple instances of the nova-scheduler service across multiple nodes. Load balancing is done automatically for the scheduler by the AMQP message broker, RabbitMQ or Qpid. The nova-conductor service should be scaled out in this same way. It can be run multiple times across multiple nodes and the load will be balanced automatically by the message broker.

Here is a second deployment model. This gives an idea of how a deployment grows beyond a single controller node.

[![nova-complex](http://russellbryantnet.files.wordpress.com/2013/02/nova-complex.png?w=258)](http://russellbryantnet.files.wordpress.com/2013/02/nova-complex.png)

There are a couple of ways to monitor the performance of nova-conductor to see if it needs to be scaled out. The first is by monitoring CPU load. The second is by monitoring message queue size. If the queues are getting backed up, it is likely time to scale out services.  Both messaging systems provide at least one way to look at the state of message queues. For Qpid, try the `qpid-stat` command. For RabbitMQ, see the `[rabbitmqctl list_queues](http://www.rabbitmq.com/man/rabbitmqctl.1.man.html)` command.
