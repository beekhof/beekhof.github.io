---

title: "Pacemaker and RHEL 6.4 (redux)"
date: 2013-11-04 09:33
comments: true
tags: 
---

The good news is that as of Novemeber 1st, [Pacemaker is now supported](https://rhn.redhat.com/errata/RHEA-2013-1493.html)
on RHEL 6.4 - with two caveats.

1. You must be using the updated `pacemaker`, `resource-agents` and `pcs` packages
1. You must be using CMAN for membership and quorum ([background](/blog/2012/pacemaker-and-cluster-filesystems/))

Technically, support is currently limited to Pacemaker's use in the
context of OpenStack.  In practice however, any bug that can be shown
to affect OpenStack deployments has a good chance of being fixed.

Since a cluster with no services is rather pointless, the _heartbeat_
OCF agents are now also [officially supported](https://rhn.redhat.com/errata/RHEA-2013-1494.html).
However, as Red Hat's policy is to only ship supported agents, some
agents are not present for this initial release.

The three primary reasons for not shipping agents were:

1. The software the agent controls is not shipped in RHEL
1. Insufficient experience to provide support
1. Avoiding agent duplication

[Filing bugs](https://bugzilla.redhat.com/enter_bug.cgi?product=Red%20Hat%20Enterprise%20Linux%206) is definitly the best way to get agents in the second
categories prioritized for inclusion.

Likewise, if there is no shipping agent that provides the
functionality of agents in the third category (IPv6addr and IPaddr2
might be an example here), filing bugs is the best way to get that
fixed.

In the meantime, since most of the agents are just shell scripts,
downloading the latest upstream agents is a viable work-around in most
cases.  For example:

        agents="Raid1 Xen"
        for a in $agents; do wget -O /usr/lib/ocf/resource.d/heartbeat/$a https://github.com/ClusterLabs/resource-agents/raw/master/heartbeat/$a; done
