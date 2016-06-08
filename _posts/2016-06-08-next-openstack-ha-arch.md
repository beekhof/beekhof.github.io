---
date: 2016-06-07 14:06
title: Evolving the OpenStack HA Architecture
excerpt: A future revision of the HA architecture should limit Pacemaker involvement to services like Galera, Rabbit and the few remaining OpenStack services that can only run active/passive.
tags: 
- cluster
- architecture
- openstack
---

In the current OpenStack HA architecture used by Red Hat, SuSE and
others, Systemd is the entity in charge of starting and stopping most
OpenStack services.  Pacemaker exists as a layer on top, signalling
when this should happen, but Systemd is the part making it happen.

This is a valuable contribution for active/passive (A/P) services and
those that require all their dependancies be available during their
startup and shutdown sequences.  However as OpenStack matures, more
and more components are able to operate in an unconstrained
active/active capacity with little regard for the startup/shutdown
order of their peers or dependancies - making them well suited to be
managed by Systemd.

For this reason, a future revision of the HA architecture should limit
Pacemaker's involvement to core services like Galera and Rabbit as
well as the few remaining OpenStack services that run A/P.

This would be particularly useful as we look towards a containerised
future.  It both allows OpenStack to play nicely with the current
generation of container managers which lack Orchestration, as well as
reduces recovery and downtime by allowing for the maximum parallelism.

Divesting most OpenStack services from the cluster also removes
Pacemaker as a potential obstacle for moving them to WSGI.  It is
as-yet unclear if services will live under a single Apache instance or
many and the former would conflict with Pacemaker's model of starting,
stopping and monitoring services as individual components.

## Objection 1 - Pacemaker as an Alerting Mechanism

Using Pacemaker as an alerting mechanism for a large software stack is
of limited value.  Of course Pacemaker needs to know when a service
dies but it necessarily takes action straight away, not wait around to
see if there will be any others with which it can correlate a root
cause.

In large complex software stacks, the recovery and alerting components
should not be the same thing because they do and should operate on
different timescales.

Pacemaker also has no way to include the context of a failure in an
alert and thus no way to report the difference between Nova failing
and Nova failing because Keystone is dead.  Indeed Keystone being the
root cause could be easily lost in a deluge of notifications about the
failure of services that depend on it.

For this reason, as the number of services and dependancies grow,
Pacemaker makes a poor substitute for a well configured monitoring and
alerting system (such as Nagios or Sensu) that can also integrate
hardware and network metrics.

## Objection 2 - Pacemaker has better Monitoring

Pacemaker's native ability to monitor services is more flexible than
Systemd's which relies on a "PID up == service healthy" mode of
thinking (1).

However, just as Systemd is the entity performing the startup and
shutdown of most OpenStack services, it is also the one performing the
actual service health checks.

To actually take advantage of Pacemaker's monitoring capabilities, you
would need to write Open Cluster Framework (OCF) agents (2) for every
OpenStack service. While this would not take a rocket scientist to
achieve, it is an opportunity for the way services are started in a
clustered and non-clustered environment to diverge.

So while it may feel good to look at a cluster and see that Pacemaker
is configured to check the health of a service every N seconds, all
that really achieves is to sync Pacemaker's understanding of the
service with what Systemd already knew.  In practice, on average, this
ends up delaying recovery by N/2 seconds instead of making it faster.

## Bonus Round - Active/Passive FTW

Some people have the impression that A/P is a better or simpler mode
of operation for services and in this was justify the continued use of
Pacemaker to manage OpenStack services.

Support for A/P configurations is important, it allows us to make
applications that are in no way cluster-aware more available by
reducing the requirements on the application to almost zero.

However, the downside is slower recovery as the service must be
bootstrapped on the passive node, which implies increased downtime.
So at the point the service becomes smart enough to run in an
unconstrained A/A configuration, you are better off to do so - with or
without a cluster manager.

## Footnotes

1. Watchdog-like functionality is only a variation on this, it only
    tells you that the thread responsible for heartbeating to Systemd
    is alive and well - not if the APIs it exposes are functioning.

2. Think SYS-V init scripts with some extra capabilities and
    requirements particular to clustered/automated environment.  It's
    a standard historically supported by the Linux Foundation but
    hasn't caught on much since it was created in the late 90's.
