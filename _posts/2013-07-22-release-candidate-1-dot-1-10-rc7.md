---
layout: post
title: "Release candidate: 1.1.10-rc7"
date: 2013-07-22 10:50
comments: true
tags:
- announce
---

Announcing the seventh [release candidate](https://github.com/ClusterLabs/pacemaker/releases/Pacemaker-1.1.10-rc7) for Pacemaker 1.1.10

This RC is a result of bugfixes to the policy engine, fencing daemon
and crmd.  We've squashed a bug involving constructing compressed
messages and stonith-ng can now recover when a configuration ordering
change is detected.

Please keep the bug reports coming in!

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

## Details - 1.1.10-rc7

<table>
  <tr><td><strong>Changesets&nbsp;</strong></td> <td>57</td></tr>
  <tr><td><strong>Diff</strong></td> <td>37 files changed, 414 insertions(+), 331 deletions(-)</td></tr>
</table>
<br/>

## Highlights
### Features added in Pacemaker-1.1.10-rc7

  + N/A

### Changes since Pacemaker-1.1.10-rc6

  + Bug cl#5168 - Prevent clones from being bounced around the cluster due to location constraints
  + Bug cl#5170 - Correctly support on-fail=block for clones
  + Bug cl#5164 - crmd: Fixes crmd crash when using pacemaker-remote
  + cib: The result is not valid when diffs fail to apply cleanly for CLI tools
  + cluster: Correctly construct the header for compressed messages
  + cluster: Detect and warn about node names with capitals
  + Core: remove the mainloop_trigger that are no longer needed.
  + corosync: Ensure removed peers are erased from all caches
  + cpg: Correctly free sent messages
  + crmd: Prevent messages for remote crmd clients from being relayed to wrong daemons
  + crmd: Properly handle recurring monitor operations for remote-node agent
  + crm_mon: Bug cl#5167 - Only print "stopped" node list for incomplete clone sets
  + crm_node: Return 0 if --remove passed
  + fencing: Correctly detect existing device entries when registering a new one
  + lrmd: Prevent use-of-NULL in client library
  + pengine: cl5164 - Fixes pengine segfault when calculating transition with remote-nodes.
  + pengine: Do the right thing when admins specify the internal resource instead of the clone
  + pengine: Re-allow ordering constraints with fencing devices now that it is safe to do so
