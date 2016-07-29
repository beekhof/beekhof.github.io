---
date: 2016-07-24 13:20
title: HA for Composible Deployments of OpenStack 
excerpt: Composable roles are a hot topic, I present a proposal for how to accommodate cluster-managed services.
tags: 
- cluster
- architecture
- openstack
---

One of the hot topics for OpenStack deployments is _composable roles_
- the ability to mix-and-match which services live on which nodes.

This is mostly a solved problem for services not managed by the
cluster, but what of the services still managed by the cluster?

# Considerations

1. Scale up  
   
   Naturally we want to be able to add more capacity easily

2. Scale down  
   
   And have the option to take it away again if it is no longer necessary

3. Role re-assignment post-deployment  
   
   Ideally the task of taking capacity from one service and giving it
   to another would be a core capability and not require a node be
   nuked from orbit first.

4. Flexible role assignments  
   
   Ideally, the architecture would not impose limitations on how roles
   are assigned.  

   By allowing roles to be assigned on an ad-hoc basis, we can allow
   arrangements that avoid single-points-of-failure (SPoF) and
   potentially take better advantage of the available hardware.  For
   example:

    * node 1: galera and rabbit  
    * node 2: galera and mongodb  
    * node 3: rabbit and mongodb  

   This also has implications when just one of the roles needs to be
   scaled up (or down).  If roles become inextricably linked at
   install time, this requires every service in the group to scale
   identically - potentially resulting in higher hardware costs when
   there are services that cannot do so and must be separated.

   Instead, even if two services (lets say `galera` and `rabbit`) are
   originally assigned to the same set of nodes, this should imply
   nothing about how either of them can or should be scaled in the
   future.

   We want the ability to deploy a new `rabbit` server without
   requiring it host `galera` too.

# Scope

This need only apply to non-OpenStack services, however it could be
extended to those as well if you were unconvinced by my other [recent
proposal](/blog/2016/next-openstack-ha-arch).

At Red Hat, the list of services affected would be:

* HAProxy
* Any VIPs
* Galera
* Redis
* Mongo DB
* Rabbit MQ
* Memcached
* openstack-cinder-volume

