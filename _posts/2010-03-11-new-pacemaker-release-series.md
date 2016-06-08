---
layout: post
title: New Pacemaker Release Series
tags:
- announce
---
A number of new branches have been created in the last few days which are
integral to how we plan to add new features in a controlled manner.

Current set of branches:

  * 1.0 - The existing stable series
  * 1.1 - The current feature series
  * 1.2 - The next stable series (expected Q4 2010)
  * devel - Where new features are added

The idea is that _1.0_ will continue receive only bugfixes (the amount
hopefully continuing to reduce over time) and all new features go first into
_devel_.

_1.1_ will be the merge point, primarily receiving bug fixes for the new
features we pulled in from _devel_ (at least two weeks before a _1.1_
release).

The intention is to pursue bi-monthly releases of _1.1_ over the next 6-12
months until we can offer a set of compelling new features, a sane way to use
them, and rock-solid stability.

At that time, we will freeze development and release it as 1.2.0

![Here is the repository layout in graphical form](/images/tumblr_kz4mgizPlt1qzagr8.jpg)

### But Can I use it in Production?

Cluster admins are naturally a cautious bunch, so in order to make _1.1_ more
palatable, we have also created two new schemas.

Schemas can be thought of as a contract between the developers and admins.
They are a declaration of what we support.

The set is the new set of validation types:

  * pacemaker-1.0
  * pacemaker-1.1
  * pacemaker-1.2

_pacemaker-1.0_ is (and will remain) **identical** to what people are using
now

When new features are added, their configuration syntax will first appear in
_pacemaker-1.1_. If, based on your feedback, a feature's syntax needs to be
modified we will change it here and solicit further feedback.

Once a feature has been locked down and become stable, its configuration
pieces will be added to _pacemaker-1.2_. Once a feature appears in
_pacemaker-1.2_, we will not change its syntax in any _1.1_ or _1.2_ release.

#### What does this mean?

It means that when used with the _pacemaker-1.0_ or _pacemaker-1.2_ schemas,
_1.1_ can be safely used in production in the same manner as _1.0_ is today.

  * If the existing syntax is all you need, consider _1.1_ with the _pacemaker-1.0_ schema.
  * If you want to try a new stable feature, use _1.1_ with the _pacemaker-1.2_ schema.
  * If you want to try a new experimental feature, use _1.1_ with the _pacemaker-1.1_ schema.

### Is 1.0 Still Supported?

Yes. 1.0 i still supported and will recieve all relevant bug fixes until at
least 2012. See our [releases](http://www.clusterlabs.org/wiki/Releases) page.

### How can I Install it?

Users of RPM-based distributions will be able to install in the usual manner.
However, to avoid forcing everyone to upgrade, packages will be located under:

    
[http://www.clusterlabs.org/rpm-next](http://www.clusterlabs.org/rpm-next)
    

At this time the intention is to only produce packages for the most recent
openSUSE (11.2), Fedora (12), and EPEL (5) based distributions. If you would
like to try 1.1 but your distribution/version is not listed, please contact
the project and we’ll see what we can do.

Packages should be available in the next week "or so".

### Identified Areas of Development

#### Existing

  1. New stonith daemon (configuration is unchanged), including: 
    * support for RHCS Stonith Agents
    * cluster-wide notifications (that allow the cluster to make more intelligent decisions)
    * ability to perform multiple fencing operations in parallel (faster recovery)
  2. Service placement influenced by the physical resources (RAM, CPU, etc) required by the service (and offered by the host)
  3. A new tool for simulating failures and the cluster's reaction to them
  4. Tell the cluster to serialize an otherwise unrelated a set of resource actions (eg. Xen migrations) 

#### Planned

  1. Failover domains, an easy way to specify ordered preferences for a set of hosts
  2. ACLs, the ability to restrict read/write access to the configuration based on users and groups
  3. Improvements to the System Health feature
  4. Freeze/thaw, an easy way to ensure a consistent backup of data in use by distributed systems.
  5. The ability to monitor services on non-cluster nodes (such as VMs)

