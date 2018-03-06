---

date: 2018-03-05 13:11
title: A New Fencing Mechanism for Pacemaker (TBD)
published: false
excerpt: Protecting database centric applications in the absense of power fencing
category:
tags: 
- cluster
- fencing
- concepts
- cinder
- openstack
---

## Protecting Database Centric Applications

In the same way that some application require the ability to persist records
to disk, for some applications the loss of access to the database means game
over.

Cinder-volume is one such application and as it moves towards an active/active
model, it is important that a failure in one peer does not represent a SPoF.
In the Cinder architecture, the API server has no way to know if the cinder-
volume process is fully functional.

A cinder-volume process that has lost access to the database will still
receive requests from the API server and act on them, however it will not be
able to record the action's success or failure in the database.  For some
operations this is ok, if wasteful, because the operation will fail and be
retried. Deletion of something that was already deleted  is usually treated as
successful and re-attempted creation operationss will return a new volume,
however performing the same resize operation twice is problematic. 

Even the safe operations may never complete because the bad cinder-volume
process may end up getting the cleanup operations from its own failures, which
would result in additional failures.

Additionally, some Cinder drivers make use of locking.  For those drivers it
is just as crucial that any locks held by a faulty or hung peer can be
recovered within a finite period of time.  Hence the need for fencing.

Since power-based fencing is so dependant on node hardware and there is always
some kind of storage involved, the idea of leveraging the
SBD[[1]](#fnote1) ( [Storage Based Death](/blog/2015/sbd-fun-and-profit) )
project's capabilities to do disk based heartbeating and poison-pills is attractive.
When combined with a hardware watchdog, it is an extremely reliable way to ensure
safe access to shared resources.

However in Cinder's case, not all vendors can provide raw access to a small
block device on the storage.  Additionally, it is really access to the
database that needs protecting not the storage.  So while useful, it is still
relatively easy to construct scenarios that would defeat SBD.

## A New Type of Death

Where SBD uses storage APIs to protect applications persisting data to disk,
we could also have one based on SQL calls that did the same for Cinder-volume
and other database centric applications.

I therefor propose TBD - "Table Based Death" (or "To Be Decided" depending on
how you're wired).

Instead of heartbeating to a designated slot on a block device, the slots
become rows in a small table in the database that this new daemon would
interact with via SQL.

When a peer is connected to the database, a cluster manager like Pacemaker can
use a poison pill to fence the peer in the event of a network, node, or
resource level failure.  Should the peer ever loose quorum or its connection
to the database, surviving peers can assume with a degree of confidence that
it will self terminate via the watchdog after a known interval.

The desired behaviour can be derived from the following properties:

1. Quorum is required to write poison pills into a peer's slot

2. A peer that finds a poison pill in its slot triggers its watchdog and reboots

3. A peer that looses connection to the database won't be able to write status
   information to its slot which will trigger the watchdog

4. A peer that looses connection to the database won't be able to write a poison
   pill into another peer's slot

5. If the underlying database looses too many peers and reverts to read-only,
   we won't be able to write to our slot which triggers the watchdog

6. When a peer that looses connection to its peers, the survivors would maintain
   quorum(1) and write a poison pill to the lost node (1) ensuring the peer will
   terminate due to scenario (2) or (3)


If `N` seconds is the worst case time a peer would need to either notice a
poison pill, or disconnection from the database, and trigger the watchdog.
Then we can arrange for services to be recovered after some multiple of `N`
has elasped in the same way that Pacemaker does for SBD.

While TBD would be a valuable addition to a traditional cluster architecture,
it is also concievable that it could be useful in a stand-alone configuration.
Consideration should therefor be given during the design phase as to how best
consume membership, quorum, and fencing requests from multiple sources - not
just a particular application or cluster manager.

## Limitations

Just as in the SBD architecture, we need TBD to be configured to use the same
persistent store (database) as is being consumed by the applications it is
protecting.  This is crucial as it means the same criteria that enables the
application to function, also results in the node self-terminating if it cannot
be satisfied.

However for security reasons, the table would ideally live in a different
namespace and with different access permissions.

It is also important to note that significant design challenges would need to
be faced in order to protect applications managed by the same cluster that was
providing the highly available database being consumed by TBD.  Consideration
would particularly need to be given to the behaviour of TBD and the
applications it was protecting during shudown and cold-start scenarios.  Care
would need to be taken in order to avoid unnecessary self-fencing operations
and that failure responses are not impacted by correctly handling these
scenarios.

## Footnotes

<a name="fnote1">[1]</a> SBD lives under the ClusterLabs banner but can
operate without a traditional corosync/pacemaker stack.
