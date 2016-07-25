---
date: 2016-07-24 13:20
title: Thoughts on HA for Multi-Subnet Deployments of OpenStack 
excerpt: Installing OpenStack in a spine-and-leaf network presents problems for high availability, these are some thoughts on what do to about it. 
tags: 
- cluster
- architecture
- openstack
---

In a normal deployment, in order to direct traffic to the same HAProxy
instance, Pacemaker will ensure that each VIP is configured on at most
one HAProxy machine.

However in a spine and leaf network architecture, the nodes are in
multiple subnets and there may be a limitation that the machines can
not be part of a common L3 network that the VIPs could be added to.

Once the traffic reaches HAProxy everything should JustWork(tm) -
modulo creating the appropriate networking rules, the problem is
getting it to the proxy.

The approach to dealing with this will need to differ based on the
latencies that can be gaurenteed between every nodes in the cluster.
At Red Hat, we define LAN-like latencies to be 2ms or better -
consistently and between every node that would make up the cluster.

# Low Latency Links

You have more flexibility in low latency scenarios as the cluster
software can operate as designed.

At a high level, the possible ways forward are:

1. Decide the benefit isn't worth it and create an L3 network just for
   the VIPs to live on.

2. Put all the controllers into a single subnet.
   
   Just be mindful of what will happen if the switch connecting them
   goes bad.

3. Replace the HAProxy/VIP portion of the architecture with a load
   balancer appliance that is accessible from the child subnets.

4. Move the HAProxy/VIP portion of the architecture into a dedicated
   3-node cluster load balancer that is accessible from the child
   subnets.
   
   The new cluster would need the list of controllers and some health
   checks which could be as simple as "is the controller up/down" or
   as complex as "is service X up/down".


Right now creating a load balancer near the network spine would have
to be an extra manual step for users of TripleO. However once
composable roles (the ability to mix and match which services go on
which nodes) are supported, it should be possible to install such a
configuration out of the box by placing three machines near the spine
and giving only them the "haproxy" role.

# Higher Latency

Corosync has very strict latency requirements of no more than 2ms for
any of its links.  Assuming your installer can deploy across subnets,
the existance of such a link would be a barrier to the creation of a
highly available dpeloyment.

To work around these requirements, we can use Pacemaker Remote to
extend Pacemaker's ability to manage services on nodes separated by
higher latency links.

In TripleO, the work needed to make this style of deployment possible
is already planned as part of our "HA for composable roles" design.

As per option 4 of the low latency case, such a deployment would
consist of a three node cluster containing only HAProxy and some
floating IPs.

The rest of the nodes that make up the traditional OpenStack
control-plane are managed as remote cluster nodes.  Meaning instead of
a traditional Corosync and Pacemaker stack, they have only the
pacemaker-remote daemon and do not participate in leader elections or
quorum calculations.

## External Load Balancers

If you you wish to use a dedicated load balancer, then the 3-node
cluster would just co-ordinate the actions of the remote nodes and not
host any services locally.

An installer may conceivably create them anyway but leave them
disabled to simplify the testing matrix.

# General Considerations

The creation of a separate subnet or set of subnets for fencing is
highly encouraged.

In general we want to avoid the possibility for a single network(ing)
failure that can take out communication to the both a set of nodes and
the device that can turn them off.

Everything is HA a trade-off between the chance of a particular
failure occurring and the consequences if it ever actually happens.
Everyone will likely draw the line in a different place based on their
risk aversion, all I can do is make recommendations based on my
background in this field.

