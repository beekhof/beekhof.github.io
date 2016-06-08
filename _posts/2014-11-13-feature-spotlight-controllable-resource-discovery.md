---
layout: post
date: 2014-11-13 13:09
title: Feature Spotlight - Controllable Resource Discovery
tags: [Feature Spotlight]
---

Coming in 1.1.13 is a new option for location constraints: `resource-discovery`

This new option controls whether or not Pacemaker performs resource
discovery for the specified resource on nodes covered by the constraint.
The default `always`, preserves the pre-1.1.13 behaviour. 

The options are:

* `always` - _(Default)_ Always perform resource discovery for the specified
  resource on this node.

* `never` - Never perform resource discovery for the specified
resource on this node.  This option should generally be used with a
`-INFINITY` score. Although that is not strictly required.

* `exclusive` - Only perform resource discovery for the specified
resource on this node. Multiple location constraints using `exclusive`
discovery for the same resource across different nodes creates a
subset of nodes `resource-discovery` is exclusive to. If a resource is
marked for `exclusive` discovery on one or more nodes, that resource
is only allowed to be placed within that subset of nodes.

## Why would I want this? 

Limiting resource discovery to a subset of nodes the resource is
physically capable of running on can significantly boost performance
when a large set of nodes are preset.  When `pacemaker_remote` is in
use to expand the node count into the 100s of nodes range, this option
can have a dramatic affect on the speed of the cluster.

## Is using this option ever a bad idea?

_Absolutely!_

Setting this option to `never` or `exclusive` allows the possibility
for the resource to be active in those locations without the cluster's
knowledge.  This can lead to the resource being active in _more than
one location!_


There are primarily three ways for this to happen:

1. If the service is started outside the cluster's control (ie. at
   boot time by `init`, `systemd`, etc; or by an admin)
1. If the `resource-discovery` property is changed while part of the cluster is
   down or suffering split-brain
1. If the `resource-discovery` property is changed for a resource/node while
   the resource is active on that node

## When is it safe?

For the most part, it is only appropriate when:

1. you have more than 8 nodes (including bare metal nodes with pacemaker-remoted), and
1. there is a way to guarentee that the resource can only run in a particular location (eg. the required software is not installed anywhere else)

## Want to know more?

Drop by [IRC](irc://freenode.org#linux) or ask us a question on the [Pacemaker mailing list](http://clusterlabs.org/wiki/Mailing_lists)

There is also plenty of [documentation](http://clusterlabs.org/doc) available.
