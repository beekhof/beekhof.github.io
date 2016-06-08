---
layout: post
date: 2015-05-13 9:21
title: Adding Managed Compute Nodes to a Highly Available Openstack Control Plane
tags: 
- openstack
- compute node
- tips
- pacemaker remote
---

As previously announced on [RDO list](https://www.redhat.com/archives/rdo-list/2015-April/msg00008.html)
and [GitHub](https://github.com/beekhof/osp-ha-deploy/blob/master/ha-openstack.md#compute-nodes),
we now have a way to allow Pacemaker to manage compute nodes within a
single cluster while still allowing us to scale beyond corosync's
limits.

Having this single administrative domain then allows us to do clever
things like automated recovery of VMs running on a failed or failing
compute node.

The main difference with the previous deployment mode is that services
on the compute nodes are now managed and driven by the Pacemaker
cluster on the control plane.

The compute nodes do not become full members of the cluster and they
no longer require the full cluster stack, instead they run
`pacemaker_remoted` which acts as a conduit.


### Assumptions

We start by assuming you have a functional Juno or Kilo control plane
configured for HA and access to the `pcs` cluster CLI.

If you don't have this already, there is a decent guide on
[Github](https://github.com/beekhof/osp-ha-deploy/blob/master/ha-openstack.md)
for how to achieve this.

### Basics

We start by installing the required packages onto the compute nodes
from your faviorite provider:

    yum install -y openstack-nova-compute openstack-utils python-cinder openstack-neutron-openvswitch openstack-ceilometer-compute python-memcached wget openstack-neutron pacemaker-remote resource-agents pcs

While we're here, we'll also install some pieces that aren't in any
packages yet (do this on both the compute nodes and the control
plane):

    mkdir /usr/lib/ocf/resource.d/openstack/
    wget -O /usr/lib/ocf/resource.d/openstack/NovaCompute https://github.com/beekhof/osp-ha-deploy/raw/master/pcmk/NovaCompute
    chmod a+x /usr/lib/ocf/resource.d/openstack/NovaCompute 
    
    wget -O /usr/sbin/fence_compute https://github.com/beekhof/osp-ha-deploy/raw/master/pcmk/fence_compute
    chmod a+x /usr/sbin/fence_compute

Next, ***on one node*** generate a key that `pacemaker` on the control
plane with use to authenticate with `pacemaker-remoted` on the compute
nodes.

    dd if=/dev/urandom of=/etc/pacemaker/authkey bs=4096 count=1

Now copy that to all the other machines (control plane and compute nodes).

At this point we can enable and start `pacemaker-remoted` on the compute nodes:

    chkconfig pacemaker_remote on
    service pacemaker_remote start


Finally, copy `/etc/nova/nova.conf`, `/etc/nova/api-paste.ini`,
`/etc/neutron/neutron.conf`, and
`/etc/neutron/plugins/openvswitch/ovs_neutron_plugin.ini` from the
control plane to each of your compute nodes.  

If you're using `ceilometer`, you'll also want
`/etc/ceilometer/ceilometer.conf` from there too.

### Preparing the Control Plane

At this point, we need to take down the control plane in order to
safely update the cluster configuration.  We don't want things to be
bouncing around while we make large scale modifications.

    pcs resource disable keystone

Next we must tell the cluster to look for and run the existing control
plane services only on the control plane (and not the about to be
defined compute nodes).  We can automate this with the clever use of
scripting tools:

    for i in $(cibadmin -Q --xpath //primitive --node-path | tr ' ' '\n' | awk -F "id='" '{print $2}' | awk -F "'" '{print $1}' | uniq | grep -v "\-fence") ; do pcs constraint location $i rule resource-discovery=exclusive score=0 osprole eq controller ; done

### Defining the Compute Node Services

Now we can create the services that can run on the compute node.  We
create them in a disabled state so that we have a chance to can limit
where they can run before the cluster attempts to start them.

    pcs resource create neutron-openvswitch-agent-compute  systemd:neutron-openvswitch-agent --clone interleave=true --disabled --force
    pcs constraint location neutron-openvswitch-agent-compute-clone rule resource-discovery=exclusive score=0 osprole eq compute 
    
    pcs resource create libvirtd-compute systemd:libvirtd  --clone interleave=true --disabled --force
    pcs constraint location libvirtd-compute-clone rule resource-discovery=exclusive score=0 osprole eq compute
    
    pcs resource create ceilometer-compute systemd:openstack-ceilometer-compute --clone interleave=true --disabled --force
    pcs constraint location ceilometer-compute-clone rule resource-discovery=exclusive score=0 osprole eq compute

    pcs resource create nova-compute ocf:openstack:NovaCompute user_name=admin tenant_name=admin password=keystonetest domain=${PHD_VAR_network_domain} --clone interleave=true notify=true --disabled --force
    pcs constraint location nova-compute-clone rule resource-discovery=exclusive score=0 osprole eq compute

>**Please note**, a previous version of this post used:
>
>    pcs resource create nova-compute ocf:openstack:NovaCompute --clone interleave=true --disabled --force
>
>Make sure you use the new form

Now that the services and where they can be located is defined, we
specify the order in which they must be started.
 
    pcs constraint order start neutron-server-clone then neutron-openvswitch-agent-compute-clone require-all=false
    
    pcs constraint order start neutron-openvswitch-agent-compute-clone then libvirtd-compute-clone
    pcs constraint colocation add libvirtd-compute-clone with neutron-openvswitch-agent-compute-clone
    
    pcs constraint order start libvirtd-compute-clone then ceilometer-compute-clone
    pcs constraint colocation add ceilometer-compute-clone with libvirtd-compute-clone
    
    pcs constraint order start ceilometer-notification-clone then ceilometer-compute-clone require-all=false
    
    pcs constraint order start ceilometer-compute-clone then nova-compute-clone
    pcs constraint colocation add nova-compute-clone with ceilometer-compute-clone
    
    pcs constraint order start nova-conductor-clone then nova-compute-clone require-all=false

### Configure Fencing for the Compute nodes

At this point we need to define how compute nodes can be powered off
('fenced' in HA terminology) in the event of a failure.

I have an switched APC PUD, the configuration for which looks like this:

    pcs stonith create fence-compute fence_apc ipaddr=east-apc login=apc passwd=apc pcmk_host_map="east-01:2;east-02:3;east-03:4;"

But you might be using Drac or iLO, which would require you do define
one for each node, eg.

    pcs stonith create fence-compute-1 fence_ipmilan login="root" passwd="supersecret" ipaddr="192.168.1.1" pcmk_host_list="compute-1"
    pcs stonith create fence-compute-2 fence_ipmilan login="root" passwd="supersecret" ipaddr="192.168.1.2" pcmk_host_list="compute-2"
    pcs stonith create fence-compute-3 fence_ipmilan login="root" passwd="supersecret" ipaddr="192.168.1.3" pcmk_host_list="compute-3"

> Be careful when using devices that loose power with the hosts they
> control.  For such devices, a power failure and network failure look
> identical to the cluster which makes automated recovery unsafe.

### Obsolete Instructions

>**Please note**, a previous version of this post included the
>following instructions, however **they are no longer required**.
>
>Next we configure the integration piece that notifies `nova` whenever
>the cluster fences one of the compute nodes.  Adjust the following
>command to conform to your environment:
>
>    pcs --force stonith create fence-nova fence_compute domain=example.com login=admin tenant-name=admin passwd=keystonetest auth-url=http://vip-keystone:35357/v2.0/
>
>Use `pcs stonith describe fence_compute` if you need more information
>about any of the options.
>
>Finally we instruct the cluster that both methods are required to
>consider the host safely fenced.  Assuming the `fence_ipmilan` case,
>you would then configure:
>
>    pcs stonith level add 1 compute-1 fence-compute-1,fence-nova
>    pcs stonith level add 1 compute-2 fence-compute-2,fence-nova
>    pcs stonith level add 1 compute-3 fence-compute-3,fence-nova
>

### Re-enabling the Control Plane and Registering Compute Nodes

The location constraints we defined above reference node properties which we now define with the help of some scripting magic:

    for node in $(cibadmin -Q -o nodes | grep uname | sed s/.*uname..// | awk -F\" '{print $1}' | awk -F. '{print $1}'); do pcs property set --node ${node} osprole=controller; done

Connections to remote hosts are modelled as resources in Pacemaker.
So in order to add them to the cluster, we define a service for each
one and set the node property that allows it to run compute services.

Once again assuming the three compute nodes from earlier, we would
run:

    pcs resource create compute-1 ocf:pacemaker:remote
    pcs resource create compute-2 ocf:pacemaker:remote
    pcs resource create compute-3 ocf:pacemaker:remote
    
    pcs property set --node compute-1 osprole=compute
    pcs property set --node compute-2 osprole=compute
    pcs property set --node compute-3 osprole=compute

### Thunderbirds are Go!

The only remaining step is to re-enable all the services and run
`crm_mon` to watch the cluster bring them and the compute nodes up:

    pcs resource enable keystone
    pcs resource enable neutron-openvswitch-agent-compute
    pcs resource enable libvirtd-compute
    pcs resource enable ceilometer-compute
    pcs resource enable nova-compute

