---
layout: post
title: "Release Candidate: 1.1.12-rc1"
date: 2014-05-07 20:16
comments: true
tags: 
- announce
---

As promised, this announcement brings the first release candidate for Pacemaker 1.1.12

   https://github.com/ClusterLabs/pacemaker/releases/Pacemaker-1.1.12-rc1

This release primarily focuses on important but mostly invisible changes under-the-hood:

- The CIB is now O(2) faster.  Thats 100x for those not familiar with Big-O notation :-)

  This has massively reduced the cluster's use of system resources,
  allowing us to scale further on the same hardware, and dramatically
  reduced failover times for large clusters.

- Support for ACLs are is enabled by default.

  The new implementation can restrict cluster access for containers
  where pacemaker-remoted is used and is also more efficient.

- All CIB updates are now serialized and pre-synchronized via the
  corosync CPG interface.  This makes it impossible for updates to be
  lost, even when the cluster is electing a new DC.

- Schema versioning changes

  New features are no longer silently added to the schema.  Instead
  the ${Y} in pacemaker-${X}-${Y} will be incremented for simple
  additions, and ${X} will be bumped for removals or other changes
  requiring an XSL transformation.

  To take advantage of new features, you will need to updates all the
  nodes and then run the equivalent of `cibadmin --upgrade`.


Thankyou to everyone that has tested out the new CIB and ACL code
already.  Please keep those bug reports coming in!


List of known bugs to be investigating during the RC phase:

- 5206	Fileencoding broken
- 5194	A resource starts with a standby node. (Latest attrd does not serve as the crmd-transition-delay parameter)
- 5197	Fail-over is delayed. (State transition is not calculated.)
- 5139	Each node fenced in its own transition during start-up fencing
- 5200	target node is over-utilized with allow-migrate=true
- 5184	Pending probe left in the cib
- 5165	Add support for transient node utilization attributes


To build `rpm` packages for testing:

1. Clone the current sources:

      # git clone --depth 0 git://github.com/ClusterLabs/pacemaker.git
      # cd pacemaker

