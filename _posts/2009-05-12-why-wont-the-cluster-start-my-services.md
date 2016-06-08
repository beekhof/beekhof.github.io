---
layout: post
title: Why Wont the Cluster Start my Services?
tags:
- tips
---
Its a common question and a worthy topic for an extended article. Here's the
steps I usually follow when diagnosing such issues.

### Is the cluster allowed to start services?

1. Check quorum status with _crm_mon --one-shot_

   [Quorum](http://en.wikipedia.org/wiki/Quorum) is a property of the
   cluster which is attained when more than half the number of known
   nodes is online.  Unlike Heartbeat, [OpenAIS](http://openais.org)
   based clusters don't pretend to have quorum when only one of a
   possible two nodes is available. In such situations, the cluster's
   default behavior is to ensure data integrity by stopping all
   services.

   Check the current value of the _no-quorum-policy_ option:

		crm_attribute -n no-quorum-policy -G


   If you don't have quorum, you can tell the cluster to ignore the
   loss of quorum and start resources anyway:

		crm_attribute -n no-quorum-policy -v ignore

   Be careful to ensure
   [STONITH](http://en.wikipedia.org/wiki/STONITH) is correctly
   configured before using the _ignore_ option.

1. Check if the cluster is managing services:

   Check the global default

		crm_attribute --type rsc_defaults -n is-managed -G

   Check the per-resource values

		cibadmin --query --xpath //nvpair[@name="is-managed"]

   Check the old location for the global default

		crm_attribute -n is-managed-default -G


   Look for any results indicating a value of _false_

1. Check target-role

   The _target-role_ setting controls what state the resource can
   achieve. The list of possible states is:

   * Stopped
   * Started
   * Slave
   * Master Look out for any places indicating a value of _Stopped_. In the case of master/slave resources that aren't being promoted, a value of _Started_ can also be problematic.

   Check the global default

		crm_attribute --type rsc_defaults -n target-role -G

   Check the per-resource values

		cibadmin --query --xpath //nvpair[@name="target-role"]

### Look for failures

1. You can see the list of failures in the crm_mon output:

		crm_mon --one-shot --operations

1. Another good source of information is _ptest_ which can simulate what the cluster would try to do.

		ptest --live-check -VVV

   Look for anything unusual in the output such as

   > WARN: unpack_rsc_op: Processing failed op drbd0:1_start_0 on nagios-clu2: unknown error

1. Check the logs

		ssh -l root nagios-clu2 -- grep drbd0:1 /var/log/messages

#### Cleaning up after failures

If you identified any failures above, you can instruct the cluster to "forget"
about them:

		crm_resource --cleanup --node nagios-clu2

This results in the resource history being erased on _nagios-clu2_. The
cluster will then attempt to start any services that were not already active.

> _NOTE:_ This will have little or no benefit if the underlying issue, the one
> that caused the resource to fail in the first place, has not been fixed. If
> the problem persists, the resource will simply return to a failed state and
> the cluster will still refuse to start it.

In a later article, I'll explain how the cluster can recover from transient
failures automatically by timing them out after a certain interval.
