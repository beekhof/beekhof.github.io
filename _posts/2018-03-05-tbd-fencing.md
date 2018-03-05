---

date: 2018-03-05 13:11
title: A New Fencing Mechanism for Pacemaker (TBD)
published: false
excerpt: For some applications the loss of a database connection means game over.
category:
tags: 
- cluster
- fencing
- concepts
- cinder
- openstack
---


In the same way that some application require the ability to persist records
to disk, for some applications the loss of access to the database means game
over.

Cinder-volume is one such application and while discussing various options for
preventing SPoFs, I landed on the idea of using the database itself as a
mechanism to ensure consistent access.

As cinder-volume moves towards an active/active model, it is important that a
failure in one peer does not represent a SPoF.  In the Cinder architecture,
the API server has no way to know if the cinder-volume process is fully
functional.

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
some kind of storage involved, we explored the idea of leveraging the SBD[[1]](#fnote1)
(Storage Based Death) project's capabilities to do disk based heartbeating and
poison-pills.  When combined with a hardware watchdog, it is an extremely
reliable way to ensure safe access to shared resources.

However in Cinder's case, not all vendors can provide raw access to a small
block device on the storage.  Additionally, it is really access to the
database that needs protecting not the storage.  So while useful, it is still
possible to construct scenarios that would defeat SBD.

Enter TBD, ("Table Based Death", or "To Be Decided" depending on your
preference).  Instead of heartbeating to a designated slot on a block device,
the slots become rows in a small table in the database.  Preferrably one in a
different namespace and with different access permissions.

The desired behaviour is derived from the following properties:

1. Quorum is required to write poison pills into a peer's slot

2. A peer that finds a poison pill in its slot triggers its watchdog and reboots

3. A peer that looses connection to the database won't be able to write status
   information[[2]](#fnote2) to its slot which will trigger the watchdog

4. A peer that looses connection to the database won't be able to write a poison
   pill into another peer's slot

5. If the underlying database looses too many peers and reverts to read-only,
   we won't be able to write to our slot which triggers the watchdog

6. When a peer that looses connection to its peers, the survivors would maintain
   quorum(1) and write a poison pill to the lost node (1) ensuring the peer will
   terminate due to scenario (2) or (3)


If `N` seconds is the worst case time a peer would need to either notice a
poison pill, or disconnection from the database, and trigger the watchdog.
Then we can arrange for the cluster to recover services after some multiple of
`N` has elasped in the same way that Pacemaker does for SBD.

## Footnotes

<a name="fnote1">[1]</a> SBD lives under the ClusterLabs banner but can
operate without a traditional corosync/pacemaker stack.

<a name="fnote2">[2]</a> By requiring that peers periodically write their
status to a slot in the table, we are effectively creating a side-channel
heartbeat mechanism that can either allow the cluster to make better
assumptions, or allow TBD to initiate fencing without the co-operation of a
traditional cluster stack.
