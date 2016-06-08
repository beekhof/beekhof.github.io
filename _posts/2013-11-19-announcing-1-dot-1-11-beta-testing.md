---

title: "Announcing 1.1.11 Beta Testing"
date: 2013-11-21 14:00
comments: true
tags: 
---

With over 400 updates since the release of 1.1.10, its time to start
thinking about a new release.

Today I have tagged [release candidate 1](https://github.com/ClusterLabs/pacemaker/releases/Pacemaker-1.1.11-rc1).
The most notable fixes include:

  + attrd: Implementation of a truely atomic attrd for use with corosync 2.x
  + cib: Allow values to be added/updated and removed in a single update
  + cib: Support XML comments in diffs
  + Core: Allow blackbox logging to be disabled with SIGUSR2
  + crmd: Do not block on proxied calls from pacemaker_remoted
  + crmd: Enable cluster-wide throttling when the cib heavily exceeds its target load
  + crmd: Use the load on our peers to know how many jobs to send them
  + crm_mon: add --hide-headers option to hide all headers
  + crm_report: Collect logs directly from journald if available
  + Fencing: On timeout, clean up the agent's entire process group
  + Fencing: Support agents that need the host to be unfenced at startup
  + ipc: Raise the default buffer size to 128k
  + PE: Add a special attribute for distinguishing between real nodes and containers in constraint rules
  + PE: Allow location constraints to take a regex pattern to match against resource IDs
  + pengine: Distinguish between the agent being missing and something the agent needs being missing
  + remote: Properly version the remote connection protocol
  + services: Detect missing agents and permission errors before forking
  + Bug cl#5171 - pengine: Don't prevent clones from running due to dependant resources
  + Bug cl#5179 - Corosync: Attempt to retrieve a peer's node name if it is not already known
  + Bug cl#5181 - corosync: Ensure node IDs are written to the CIB as unsigned integers

If you are a user of `pacemaker_remoted`, you should take the time to
read about changes to the [online wire protocol](http://blog.clusterlabs.org/blog/2013/changes-to-the-remote-wire-protocol/)
that are present in this release.

To build `rpm` packages for testing:

1. Clone the current sources:

       # git clone --depth 0 git://github.com/ClusterLabs/pacemaker.git
       # cd pacemaker

1. If you haven't already, install Pacemaker's dependancies

       [Fedora] # sudo yum install -y yum-utils
       [ALL]	# make rpm-dep

1. Build Pacemaker

       # make rc

1. Copy the rpms and deploy as needed
