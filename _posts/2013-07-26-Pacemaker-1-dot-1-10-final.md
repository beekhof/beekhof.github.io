---
layout: post
title: "Pacemaker 1.1.10 - final"
date: 2013-07-26 10:11
comments: true
tags:
- announce
---

Announcing the release of [Pacemaker 1.1.10](https://github.com/ClusterLabs/pacemaker/releases/Pacemaker-1.1.10)

There were three changes of note since rc7:

  + Bug cl#5161 - crmd: Prevent memory leak in operation cache
  + cib: Correctly read back archived configurations if the primary is corrupted
  + cman: Do not pretend we know the state of nodes we've never seen

Along with assorted bug fixes, the major topics for this release were:

- stonithd fixes
- fixing memory leaks, often caused by incorrect use of glib reference counting
- supportability improvements (code cleanup and deduplication, standardized error codes)

Release candidates for the next Pacemaker release (1.1.11) can be
expected some time around Novemeber.

A big thankyou to everyone that spent time testing the release
candidates and/or contributed patches.  However now that Pacemaker is
perfect, anyone reporting bugs will be shot :-)

To build `rpm` packages:

1. Clone the current sources:

       # git clone --depth 0 git://github.com/ClusterLabs/pacemaker.git
       # cd pacemaker

1. Install dependancies (if you haven't already)

       [Fedora] # sudo yum install -y yum-utils
       [ALL]	# make rpm-dep

1. Build Pacemaker

       # make release

1. Copy and deploy as needed

## Details - 1.1.10 - final

<table>
  <tr><td><strong>Changesets&nbsp;</strong></td> <td>602</td></tr>
  <tr><td><strong>Diff</strong></td> <td>
  143 files changed, 8162 insertions(+), 5159 deletions(-)
  </td></tr>
</table>
<br/>

## Highlights

### Features added since Pacemaker-1.1.9

  + Core: Convert all exit codes to positive errno values
  + crm_error: Add the ability to list and print error symbols
  + crm_resource: Allow individual resources to be reprobed
  + crm_resource: Allow options to be set recursively
  + crm_resource: Implement --ban for moving resources away from nodes and --clear (replaces --unmove)
  + crm_resource: Support OCF tracing when using --force-(check|start|stop)
  + PE: Allow active nodes in our current membership to be fenced without quorum
  + PE: Suppress meaningless IDs when displaying anonymous clone status
  + Turn off auto-respawning of systemd services when the cluster starts them
  + Bug cl#5128 - pengine: Support maintenance mode for a single node

### Changes since Pacemaker-1.1.9

  + crmd: cib: stonithd: Memory leaks resolved and improved use of glib reference counting
  + attrd: Fixes deleted attributes during dc election
  + Bug cf#5153 - Correctly display clone failcounts in crm_mon
  + Bug cl#5133 - pengine: Correctly observe on-fail=block for failed demote operation
  + Bug cl#5148 - legacy: Correctly remove a node that used to have a different nodeid
  + Bug cl#5151 - Ensure node names are consistently compared without case
  + Bug cl#5152 - crmd: Correctly clean up fenced nodes during membership changes
  + Bug cl#5154 - Do not expire failures when on-fail=block is present
  + Bug cl#5155 - pengine: Block the stop of resources if any depending resource is unmanaged
  + Bug cl#5157 - Allow migration in the absence of some colocation constraints
  + Bug cl#5161 - crmd: Prevent memory leak in operation cache
  + Bug cl#5164 - crmd: Fixes crash when using pacemaker-remote
  + Bug cl#5164 - pengine: Fixes segfault when calculating transition with remote-nodes.
  + Bug cl#5167 - crm_mon: Only print "stopped" node list for incomplete clone sets
  + Bug cl#5168 - Prevent clones from being bounced around the cluster due to location constraints
  + Bug cl#5170 - Correctly support on-fail=block for clones
  + cib: Correctly read back archived configurations if the primary is corrupted
  + cib: The result is not valid when diffs fail to apply cleanly for CLI tools
  + cib: Restore the ability to embed comments in the configuration
  + cluster: Detect and warn about node names with capitals
  + cman: Do not pretend we know the state of nodes we've never seen
  + cman: Do not unconditionally start cman if it is already running
  + cman: Support non-blocking CPG calls
  + Core: Ensure the blackbox is saved on abnormal program termination
  + corosync: Detect the loss of members for which we only know the nodeid
  + corosync: Do not pretend we know the state of nodes we've never seen
  + corosync: Ensure removed peers are erased from all caches
  + corosync: Nodes that can persist in sending CPG messages must be alive afterall
  + crmd: Do not get stuck in S_POLICY_ENGINE if a node we couldn't fence returns
  + crmd: Do not update fail-count and last-failure for old failures
  + crmd: Ensure all membership operations can complete while trying to cancel a transition
  + crmd: Ensure operations for cleaned up resources don't block recovery
  + crmd: Ensure we return to a stable state if there have been too many fencing failures
  + crmd: Initiate node shutdown if another node claims to have successfully fenced us
  + crmd: Prevent messages for remote crmd clients from being relayed to wrong daemons
  + crmd: Properly handle recurring monitor operations for remote-node agent
  + crmd: Store last-run and last-rc-change for all operations
  + crm_mon: Ensure stale pid files are updated when a new process is started
  + crm_report: Correctly collect logs when 'uname -n' reports fully qualified names
  + fencing: Fail the operation once all peers have been exhausted
  + fencing: Restore the ability to manually confirm that fencing completed
  + ipc: Allow unpriviliged clients to clean up after server failures
  + ipc: Restore the ability for members of the haclient group to connect to the cluster
  + legacy: Support "crm_node --remove" with a node name for corosync plugin (bnc#805278)
  + lrmd: Default to the upstream location for resource agent scratch directory
  + lrmd: Pass errors from lsb metadata generation back to the caller
  + pengine: Correctly handle resources that recover before we operate on them
  + pengine: Delete the old resource state on every node whenever the resource type is changed
  + pengine: Detect constraints with inappropriate actions (ie. promote for a clone)
  + pengine: Ensure per-node resource parameters are used during probes
  + pengine: If fencing is unavailable or disabled, block further recovery for resources that fail to stop
  + pengine: Implement the rest of get_timet_now() and rename to get_effective_time
  + pengine: Re-initiate _active_ recurring monitors that previously failed but have timed out
  + remote: Workaround for inconsistent tls handshake behavior between gnutls versions
  + systemd: Ensure we get shut down correctly by systemd
  + systemd: Reload systemd after adding/removing override files for cluster services
  + xml: Check for and replace non-printing characters with their octal equivalent while exporting xml text
  + xml: Prevent lockups by setting a more reliable buffer allocation strategy
