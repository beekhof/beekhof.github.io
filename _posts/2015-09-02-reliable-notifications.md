---

date: 2015-09-01 12:16
title: Receiving Reliable Notification of Cluster Events
published: true
excerpt: A more reliable way to receive notification of cluster events is coming in Pacemaker 1.1.14.
category:
tags: 
- cluster
- concepts
---

When Pacemaker 1.1.14 arrives, there will be a more reliable way to
receive notification of cluster events.

In the past, we relied on a `ocf:pacemaker:ClusterMon` resource to
monitor the cluster status with the `crm_mon` daemon and trigger
alerts on each cluster event.

One of the arguments to `ClusterMon` was the location of a custom
script that would be called when the event happened.  This script
could then create an SNMP trap, SMS, email, etc to alert the admin
based on dynamically filled environment variables describing precisely
the cluster event that occurred.

### The Problem

Relying on a cluster resource proved to be not such a great idea for a number of reasons:

- Alerts ceased if the resource was not running
- There was no indication that the alerts had ceased
- The resource was likely to be stopped at exactly the point that something interesting was happening
- Old alerts were likely to be resent whenever the status section of the `cib` was rebuilt when a new DC was elected

### The Solution

Clearly support for notifications needed to be baked into the core of
Pacemaker, so thats what we've now done.  Finally (sorry, you wouldn’t
believe the length of my TODO list).

To make it work, drop a script onto each of the nodes,
`/var/lib/pacemaker/notify.sh` would be a good option, then tell the
cluster to start using it:

        pcs property set notification-agent=/var/lib/pacemaker/notify.sh

Like resource agents, this one can be written in whatever language
makes you happy - as long as it can read environment variables.

Pacemaker will also check your agent completed and report the return
code.  If the return code is not `0`, Pacemaker will also log any
output from your agent.

The agent is called asynchronously and should complete quickly. If it
has not completed after _5 minutes_ it will be terminated by the
cluster.

### Where to Send Alerts

I think we can all agree that hard coding the intended recipient of
the notification into the scripts would be a bad idea.  It would make
updating the recipient (vacation, change of role, change of employer)
annoying and prevent the scripts from being reused between different
clusters.

So there is also a `notification-recipient` cluster property which
will be passed to the script.  It can contain whatever you like, in
whatever format you like, as long as the `notification-agent` knows
what to do with it.

To get people started, the source includes a
[sample agent](https://github.com/ClusterLabs/pacemaker/blob/master/extra/pcmk_notify_sample.sh)
which assumes `notification-recipient` is a filename, eg.

        pcs property set notification-recipient=/var/lib/pacemaker/notify.log

### Interface

We preserved the old list of environment variables, so any existing
`ClusterMon` scripts will still work in this new mode.  I have added a
few extra ones though.

Environment variables common to all notification events:

- **CRM_notify_kind** _(New)_ Indicates the type of notification. One of `resource`, `node`, and `fencing`.
- **CRM_notify_version** _(New)_ Indicates the version of Pacemaker sending the notification.
- **CRM_notify_recipient** The value specified by `notification-recipient` from the cluster configuration.

Additional environment variables available for notification of node up/down events _(new)_:

- **CRM_notify_node** The node name for which the status changed
- **CRM_notify_nodeid** _(New)_ The node id for which the status changed
- **CRM_notify_desc** The current node state. One of `member` or `lost`.

Additional environment variables available for notification of fencing events (both successful and failed):

- **CRM_notify_node** The node for which the status changed.
- **CRM_notify_task** The operation that caused the status change.
- **CRM_notify_rc** The numerical return code of the operation.
- **CRM_notify_desc** The textual output relevant error code of the operation (if any) that caused the status change.

Additional environment variables available for notification of resource operations:

- **CRM_notify_node** The node on which the status change happened.
- **CRM_notify_rsc** The name of the resource that changed the status.
- **CRM_notify_task** The operation that caused the status change.
- **CRM_notify_interval** _(New)_ The interval of a resource operation
- **CRM_notify_rc** The numerical return code of the operation.
- **CRM_notify_target_rc** The expected numerical return code of the operation.
- **CRM_notify_status** The numerical representation of the status of the operation.
- **CRM_notify_desc** The textual output relevant error code of the operation (if any) that caused the status change.