1. Install dependancies (if you haven't already)

      [Fedora] # sudo yum install -y yum-utils
      [ALL]	# make rpm-dep

1. Build Pacemaker

      # make rc

1. Copy and deploy as needed


## Details

Changesets: 633
Diff: 184 files changed, 12690 insertions(+), 5843 deletions(-)

## Highlights

### Features added since Pacemaker-1.1.11
  + Changes to the ACL schema to support nodes and unix groups
  + cib: Check ACLs prior to making the update instead of parsing the diff afterwards
  + cib: Default ACL support to on
  + cib: Enable the more efficient xml patchset format
  + cib: Implement zero-copy status update (performance)
  + cib: Send all r/w operations via the cluster connection and have all nodes process them
  + crm_mon: Display brief output if "-b/--brief" is supplied or 'b' is toggled
  + crm_ticket: Support multiple modifications for a ticket in an atomic operation
  + Fencing: Add the ability to call stonith_api_time() from stonith_admin
  + logging: daemons always get a log file, unless explicitly set to configured 'none'
  + PE: Automatically re-unfence a node if the fencing device definition changes
  + pengine: cl#5174 - Allow resource sets and templates for location constraints
  + pengine: Support cib object tags
  + pengine: Support cluster-specific instance attributes based on rules
  + pengine: Support id-ref in nvpair with optional "name"
  + pengine: Support per-resource maintenance mode
  + pengine: Support site-specific instance attributes based on rules
  + tools: Display pending state in crm_mon/crm_resource/crm_simulate if --pending/-j is supplied (cl#5178)
  + xml: Add the ability to have lightweight schema revisions
  + xml: Enable resource sets in location constraints for 1.2 schema
  + xml: Support resources that require unfencing

### Changes since Pacemaker-1.1.11
  + acl: Authenticate pacemaker-remote requests with the node name as the client
  + cib: allow setting permanent remote-node attributes
  + cib: Do not disable cib disk writes if on-disk cib is corrupt
  + cib: Ensure 'cibadmin -R/--replace' commands get replies
  + cib: Fix remote cib based on TLS
  + cib: Ingore patch failures if we already have their contents
  + cib: Resolve memory leaks in query paths
  + cl#5055: Improved migration support.
  + cluster: Fix segfault on removing a node
  + controld: Do not consider the dlm up until the address list is present
  + controld: handling startup fencing within the controld agent, not the dlm
  + crmd: Ack pending operations that were cancelled due to rsc deletion
  + crmd: Actions can only be executed if their pre-requisits completed successfully
  + crmd: Do not erase the status section for unfenced nodes
  + crmd: Do not overwrite existing node state when fencing completes
  + crmd: Do not start timers for already completed operations
  + crmd: Fenced nodes that return prior to an election do not need to have their status section reset
  + crmd: make lrm_state hash table not case sensitive
  + crmd: make node_state erase correctly
  + crmd: Prevent manual fencing confirmations from attempting to create node entries for unknown nodes
  + crmd: Prevent memory leak in error paths
  + crmd: Prevent memory leak when accepting a new DC
  + crmd: Prevent message relay from attempting to create node entries for unknown nodes
  + crmd: Prevent SIGPIPE when notifying CMAN about fencing operations
  + crmd: Report unsuccessful unfencing operations
  + crm_diff: Allow the generation of xml patchsets without digests
  + crm_mon: Allow the file created by --as-html to be world readable
  + crm_mon: Ensure resource attributes have been unpacked before displaying connectivity data
  + crm_node: Only remove the named resource from the cib
  + crm_node: Prevent use-after-free in tools_remove_node_cache()
  + crm_resource: Gracefully handle -EACCESS when querying the cib
  + fencing: Advertise support for reboot/on/off in the metadata for legacy agents
  + fencing: Automatically switch from 'list' to 'status' to 'static-list' if those actions are not advertised in the metadata
  + fencing: Correctly record which peer performed the fencing operation
  + fencing: default to 'off' when agent does not advertise 'reboot' in metadata
  + fencing: Execute all required fencing devices regardless of what topology level they are at
  + fencing: Pass the correct options when looking up the history by node name
  + fencing: Update stonith device list only if stonith is enabled
  + get_cluster_type: failing concurrent tool invocations on heartbeat
  + iso8601: Different logic is needed when logging and calculating durations
  + lrmd: Cancel recurring operations before stop action is executed
  + lrmd: Expose logging variables expected by OCF agents
  + lrmd: Merge duplicate recurring monitor operations
  + lrmd: Provide stderr output from agents if available, otherwise fall back to stdout
  + mainloop: Fixes use after free in process monitor code
  + make resource ID case sensitive
  + mcp: Tell systemd not to respawn us if we exit with rc=100
  + pengine: Allow container nodes to migrate with connection resource
  + pengine: cl#5186 - Avoid running rsc on two nodes when node is fenced during migration
  + pengine: cl#5187 - Prevent resources in an anti-colocation from even temporarily running on a same node
  + pengine: Correctly handle origin offsets in the future
  + pengine: Correctly search failcount
  + pengine: Default sequential to TRUE for resource sets for consistency with colocation sets
  + pengine: Delay unfencing until after we know the state of all resources that require unfencing
  + pengine: Do not initiate fencing for unclean nodes when fencing is disabled
  + pengine: Do not unfence nodes that are offline, unclean or shutting down
  + pengine: Fencing devices default to only requiring quorum in order to start
  + pengine: fixes invalid transition caused by clones with more than 10 instances
  + pengine: Force record pending for migrate_to actions
  + pengine: handles edge case where container order constraints are not honored during migration
  + pengine: Ignore failure-timeout only if the failed operation has on-fail="block"
  + pengine: Log when resources require fencing but fencing is disabled
  + pengine: Memory leaks
  + pengine: Unfencing is based on device probes, there is no need to unfence when normal resources are found active
  + Portability: Use basic types for DBus compatability struct
  + remote: Allow baremetal remote-node connection resources to migrate
  + remote: Enable migration support for baremetal connection resources by default
  + services: Correctly reset the nice value for lrmd's children
  + services: Do not allow duplicate recurring op entries
  + services: Do not block synced service executions
  + services: Fixes segfault associated with cancelling in-flight recurring operations.
  + services: Reset the scheduling policy and priority for lrmd's children without replying on SCHED_RESET_ON_FORK
  + services_action_cancel: Interpret return code from mainloop_child_kill() correctly
  + stonith_admin: Ensure pointers passed to sscanf() are properly initialized
  + stonith_api_time_helper now returns when the most recent fencing operation completed
  + systemd: Prevent use-of-NULL when determining if an agent exists
  + upstart: Allow comilation with glib versions older than 2.28
  + xml: Better move detection logic for xml nodes
  + xml: Check all available schemas when doing upgrades
  + xml: Convert misbehaving #define into a more easily understood inline function
  + xml: If validate-with is missing, we find the most recent schema that accepts it and go from there
  + xml: Update xml validation to allow '<node type=remote />'
