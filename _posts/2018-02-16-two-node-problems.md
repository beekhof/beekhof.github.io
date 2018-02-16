---
date: 2018-02-16 10:52
title: Two Nodes - The Devil is in the Details
excerpt: Many people love 2-nodes because they seem conceptually simpler and 33% cheaper, but most will have subtle failure modes
tags:
- high availability
---

_tl;dr - Many people love 2-nodes because they seem conceptually simpler and 33%
cheaper, but while it's possible to construct good ones, most will have subtle
failure modes_


The first step towards creating any HA system is to look for and try to
eliminate single points of failure, often abbreviated as `SPoF`.

It is impossible to eliminate _all_ risk of downtime and especially when one
considers the additional complexity that comes with introducing additional
redunancy, concentrating on single (rather than chains of related and therefor
decreasingly probable) points of failure is widely accepted as a suitable
compromise.

The natural starting point then is to have more than one node.  However before
the system can move services to the surviving node after a failure, in
general, it needs to be sure that they are not still active elsewhere.

So not only are we looking for SPoFs, but we are also looking to balance risks
and consequences and the calculus will be different for every deployment [1](#fnote1)

There is no downside if a failure causes both members of a two node cluster to
serve up the same static website. However its a very different story if it
results in **both sides are independently managing a shared job queue or
providing uncoordinated write access to a replicated database or shared
filesystem.**

So in order to prevent a single node failure from corrupting your data or
blocking recovery, we rely on something called fencing.

## Fencing

At it s heart, fencing turns a question _Can our peer cause data
corruption?_ into an answer _no_ by isolating it both from incoming
requests and persistent storage. The most common approach to fencing is to
power off failed nodes.

There are two categories of fencing which I will call _direct_ and _indirect_
but could equally be called _active_ and _passive_. Direct methods involve
action on the part of surviving peers, such interacting with an IPMI or iLO
device, whereas indirect relies on the failed node to somehow recognise it is
in an unhealthy state (or is at least preventing remaining members from
recovering) and signal a [hardware
watchdog](https://en.wikipedia.org/wiki/Watchdog_timer) to panic the machine.

Quorum helps in both these scenarios.

### Direct Fencing

In the case of direct fencing, we can use it to prevent fencing races when the
network fails.   By including the concept of quorum, there is enough
information in the system (even without connectivity to their peers) for nodes
to automatically know whether they should initiate fencing and/or recovery.

Without quorum, both sides of a network split will rightly assume the other is
dead and rush to fence the other. In the worst case, both sides succeed
leaving the entire cluster offline.   The next worse is a _death match_ , a
never ending cycle of nodes coming up, not seeing their peers, rebooting them
and initiating recovery only to be rebooted when their peer goes through the
same logic.

The problem with fencing is that the most commonly used devices become
inaccessible due to the same failure events we want to use them to recover
from. Most IPMI and iLO cards both loose power with the hosts they control and
by default use the same network that is causing the peers to believe the
others are offline.

Sadly the intricacies of IPMI and iLo devices is rarely a consideration at the
point hardware is being purchased.

### Indirect Fencing

Quorum is also crucial for driving indirect fencing and, when done right, can
allow survivors to safely assume that missing nodes have entered a safe state
after a defined period of time.

In such a setup, the watchdog's timer is reset every N seconds unless quorum
is lost. If the timer (usually some multiple of N) expires, then the machine
performs an ungraceful power off (not shutdown).

This is very effective but without quorum to drive it, there is insufficient
information from within the cluster to determine the difference between a
network outage and the failure of your peer. The reason this matters is that
without a way to differentiate between the two cases, you are forced to choose
a single behaviour mode for both.

The problem with choosing a single response is that there is no course of
action that both maximises availability and prevents corruption.

- If you choose to assume the peer is alive but it actually failed, then the
  cluster has unnecessarily stopped services.

- If you choose to assume the peer is dead but it was just a network outage,
  then the best case scenario is that you have signed up for some manual
  reconciliation of the resulting datasets.

No matter what heuristics you use, it is trivial to construct a single failure
that either leaves both sides running or where the cluster unnecessarily shuts
down the surviving peer(s). Taking quorum away really does deprive the cluster
of one of the most powerful tools in its arsenal.

Given no other alternative, the best approach is normally to sacrificing
availability. Making corrupted data highly available does no-one any good and
manually reconciling diverant datasets is no fun either.

## Quorum

Quorum sounds great right?

The only drawback is that in order to have it in a cluster with N members, you
need to be able to see `N/2 + 1` of your peers. Which is impossible in a two
node cluster after one node has failed.

Which finally brings us to the fundamental issue with two-nodes:

> quorum does not make sense in two node clusters, and
>
> without it there is no way to reliably determine a course of action that
> both maximises availability and prevents corruption

Even in a system of two nodes connected by a crossover cable, there is no way
to conclusively differentiate between a network outage and a failure of the
other node. Unplugging one end (who's likelihood is surely proportional to the
distance between the nodes) would be enough to invalidate any assumption that
link health equals peer node health.

## What Can Be Done?

Sometimes the client can't or wont make the additional purchase of a third
node and we need to look for alternatives.

### Option 1 - Add a Backup Fencing Method

A node's iLO or IPMI device represents a SPoF because, by definition, if it
fails the survivors cannot use it to put the node into a safe state. In a
cluster of 3 nodes or more, we can mitigate this a quorum calculation and a
hardware watchdog (an indirect fencing mechanism as previously discussed).  In
a two node case we must instead use _network power switches_ (aka. _power
distribution units_ or PDUs).

After a failure, the survivor first attempts to contact the primary (the
built-in iLO or IPMI) fencing device.  If that succeeds, recovery proceeds as
normal.  Only if the iLO/IPMI device fails is the PDU invoked and assuming it
succeeds, recovery can again continue.

Be sure to place the PDU on a _different network to the cluster traffic_,
otherwise a single network failure will prevent access to both fencing devices
and block service recovery.

You might be wondering at this point... _doesn't the PDU represent a single
point of failure?_ To which the answer is “definitely“.

If that risk concerns you, and you would not be alone, connect both peers to
two PDUs and tell your cluster software to use both when powering peers on and
off. Now the cluster remains active if one PDU dies, and would require a
second fencing failure of either the other PDU or an IPMI device in order to
block recovery.

### Option 2 - Add an Arbitrator

In some scenarios, although a backup fencing method would be technically
possible, it is politically challenging. Many companies like to have a degree
of separation between the admin and application folks, and security conscious
network admins are not always enthusiastic about handing over the usernames
and passwords to the PDUs.

In this case, the recommended alternative is to create a neutral third-party
that can supplement the quorum calculation.

In the event of a failure, a node needs to be able to see ether its peer or
the arbitrator in order to recover services. The arbitrator also includes to
act as a tie-breaker if both nodes can see the arbitrator but not each other.

This option needs to be paired with an indirect fencing method, such as a
watchdog that is configured to panic the machine if it looses connection to
its peer and the arbitrator. In this way, the survivor is able to assume with
reasonable confidence that its peer will be in a safe state after the watchdog
expiry interval.

The practical difference between an arbitrator and a third node is that the
arbitrator has a much lower footprint and can act as a tie-breaker for more
than one cluster.

### Option 3 - More Human Than Human

The final approach is for survivors to continue hosting whatever services they
were already running, but _not start any new ones_ until either the problem
resolves itself (network heals, node reboots) or a human takes on the
responsibility of manually confirming that the other side is dead.

### Bonus Option

Did I already mention you could add a third node?
We test those a lot :-)

## Two Racks

For the sake of argument, lets imagine I've convinced you the reader on the
merits of a third node, we must now consider the physical arrangement of the
nodes. If they are placed in (and obtain power from), the same rack, that too
represents a SPoF and one that cannot be resolved by adding a second rack.

If this is surprising, consider what happens when the rack with two nodes
fails and how the surviving node would differentiate between this case and a
network failure.

The short answer is that it can't and we're back to having all the problems of
the two-node case. Either the survivor:

- ignores quorum and incorrectly tries to initiate recovery during network
outages (whether fencing is able to complete is a different story and depends
on whether PDU is involved and if they share power with any of the racks), or

- respects quorum and unnecessarily shuts itself down when its peer fails

Either way, two racks is no better than one and the nodes must either be given
independant supplies of power or be distributed accross three (or more
depending on how many nodes you have) racks.

## Two Datacenters

By this point the more risk averse readers might be thinking about disaster
recovery. What happens when an asteroid hits the one datacenter with our three
nodes distributed across three different racks? Obviously _Bad Things(tm)_ but
depending on your needs, adding a second datacenter might not be enough.

Done properly, a second datacenter gives you a (reasonably) up-to-date and
consistent copy of your services and their data.  However just like the two-
node and two-rack scenarios, there is not enough information in the system to
both maximise availability and prevent corruption (or diverging datasets).
Even with three nodes (or racks), distributing them across only two
datacenters leaves the system unable to reliably make the correct decision in
the (now far more likely) event that the two sides cannot communicate.

Which is not to say that a two datacenters solution is never appropriate. It
is not uncommon for companies to _want_ a human in the loop before taking the
extraordinary step of failing over to a backup datacenter.  Just be aware that
if you want automated failure, you're either going to need a third datacenter
in order for quorum to make sense (either directly or via an arbitrator) or
find a way to reliably power fence an entire datacenter.



###### Footnotes

 <a name="fnote1"></a>[1] Not everyone needs redundant power companies with independent transmission
lines.  Which is a true story,  _and_ their paranoia paid off when their
monitoring detected one power company's transformer was failing.  The customer
was on the phone trying to warn the power company when it finally blew.
