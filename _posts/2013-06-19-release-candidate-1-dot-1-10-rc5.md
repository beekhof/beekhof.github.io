---
layout: post
title: "Release candidate: 1.1.10-rc5"
date: 2013-06-19 10:32
comments: true
tags:
- announce
---

Lets try this again...
Announcing the fourth and a half [release candidate](/blog/2013/release-candidate-1-dot-1-10-rc1/) for Pacemaker 1.1.10

I previously tagged rc4 but ended up making several changes shortly
afterwards, so it was pointless to announce it.

This RC is a result of cleanup work in several ancient areas of the
codebase:

* A number of internal membership caches have been combined
* The three separate CPG code paths have been combined

As well as:

* Moving clones is now both possible and sane
* Improved behavior on systemd based nodes
* and other assorted bugfixes (see below)  

Please keep the bug reports coming in!

Help is specifically requested for testing plugin-based clusters,
ACLs, the new --ban and --clear commands, and admin actions (such as
moving and stopping resources, calls to stonith_admin) which are hard
to test in an automated manner.

Also any light that can be shed on possible memory leaks would be much
appreciated.

If everything looks good in a week from now, I will re-tag rc5 as final. 

To build `rpm` packages for testing:

1. Clone the current sources:

       # git clone --depth 0 git://github.com/ClusterLabs/pacemaker.git
       # cd pacemaker

1. Install dependancies

       [Fedora] # sudo yum install -y yum-utils
       [ALL]	# make rpm-dep

1. Build Pacemaker

       # make rc

1. Copy and deploy as needed

## Details - 1.1.10-rc5

<table>
  <tr><td><strong>Changesets&nbsp;</strong></td> <td>168</td></tr>
  <tr><td><strong>Diff</strong></td> <td>96 files changed, 4983 insertions(+), 3097 deletions(-)</td></tr>
</table>
<br/>

## Highlights
### Features added in Pacemaker-1.1.10-rc5

  + crm_error: Add the ability to list and print error symbols
  + crm_resource: Allow individual resources to be reprobed
  + crm_resource: Implement --ban for moving resources away from nodes and --clear (replaces --unmove)
  + crm_resource: Support OCF tracing when using --force-(check|start|stop)
  + PE: Allow active nodes in our current membership to be fenced without quorum
  + Turn off auto-respawning of systemd services when the cluster starts them

### Changes since Pacemaker-1.1.10-rc3

  + Bug pengine: cl#5155 - Block the stop of resources if any depending resource is unmanaged
  + Convert all exit codes to positive errno values
  + Core: Ensure the blackbox is saved on abnormal program termination
  + corosync: Detect the loss of members for which we only know the nodeid
  + corosync: Do not pretend we know the state of nodes we've never seen
  + corosync: Nodes that can persist in sending CPG messages must be alive afterall
  + crmd: Do not get stuck in S_POLICY_ENGINE if a node we couldn't fence returns
  + crmd: Ensure all membership operations can complete while trying to cancel a transition
  + crmd: Everyone who gets a fencing notification should mark the node as down
  + crmd: Initiate node shutdown if another node claims to have successfully fenced us
  + crmd: Update the status section with details of nodes for which we only know the nodeid
  + crm_report: Find logs in compressed files
  + logging: If SIGTRAP is sent before tracing is turned on, turn it on
  + pengine: If fencing is unavailable or disabled, block further recovery for resources that fail to stop
  + remote: Workaround for inconsistent tls handshake behavior between gnutls versions
  + systemd: Ensure we get shut down correctly by systemd
