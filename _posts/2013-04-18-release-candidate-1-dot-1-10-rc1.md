---

title: "Release candidate: 1.1.10-rc1"
date: 2013-04-18 09:32
comments: true
tags:
- announce
---

> A funny thing happened on the way to 1.1.9...

Between tagging it on the Friday, and announcing it on the following
Monday, people started actually testing it and found a couple of
problems.

Specifically, there were some significant memory leaks and problems in
a couple of areas that our unit and integration tests can't sanely
test.

So while 1.1.9 is out, it was never formally announced.  Instead we've
been fixing the reported bugs (as well as looking for a few more by
running valgrind on a live cluster) and preparing for 1.1.10.

Also, in an attempt to learn from previous mistakes, the new release
procedure involves release candidates.  If no blocker bugs are
reported in the week following a release candidate, it is re-tagged as
the official release.

So without further ado, here is the 1.1.9 release notes as well as what changed in 1.1.10-rc1.


## Details - 1.1.10-rc1

<table>
  <tr><td><strong>Changesets&nbsp;</strong></td> <td>143</td></tr>
  <tr><td><strong>Diff</strong></td> <td>104 files changed, 3327 insertions(+), 1186 deletions(-)</td></tr>
</table>
<br/>

## Highlights
### Features added in Pacemaker-1.1.10

  * crm_resource: Allow individual resources to be reprobed
  * mcp: Alternate Upstart job controlling both pacemaker and corosync
  * mcp: Prevent the cluster from trying to use cman even when it is installed

