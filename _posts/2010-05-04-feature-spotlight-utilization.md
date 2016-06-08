---

title: ! 'Feature Spotlight: Utilization'
tags:
- tips
---
New in 1.1 is the ability for Pacemaker to factor the system resources (RAM,
CPU, etc) into its placement algorithms.

First, simply define the system resources provided by your nodes. We'll use
_cores_ in this example, but you can literally use any name you care to dream
up.

    
    crm configure node pcmk-1 utilization cores=2
    crm configure node pcmk-2 utilization cores=4
    

Then we tell the cluster how many _cores_ are needed by each VM.

    
    crm configure primitive kvm-small ocf:heartbeat:VirtualDomain utilization cores=1
    crm configure primitive kvm-medium ocf:heartbeat:VirtualDomain utilization cores=2
    crm configure primitive kvm-big ocf:heartbeat:VirtualDomain utilization cores=3
    

Finally, we tell Pacemaker how to use the information:

    
    crm configure property placement-strategy=utilization
    

Now Pacemaker will ensure the load from your virtual machines will be
distributed "evenly" throughout the cluster - without the need for convoluted
sets of colocation constraints.

Download 1.1 today from: [http://www.clusterlabs.org/rpm-next/](http://www.clusterlabs.org/rpm-next/)

### Limitations

This type of problem Pacemaker is dealing with here is known as the [knapsack problem](http://en.wikipedia.org/wiki/Knapsack_problem) and falls into the
[NP-complete](http://en.wikipedia.org/wiki/NP-complete) category of computer
science problems - which is fancy way of saying "takes a really long time to
solve".

Clearly in a HA cluster, its not acceptable to spend minutes, let alone hours
or days, finding an optional solution while services remain unavailable.

So instead of trying to solve the problem completely, Pacemaker uses a _best
effort_ algorithm for determining which node should host a particular service.
This means it arrives at a "solution" much faster than traditional linear
programming algorithms, but my do so at the price of leaving some services
stopped.

In the contrived example above:

  * kvm-small would be allocated to pcmk-1
  * kvm-medium would be allocated to pcmk-2
  * kvm-large would remain inactive

Which is not ideal.

#### Strategies for Dealing with the Limitations

  * Ensure you have sufficient physical capacity It might sounds obvious, but if the physical capacity of your nodes is (close to) maxed out by the cluster under normal conditions, then failover isn't going to go well. Even without the Utilization feature, you'll start hitting timeouts and getting secondary "failures".
  * Build some buffer into the capabilities advertised by the nodes Advertise slightly more resources than we physically have on the (usually valid) assumption that a VM will not use 100% of the configured number of cores/RAM/etc _all_ the time. This practice is also known as "over commit".
  * Specify service priorities If the cluster is going to sacrifice services, it should be the ones you care (comparatively) about the least.

