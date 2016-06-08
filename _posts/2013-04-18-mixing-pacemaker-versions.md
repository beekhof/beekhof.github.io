---
layout: post
title: "Mixing Pacemaker versions"
date: 2013-04-19 9:47
comments: true
tags: [tips,internals]
---

When mixing Pacemaker versions, there are two factors that need to be
considered.  The first is obviously the package version - if that is
the same, then there is no problem.

If not, then the Pacemaker _feature set_ needs to be checked.  This
feature set increases far less regularly than the normal package
version.  Newer versions of Pacemaker expose this value in the output
of `pacemakerd --features`:

> $ pacemakerd --features  
> Pacemaker 1.1.9 (Build: 9048b7b)  
> Supporting v3.0.7:  generated-manpages agent-manpages ascii-docs publican-docs ncurses gcov libqb-logging libqb-ipc lha-fencing upstart systemd nagios  heartbeat corosync-native snmp

In this case, the feature set is `3.0.7` (major 3, minor 0, revision 7).

For older versions, you should refer to the definition of
_CRM_FEATURE_SET_ in crm.h, usually this will be located at
`/usr/include/pacemaker/crm/crm.h`.

If two packages or versions share the same feature set, then the
expectation is that they are fully compatible.  Any other behavior is
a bug which needs to be reported.

If the feature sets between two versions differ but have the _same
major value_ (ie. the 3 in 3.0.7 and 3.1.5), then they are said to be
_upgrade compatible_.

## What does _upgrade compatible_ mean?

When two versions are upgrade compatible, it means that they will
co-exist during a rolling upgrade but not on an extended or permanent
basis as the newer version requires all its peers to support API
feature(s) that the old one does not have.

The following two rules apply when mixing installations with different
feature sets:

* When electing a node to run the cluster (the Designated Co-ordinator
  or "DC"), the node with the lowest feature set always wins.
* The DC records its feature set in the CIB
* Nodes may not join the cluster if their feature set is less than the
  one recorded in the CIB

## Example

Consider node1 with a feature set of 3.0.7 and node2 with feature set
3.0.8... when node2 first joins the cluster, node1 will naturally
remain the DC.

However if node1 leaves the cluster, either by being shut down or due
to a failure, node2 will become the DC (as it is by itself and by
definition has the lowest feature set of any active node).

At this point, **node1 will be rejected if it attempts to rejoin the
cluster and will shut down**, as its feature set is lower than that of
the DC (node2).

## Is this happening to me?

If you are affected by this, you will see an error in the logs along the lines of:

    error: We can only support up to CRM feature set 3.0.7 (current=3.0.8)

In this case, the DC (node2) has feature set 3.0.8 but we (node1) only have 3.0.7.

To get these two nodes talking to each other again:

1. stop the cluster on both nodes
1. on both nodes, run:
       CIB_file=/path/to/cib.xml cibadmin -M -X '<cib crm_feature_set="3.0.7"/>'
1. start node1 and wait until it is elected as the DC
1. start node2