### Changes since Pacemaker-1.1.9

  * Allow programs in the haclient group to use CRM_CORE_DIR
  * cman: Do not unconditionally start cman if it is already running
  * core: Ensure custom error codes are less than 256
  * crmd: Clean up memory at exit
  * crmd: Do not update fail-count and last-failure for old failures
  * crmd: Ensure we return to a stable state if there have been too many fencing failures
  * crmd: Indicate completion of refresh to callers
  * crmd: Indicate completion of re-probe to callers
  * crmd: Only perform a dry run for deletions if built with ACL support
  * crmd: Prevent use-after-free when the blackbox is enabled
  * crmd: Suppress secondary errors when no metadata is found
  * doc: Pacemaker Remote deployment and reference guide
  * fencing: Avoid memory leak in can_fence_host_with_device()
  * fencing: Clean up memory at exit
  * fencing: Correctly filter devices when no nodes are configured yet
  * fencing: Correctly unpack device parameters before using them
  * fencing: Fail the operation once all peers have been exhausted
  * fencing: Fix memory leaks during query phase
  * fencing: Prevent empty call-id during notification processing
  * fencing: Prevent invalid read in parse_host_list()
  * fencing: Prevent memory leak when registering devices
  * crmd: lrmd: stonithd: fixed memory leaks
  * ipc: Allow unpriviliged clients to clean up after server failures
  * ipc: Restore the ability for members of the haclient group to connect to the cluster
  * legacy: cl#5148 - Correctly remove a node that used to have a different nodeid
  * legacy: Support "crm_node --remove" with a node name for corosync plugin (bnc#805278)
  * logging: Better checks when determining if file based logging will work
  * Pass errors from lsb metadata generation back to the caller
  * pengine: Do not use functions from the cib library during unpack
  * Prevent use-of-NULL when reading CIB_shadow from the environment
  * Skip WNOHANG when waiting after sending SIGKILL to child processes
  * tools: crm_mon - Print a timing field only if its value is non-zero
  * Use custom OCF_ROOT_DIR if requested
  * xml: Prevent lockups by setting a more reliable buffer allocation strategy
  * xml: Prevent use-after-free in cib_process_xpath()
  * xml: Prevent use-after-free when not processing all xpath query results

## Details - 1.1.9
<table>
  <tr><td><strong>Changesets&nbsp;</strong></td> <td>731</td></tr>
  <tr><td><strong>Diff</strong></td> <td>1301 files changed, 92909 insertions(+), 57455 deletions(-)</td></tr>
</table>
<br/>

## Highlights
### Features added in Pacemaker-1.1.9

  * corosync: Allow cman and corosync 2.0 nodes to use a name other than uname()
  * corosync: Use queues to avoid blocking when sending CPG messages
  * ipc: Compress messages that exceed the configured IPC message limit
  * ipc: Use queues to prevent slow clients from blocking the server
  * ipc: Use shared memory by default
  * lrmd: Support nagios remote monitoring
  * lrmd: Pacemaker Remote Daemon for extending pacemaker functionality outside corosync cluster.
  * pengine: Check for master/slave resources that are not OCF agents
  * pengine: Support a 'requires' resource meta-attribute for controlling whether it needs quorum, fencing or nothing
  * pengine: Support for resource container
  * pengine: Support resources that require unfencing before start

### Changes since Pacemaker-1.1.8

  * attrd: Correctly handle deletion of non-existant attributes
  * Bug cl#5135 - Improved detection of the active cluster type
  * Bug rhbz#913093 - Use crm_node instead of uname
  * cib: Avoid use-after-free by correctly support cib_no_children for non-xpath queries
  * cib: Correctly process XML diff's involving element removal
  * cib: Performance improvements for non-DC nodes
  * cib: Prevent error message by correctly handling peer replies
  * cib: Prevent ordering changes when applying xml diffs
  * cib: Remove text nodes from cib replace operations
  * cluster: Detect node name collisions in corosync
  * cluster: Preserve corosync membership state when matching node name/id entries
  * cman: Force fenced to terminate on shutdown
  * cman: Ignore qdisk 'nodes'
  * core: Drop per-user core directories
  * corosync: Avoid errors when closing failed connections
  * corosync: Ensure peer state is preserved when matching names to nodeids
  * corosync: Clean up CMAP connections after querying node name
  * corosync: Correctly detect corosync 2.0 clusters even if we don't have permission to access it
  * crmd: Bug cl#5144 - Do not updated the expected status of failed nodes
  * crmd: Correctly determin if cluster disconnection was abnormal
  * crmd: Correctly relay messages for remote clients (bnc#805626, bnc#804704)
  * crmd: Correctly stall the FSA when waiting for additional inputs
  * crmd: Detect and recover when we are evicted from CPG
  * crmd: Differentiate between a node that is up and coming up in peer_update_callback()
  * crmd: Have cib operation timeouts scale with node count
  * crmd: Improved continue/wait logic in do_dc_join_finalize()
  * crmd: Prevent election storms caused by getrusage() values being too close
  * crmd: Prevent timeouts when performing pacemaker level membership negotiation
  * crmd: Prevent use-after-free of fsa_message_queue during exit
  * crmd: Store all current actions when stalling the FSA
  * crm_mon: Do not try to render a blank cib and indicate the previous output is now stale
  * crm_mon: Fixes crm_mon crash when using snmp traps.
  * crm_mon: Look for the correct error codes when applying configuration updates
  * crm_report: Ensure policy engine logs are found
  * crm_report: Fix node list detection
  * crm_resource: Have crm_resource generate a valid transition key when sending resource commands to the crmd
  * date/time: Bug cl#5118 - Correctly convert seconds-since-epoch to the current time
  * fencing: Attempt to provide more information that just 'generic error' for failed actions
  * fencing: Correctly record completed but previously unknown fencing operations
  * fencing: Correctly terminate when all device options have been exhausted
  * fencing: cov#739453 - String not null terminated
  * fencing: Do not merge new fencing requests with stale ones from dead nodes
  * fencing: Do not start fencing until entire device topology is found or query results timeout.
  * fencing: Do not wait for the query timeout if all replies have arrived
  * fencing: Fix passing of parameters from CMAN containing '='
  * fencing: Fix non-comparison when sorting devices by priority
  * fencing: On failure, only try a topology device once from the remote level.
  * fencing: Only try peers for non-topology based operations once
  * fencing: Retry stonith device for duration of action's timeout period.
  * heartbeat: Remove incorrect assert during cluster connect
  * ipc: Bug cl#5110 - Prevent 100% CPU usage when looking for synchronous replies
  * ipc: Use 50k as the default compression threshold
  * legacy: Prevent assertion failure on routing ais messages (bnc#805626)
  * legacy: Re-enable logging from the pacemaker plugin
  * legacy: Relax the 'active' check for plugin based clusters to avoid false negatives
  * legacy: Skip peer process check if the process list is empty in crm_is_corosync_peer_active()
  * mcp: Only define HA_DEBUGLOG to avoid agent calls to ocf_log printing everything twice
  * mcp: Re-attach to existing pacemaker components when mcp fails
  * pengine: Any location constraint for the slave role applies to all roles
  * pengine: Avoid leaking memory when cleaning up failcounts and using containers
  * pengine: Bug cl#5101 - Ensure stop order is preserved for partially active groups
  * pengine: Bug cl#5140 - Allow set members to be stopped when the subseqent set has require-all=false
  * pengine: Bug cl#5143 - Prevent shuffling of anonymous master/slave instances
  * pengine: Bug rhbz#880249 - Ensure orphan masters are demoted before being stopped
  * pengine: Bug rhbz#880249 - Teach the PE how to recover masters into primitives
  * pengine: cl#5025 - Automatically clear failcount for start/monitor failures after resource parameters change
  * pengine: cl#5099 - Probe operation uses the timeout value from the minimum interval monitor by default (#bnc776386)
  * pengine: cl#5111 - When clone/master child rsc has on-fail=stop, insure all children stop on failure.
  * pengine: cl#5142 - Do not delete orphaned children of an anonymous clone
  * pengine: Correctly unpack active anonymous clones
  * pengine: Ensure previous migrations are closed out before attempting another one
  * pengine: Introducing the whitebox container resources feature
  * pengine: Prevent double-free for cloned primitive from template
  * pengine: Process rsc_ticket dependencies earlier for correctly allocating resources (bnc#802307)
  * pengine: Remove special cases for fencing resources
  * pengine: rhbz#902459 - Remove rsc node status for orphan resources
  * systemd: Gracefully handle unexpected DBus return types
  * Replace the use of the insecure mktemp(3) with mkstemp(3)
