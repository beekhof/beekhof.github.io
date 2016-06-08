---

title: Highly Available Data Corruption
tags:
---

> Whenever there is doubt, there is no doubt
>
> _- Robert De Niro, Ronin_

There is little point ensuring service continuity if the underlying data is
toast. Pacemaker makes use of a concept called STONITH to prevent this from
happening but many people don't understand what it is or why it is so
important.

### What is STONITH?

STONITH is an acronym for "Shoot The Other Node In The Head" and is a form of
fencing, a mechanism for isolating "bad" nodes from an otherwise correctly
functioning cluster.

STONITH usually takes the form of a power switch that can be remotely
controlled over the network, however other forms are also possible.

### Do I Need STONITH?

Almost always, the answer is _Yes!_

At the most basic level, if having one or more services active on more than
one cluster node is a problem, then you need STONITH.

About the only clusters that _do not_ fall into this category are those where
each machine has a non-overlapping data set or shares data that is kept in
sync _manually_ (using rsync or something similar).

As soon as shared storage is involved, STONITH is a must. This also applies to
clusters with automated synchronization, such as DRBD.

#### Thought Experiment

Consider a cluster with 3 nodes (lets call them A, B & C) connected to a SAN.
If a cleaner accidentally unplugs node A, it is safe for nodes B and C to
continue writing to the SAN. However, the cluster has no way to know this.

The cluster could assume that any node it can't see is safely dead, but what
if the node was simply cut-off from the network instead? Two nodes could
easily end up writing to the same file, causing corruption.

#### Quorum

It is true that, in this situation, node A would eventually loose quorum and
could be made to stop anything that might be accessing the SAN. This reduces
the window during which corruption can occur, but the possibility still
exists.

This is the problem that STONITH is designed to handle, it creates certainty
where there was doubt.

Fundamentally, STONITH provides an answer to the question: _Is it safe to
start cluster services yet?_

#### Disk-based Heartbeats

The cluster could also use the SAN as another communication media, however
this is not a generic solution as not all clusters have one (many opt for a
software solution like DRBD) and more importantly, disk-based communication is
not currently supported by the OpenAIS and Heartbeat.

#### Other Reasons to use STONITH

STONITH also has a role to play in the event that a clustered service cannot
be stopped. In this case, the cluster uses STONITH to force the whole node
offline, thereby making it safe to start the rogue service elsewhere.

### Things to Look for When Choosing a STONITH Device

It is crucial that the STONITH device can allow the cluster to differentiate
between a node failure and a network one.

The biggest mistake people make in choosing a STONITH device is to use remote
power switch (such as many onboard IPMI controllers) that shares power with
the node it controls. These devices create what is known as a single point of
failure (SPoF).

If the node looses power, the cluster cannot be sure if the node is really
offline, or active and suffering from a network fault. The cluster will try to
turn off the node, but that will fail because the STONITH device has also lost
power. The only safe thing to do in this situation is block and wait for
further input (such as the node coming back online).

For the same reason, anything that relies on the machine being active (such as
the SSH-based “device” used during testing) is also inappropriate.

Clever algorithms exist to improve the reliability of these devices, however
they can only ever be approximations - which defeats the point of using
STONITH in the first place.

### But I Can't Afford STONITH

After explaining STONITH to people, the most common objection is that they
can't afford it.

First and foremost, this is nonsense.

Sure you _could_ go crazy and spend thousands, but Google quickly turns up
devices for as low as $20. Even assuming that these devices are complete junk
and you need to pay 10x that to get something reliable, we're not exactly
talking about a second mortgage here.

But for the sake of argument, lets assume that the only suitable device for
your cluster costs $3,000 (twice as much as the most expensive device from
WTI).

Ask yourself (or your boss):

1. How much is that compared to the rest of your hardware budget?
1. _When_ data corruption occurs, what is the cost of having the servers offline while you restore sane data from a backup?
1. Was the last backup corrupted too?
1. Indeed, what is the cost of loosing all the changes (orders?) that occurred since that last good backup?

Does STONITH sound like a worthwhile investment now?
