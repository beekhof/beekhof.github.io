---
date: 2017-02-15 12:45
title: Containerizing Databases with Kubernetes and Stateful Sets 
excerpt: An examination of the canonical StatefulSet example for managing databases with Kubernetes from a rigorous HA perspective.
tags:
- high availability
- database
- openstack
- kubernetes
---

The [canonical example](https://kubernetes.io/docs/tutorials/stateful-application/run-replicated-stateful-application/)
for Stateful Sets with a replicated application in Kubernetes is a
database.

As someone looking at how to move foundational OpenStack services to
containers, and eventually Kubernetes in the future, this is great
news as databases are very typical of applications with complex
boostrap and recovery processes.

If we can successfully show Kubernetes managing a multi-master
database natively safely, the patterns would be broadly applicable and
there is one less reason to have a traditional cluster manager in such
contexts.

## TL;DR

Kubernetes today is arguably unsuitable for deploying databases UNLESS
the pod owner has the ability to verify the physical status of the
underlying hardware and is prepared to perform manual recovery in some
scenarios.

# General Comments

The example allows for `N` slaves but limits itself to a single master.

Which is absolutely a valid deployment, but does prevent us from
exploring some of the more interesting corner multi-master cases and
unfortunately from a HA perspective makes `pod 0` a single point of
failure because:

- although MySQL slaves can be easily promoted to masters, the
  containers do not expose such a mechanism, and even if they did

- writers are told to connect to `pod 0` explicitly rather than use the
  `mysql-read` service

So if the worker on which `pod 0` is running hangs or becomes
unreachable, you're out of luck.

The loss of this worker currently puts Kubernetes in a no-win
situation.  Either it does the safe thing (the current behaviour) and
prevents the pod from being recovered or the attached volume from
being accessed, leading to more downtime (because it requires an admin
to intervene) than a traditional HA solution.  Or it allows the pod to
be recovered, risking data corruption if the worker (and by inference,
the pod) is not completely dead.

## Ordered Bootstrap and Recovery

One of the more important capabilities of StatefulSets is that:

> `Pod N` cannot be recovered, created or destroyed until all pods
> `0` to `N-1` are active and healthy.

This allows container authors to make many simplifying assumptions
during bootstrapping and scaling events (such as who has the most
recent copy of the data at a given point).

Unfortunately, until we get [pod safety and termination
guarantees](https://github.com/kubernetes/kubernetes/pull/34160/files),
it means that if a worker node crashes or becomes unreachable, it's
pods are unrecoverable and any auto-scaling policies cannot be
enacted.

Additionally, the enforcement of this policy only happens at
scheduling time.

This means that if there is a delay enacting the scheduler's results,
an image must be downloaded, or an init container is part of the scale
up process, there is a significant period of time in which an existing
pod may die before new replicas can be constructed.

As I type this, the current status on my testbed demonstrates this
fragility:

>     # kubectl get pods
>     NAME                         READY     STATUS        RESTARTS   AGE
>     [...]
>     hostnames-3799501552-wjd65   0/1       Pending       0          4m
>     mysql-0                      2/2       Running       4          4d
>     mysql-2                      0/2       Init:0/2      0          19h
>     web-0                        0/1       Unknown       0          19h

As described, the feature suggests this state (`mysql-2` to be in the
`init` state while `mysql-1` is not active) can never happen.

While such behaviour remains possible, container authors must take
care to include logic to detect and handle such scenarios.  The
easiest course of action is to call `exit` and cause the container to
be re-scheduled.

The example partially addresses this race condition by bootstrapping
`pod N` from `N-1`.  This limits the impact of `pod N`'s failure to `pod
N+1`'s startup/recovery period.

It is easy to conceive of an extended solution that closed the window
completely by trying pods `N-1` to `0` in order until it found an active
peer to sync from.

## Extending the Pattern to Galera

All Galera peers are writable, which makes some aspects easier and
others more complicated.

Bootstrapping `pod 0` would require some logic to determine if it is
bootstrapping the cluster (`--wsrep=""`) or in recovery mode
(`--wsrep=all:6868,the:6868,peers:6868`) but special handling of `pod
0` has precedent and is not onerous.  The remaining pods would
unconditionally use `--wsrep=all:6868,the:6868,peers:6868`.

`pod 0` is no longer a single point of failure with respect to writes,
however the loss of the worker it is hosted on will continue to
inhibit scaling events until manually confirmed and cleaned up by an
admin.

Violations of the linear start/stop ordering could be significant if
they result from a failure of `pod 0` and occur while bootstrapping
`pod 1`. Further, if `pod 1` was stopped significantly earlier than
`pod 0`, then depending on the implementation details of Galera, it is
conceivable that a failure of `pod 0` while `pod 1` is synchronising
might result in either data loss or `pod 1` becoming out of sync.

## Removing Shared Storage 

One of the main reasons to choose a replicated database is that it
doesn't require shared storage.

Having mutiple slaves certainly assists read scalability and if we
modified the example to use multiple masters it would likely improve
write performance and failover times.  However having multiple copies
of the database on the same shared storage does not provide additional
redundancy over what the storage already provides - and that is
important to some customers.

While there are ways to give containers access to local storage,
attempting to make use of them for a replicated database is problematic:

- It is currently not possible to enforce that pods in a Stateful Set
  always run on the same node.

  Kubernetes does have the ability to assign [node
  affinity](https://kubernetes.io/docs/user-guide/node-selection/) for
  pods, however since the Stateful Sets are a template, there is no
  opportunity to specify a different `kubernetes.io/hostname` selector
  for each copy.

  As the example is written, this is particularly important for `pod 0`
  as it is the only writer and the only one guaranteed to have the
  most up-to-date version of the data.

  It is possible that to work-around this problem if the replica count
  exceeds the worker count and all peers were writable masters,
  however incorporating such logic into the pod would negate much of
  the benefit of using Stateful Sets.

- A worker going offline prevents the pod from being started.

  In the shared storage case, it was possible to manually verify the
  host was down, delete the pod and have Kubernetes restart it.

  Without shared storage this is no longer possible for `pod 0` as that
  worker is the only one with the data used to bootstrap the slaves.

  The only options are to bring the worker back, or manually alter the
  node affinities to have `pod 0` replace the slave on the worker with
  the most up-to-date one.

## Summing Up

While Stateful Sets may not satisfy those looking for data redundancy,
they are a welcome addition to Kubernetes that will require pod safety
and termination guarantees before they can really shine.  The example
gives us a glimpse of the future but arguably shouldn't be used in
production yet.

Those looking to manage a database with Kubernetes today would be
advised to use individual pods and/or vanilla ReplicaSets, need the
ability to verify the physical status of the underlying hardware and
should be prepared to perform manual recovery in some scenarios.

