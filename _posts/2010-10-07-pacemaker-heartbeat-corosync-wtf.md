---

title: Pacemaker, Heartbeat, Corosync, WTF?
tags: 
---
One question I still get a lot is what all these projects are/do and how they
all relate.

Here is the list of the possible components that might make up a
[Pacemaker](http://www.clusterlabs.org) install is:

  * Pacemaker - Resource manager
  * Corosync - Messaging layer
  * Heartbeat - Also a messaging layer
  * Resource Agents - Scripts that know how to control various services

_Pacemaker_ is the thing that starts and stops services (like your database or
mail server) and contains logic for ensuring both that they're running, and
that they're **only** running in one location (to avoid data corruption).

But it can't do that without the ability to talk to instances of itself on the
other node(s), which is where [Heartbeat](http://linux-ha.org) and/or
[Corosync](http://corosync.org) come in.

Think of _Heartbeat_ and _Corosync_ as
[dbus](http://www.freedesktop.org/wiki/Software/dbus) but between nodes.
Somewhere that any node can throw messages on and know that they'll be
received by all its peers. This bus also ensures that everyone agrees who is
(and is not) connected to the bus and tells Pacemaker when that list changes.

For two nodes Pacemaker could just as easily use sockets, but beyond that the
complexity grows quite rapidly and is very hard to get right - so it really
makes sense to use existing components that have proven to be reliable.

You only need one of them though :-)

Finally, in order to avoid teaching Pacemaker about every possible service
that people might want to make highly available, we make use of the
[OCF](http://opencf.org/home.html) standard to hide the details in scripts -
which we call _Resource Agents_. Any series of command-line actions can be
easily turned into a resource agent by adding them to an existing [template](h
ttp://hg.clusterlabs.org/pacemaker/1.1/file/tip/extra/resources/Dummy).

However a collection of the most commonly useful ones are made available as
part of the [Resource Agents](http://www.linux-ha.org/wiki/Resource_Agents)
project.

And of course pre-built packages for all these come with most of the popular
Linux distributions, including Fedora, openSUSE, SLES >= 10, RHEL >= 6,
Debian, and Ubuntu.

