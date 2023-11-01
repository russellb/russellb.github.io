---
title: "OpenStack Compute (Nova) Roadmap for Havana"
date: "2013-05-13"
categories: 
  - "openstack"
---

The Havana design summit was held mid-April.  Since then we have been documenting the Havana roadmap and going full speed ahead on development of these features.  The list of features that developers have committed to completing for the Havana release is tracked using [blueprints on Launchpad](https://blueprints.launchpad.net/nova/havana). At the time of writing, we have 74 blueprints listed that cover a wide range of development efforts.  Here are some highlights in no particular order:

## Database Handling

Vish Ishaya made a change at the very beginning of the development cycle that will allow us to [backport database migrations](https://blueprints.launchpad.net/nova/+spec/backportable-db-migrations) to the Grizzly release if needed. This is needed in case we need to backport a bug fix that requires a migration.

Dan Smith and Chris Behrens are working on a [unified object model](https://blueprints.launchpad.net/nova/+spec/unified-object-model). One of the things that has been in the way of rolling upgrades of a Nova deployment is that the code and the database schema are very tightly coupled. The primary goal of this effort is to decouple these things. This effort is bringing in some other improvements, as well, including better object serialization handling for rpc, as well as object versioning.

Boris Pavlovic continues to do a lot of cleanup of database support in Nova.  He's [adding tests](https://blueprints.launchpad.net/nova/+spec/db-api-tests) (and [more tests](https://blueprints.launchpad.net/nova/+spec/db-api-tests-on-all-backends)), adding [unique constraints](https://blueprints.launchpad.net/nova/+spec/db-enforce-unique-keys), [improving session handling](https://blueprints.launchpad.net/nova/+spec/db-session-cleanup), and [improving archiving](https://blueprints.launchpad.net/nova/+spec/db-improve-archiving).

Chris Behrens has been working on a [native MySQL database driver](https://blueprints.launchpad.net/nova/+spec/db-mysqldb-impl) that performs much better than the SQLAlchemy driver for use in large scale deployments.

Mike Wilson is working on supporting [read-only database slaves](https://blueprints.launchpad.net/nova/+spec/db-slave-handle). This will allow distributing some queries to other database servers to help scaling in large scale deployments.

## Bare Metal

The Grizzly release of Nova included the bare metal provisioning driver. Interest in this functionality has been rapidly increasing. Devananda van der Veen proposed that the bare metal provisioning code be [split out into a new project](http://https://blueprints.launchpad.net/nova/+spec/deprecate-baremetal-driver) called Ironic. The new project was [approved for incubation](http://http://eavesdrop.openstack.org/meetings/tc/2013/tc.2013-05-07-20.01.html) by the OpenStack Technical Committee last week. Once this has been completed, there will be a driver in Nova that talks to the Ironic API. The Ironic API will present some additional functionality that doesn't make sense to use to present in the Compute API in Nova.

Prior to the focus shift to Ironic, some new features were added to the bare metal driver. USC-ISI [added support for Tilera](https://blueprints.launchpad.net/nova/+spec/add-tilera-to-baremetal) and Devananda added a feature that allows you to [request a specific bare metal node](https://blueprints.launchpad.net/nova/+spec/baremetal-force-node) when provisioning a server.

## Version 3 (v3) of the Compute API

The Havana release will include [a new revision of the compute REST API](https://blueprints.launchpad.net/nova/+spec/nova-v3-api) in Nova. This effort is being led by Christopher Yeoh, with help from others. The v3 API will include [a new framework for implementing extensions](https://blueprints.launchpad.net/nova/+spec/v3-api-extension-framework), [extension versioning](https://blueprints.launchpad.net/nova/+spec/v3-api-extension-versioning), and a whole bunch of cleanup: ([1](https://blueprints.launchpad.net/nova/+spec/v3-api-remove-project-id)) ([2](https://blueprints.launchpad.net/nova/+spec/v3-api-expected-errors)) ([3](https://blueprints.launchpad.net/nova/+spec/v3-api-inconsistencies)) ([4](https://blueprints.launchpad.net/nova/+spec/v3-api-return-codes)).

## Networking

The OpenStack community has been maintaining two network stacks for some time. Nova includes the nova-network service. Meanwhile, the OpenStack Networking project has been developed from scratch to support much more than nova-network does. Nova currently supports both. OpenStack Networking is expected to reach and surpass feature parity with nova-network in the Havana cycle. As a result, it's time to deprecate [nova-network](https://blueprints.launchpad.net/nova/+spec/deprecate-nova-network). Vish Ishaya (from the Nova side) and Gary Kotton (from the OpenStack Networking side) have agreed to take on the challenging task of figuring out how to migrate existing deployments using nova-network to an updated environment that includes OpenStack Networking.

## Scheduling

The Havana roadmap includes a mixed bag of scheduler features.

Andrew Laski is going to make the changes required so that the scheduler becomes exclusively a [resource that gets queried](https://blueprints.launchpad.net/nova/+spec/query-scheduler). Currently, when starting an instance, the request is handed off to the scheduler, which then hands it off to the compute node that is selected. This change will make it so proxying through nova-scheduler is no longer done. This will mean that _every_ operation that uses the scheduler will interact with it the same way, as opposed to some operations querying and others proxying.

Phil Day will be adding an API extension that allows you to [discover which scheduler hints are supported](https://blueprints.launchpad.net/nova/+spec/scheduler-hints-api).  Phil is also looking at adding a way to [allocate an entire host](https://blueprints.launchpad.net/nova/+spec/whole-host-allocation) to a single tenant.

Inbar Shapira is looking at allowing [multiple scheduling policies](https://blueprints.launchpad.net/nova/+spec/multiple-scheduler-drivers) to be in effect at the same time.  This will allow you to have different sets of scheduler filters activated depending on some type of criteria (perhaps the requested availability zone).

Rerngvit Yanggratoke is implementing support for [weighting scheduling decisions based on the CPU utilization](https://blueprints.launchpad.net/nova/+spec/utilization-based-scheduling) of existing instances on a host.

## Migrations

Nova includes support for different types of migrations. We have cold migrations (migrate) and live migrations (live-migrate). We also have resize and evactuate, which are very related functions. The code paths for all of these features have evolved separately. It turns out that we can rework all of these things to share a lot of code. While we're at it, we are restructuring the way these operations work to be primarily driven by the nova-conductor service.  This will allow the tasks to be tracked in a single place, as opposed to the flow of control being passed around between compute nodes. Having compute nodes tell each other what to do is also a very bad thing from a security perspective. These efforts are well underway. Tiago Rodrigues de Mello is working on [moving cold migrations to nova-conductor](https://blueprints.launchpad.net/nova/+spec/cold-migrations-to-conductor) and John Garbutt is working on [moving live migrations](https://blueprints.launchpad.net/nova/+spec/live-migration-to-conductor). All of this is tracked under the [parent blueprint for unified migrations](https://blueprints.launchpad.net/nova/+spec/unified-migrations).

## And More!

This post doesn't include every feature on the roadmap. You can find that [here](https://blueprints.launchpad.net/nova/havana). I fully expect that more will be added to this list as Havana progresses. We don't always know what features are being worked on in advance. If you have another feature you would like to propose, let's talk about it on [the openstack-dev list](http://lists.openstack.org/pipermail/openstack-dev/)!