Additionally, if the deployment has been configured to provide [Highly
Available Instances](https://access.redhat.com/articles/1544823):

* nova-compute-wait
* nova-compute
* nova-evacuate
* fence-nova

# Proposed Solution

In essance, I propose that there be a single native cluster,
consisting of between 3 (the minimum sane cluster size) and 16
(roughly Corosync's out-of-the-box limit) nodes, augmented by a
collection of zero-or-more remote nodes.

Both native and remote nodes will have roles assigned to them,
allowing Pacemaker to automagically move resources to the right
location based on the roles.

> Note that all nodes, both native and remote, can have zero-or-more
> roles and it is also possible to have a mixture of native and
> remote nodes assigned to the same role.

This will allow us, by changing a few flags (and potentially adding
extra remote nodes to the cluster) go from a fully collapsed
deployment to a fully segregated one - and not only at install time.

If installers wish to support it[^1], this architecture can cope with
roles being split out (or recombined) after deployment and of course
the cluster wont need to be taken down and resources will move as
appropriate.

Although there is no hard requirement that anything except the fencing
devices run on the native nodes, best practice would arguably dictate
that HAProxy and the VIPs be located there unless an external load
balancer is in use.

The purpose of this would be to limit the impact of a hypothetical
pacemaker-remote bug.  Should such a bug exist, by virtue of being the
gateway to all the other APIs, HAProxy and the VIPs are the elements
one would least want to be affected.

Some installers may even choose to enforce this in the configuration,
but "by convention" is probably sufficient.

# Implementation Details

The key to this implementation is Pacemaker's concept of [node
attributes](http://clusterlabs.org/doc/en-US/Pacemaker/1.1-pcs/html/Pacemaker_Explained/s-node-attributes.html)
and
[expressions](http://clusterlabs.org/doc/en-US/Pacemaker/1.1-pcs/html/Pacemaker_Explained/_node_attribute_expressions.html)
that make use of them.

Instance attributes can be created with commands of the form:

    pcs property set --node controller-0 proxy-role=true

> Note that this differs from the `osprole=compute/controller` scheme
> used in the [Highly Available
> Instances](https://access.redhat.com/articles/1544823) instructions.
> That arrangement wouldn't work here as each node may have serveral
> roles assigned to it.

Under the covers, the result in Pacemaker's configuration would look something like:

    <cib ...>
      <configuration>
        <nodes>
          <node id="1" uname="controller-0">
            <instance_attributes id="controller-0-attributes">
              <nvpair id="controller-0-proxy-role" name="proxy-role" value="true"/>
    ...

These attributes can then be referenced in [location
constraints](http://clusterlabs.org/doc/en-US/Pacemaker/1.1-pcs/html/Pacemaker_Explained/_deciding_which_nodes_a_resource_can_run_on.html)
that restrict the resource to a subset of the available nodes based on
[certain
criteria](http://clusterlabs.org/doc/en-US/Pacemaker/1.1-pcs/html/Pacemaker_Explained/_using_rules_to_determine_resource_location.html#_location_rules_based_on_other_node_properties)

For example, we would use the following for HAProxy:

    pcs constraint location haproxy-clone rule score=0 proxy-role eq true

which would create the following under the covers:

    <rsc_location id="location-haproxy" rsc="haproxy-clone">
      <rule id="location-haproxy-rule" score="0">
        <expression id="location-haproxy-rule-expr" attribute="proxy-role" operation="eq" value="true"/>
      </rule>
    </rsc_location>

Any node, native or remote, not meeting the criteria is automatically
eliminated as a possible host for the service.


Pacemaker also defines some node attributes automatically based on a
node's name and type.  These are also available for use in
constraints.  This allows us, for example, to force a resource such as
`nova-evacuate` to run on a "real" cluster node with the command:

    pcs constraint location nova-evacuate rule score=0 "#kind" ne remote

For deployments based on Pacemaker 1.1.15 or later, we can also
simplify the configuration by using pattern matching in our
constraints.

1. Restricting all the VIPs to nodes with the proxy role:

        <rsc_location id="location-haproxy-ips" resource-discovery="exclusive" rsc-pattern="^(ip-.*)"/>

2. Restricting `nova-compute` to compute nodes (assuming a
standardized naming convention is used):

        <rsc_location id="location-nova-compute-clone" resource-discovery="exclusive" rsc-pattern="nova-compute-(.*)"/>

# Final Result

This is what a fully active cluster would look like:

    9 nodes configured
    87 resources configured
    
    Online: [ overcloud-controller-0 overcloud-controller-1 overcloud-controller-2 ]
    RemoteOnline: [ overcloud-compute-0 overcloud-compute-1
    overcloud-compute-2 rabbitmq-extra-0 storage-0 storage-1 ]
    
     ip-172.16.3.4 (ocf::heartbeat:IPaddr2): Started overcloud-controller-0
     ip-192.0.2.17 (ocf::heartbeat:IPaddr2): Started overcloud-controller-1
     ip-172.16.2.4 (ocf::heartbeat:IPaddr2): Started overcloud-controller-2
     ip-172.16.2.5 (ocf::heartbeat:IPaddr2): Started overcloud-controller-0
     ip-172.16.1.4 (ocf::heartbeat:IPaddr2): Started overcloud-controller-1
     Clone Set: haproxy-clone [haproxy]
         Started: [ overcloud-controller-0 overcloud-controller-1
    overcloud-controller-2 ]
     Master/Slave Set: galera-master [galera]
         Slaves: [ overcloud-controller-0 overcloud-controller-1
    overcloud-controller-2 ]
     ip-192.0.3.30 (ocf::heartbeat:IPaddr2): Started overcloud-controller-2
     Master/Slave Set: redis-master [redis]
         Slaves: [ overcloud-controller-0 overcloud-controller-1
    overcloud-controller-2 ]
     Clone Set: mongod-clone [mongod]
         Started: [ overcloud-controller-0 overcloud-controller-1
    overcloud-controller-2 ]
     Clone Set: rabbitmq-clone [rabbitmq]
         Started: [ overcloud-controller-0 overcloud-controller-1
    overcloud-controller-2 rabbitmq-extra-0 ]
     Clone Set: memcached-clone [memcached]
         Started: [ overcloud-controller-0 overcloud-controller-1
    overcloud-controller-2 ]
     openstack-cinder-volume (systemd:openstack-cinder-volume): Started storage-0
     Clone Set: nova-compute-clone [nova-compute]
         Started: [ overcloud-compute-0 overcloud-compute-1 overcloud-compute-2 ]
     Clone Set: nova-compute-wait-clone [nova-compute-wait]
         Started: [ overcloud-compute-0 overcloud-compute-1 overcloud-compute-2 ]
     nova-evacuate (ocf::openstack:NovaEvacuate): Started overcloud-controller-0
     fence-nova (stonith:fence_compute): Started overcloud-controller-0
     storage-0 (ocf::pacemaker:remote): Started overcloud-controller-1
     storage-1 (ocf::pacemaker:remote): Started overcloud-controller-2
     overcloud-compute-0 (ocf::pacemaker:remote): Started overcloud-controller-0
     overcloud-compute-1 (ocf::pacemaker:remote): Started overcloud-controller-1
     overcloud-compute-2 (ocf::pacemaker:remote): Started overcloud-controller-2
     rabbitmq-extra-0 (ocf::pacemaker:remote): Started overcloud-controller-0

A small wish, but it would be nice if installers used meaningful names
for the VIPs instead the underlying IP addresses they manage.

[^1]: One reason they may not do so on day one, is the careful co-ordination that some services can require when there is no overlap between the old and new sets of nodes assigned to a given role. Galera is one specific case that comes to mind.
