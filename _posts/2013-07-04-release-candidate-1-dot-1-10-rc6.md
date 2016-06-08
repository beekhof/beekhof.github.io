---
layout: post
title: "Release candidate: 1.1.10-rc6"
date: 2013-07-04 16:46
comments: true
tags:
- announce
---

Announcing the sixth [release candidate](https://github.com/ClusterLabs/pacemaker/releases/Pacemaker-1.1.10-rc6) for Pacemaker 1.1.10

This RC is a result of bugfixes in the policy engine, fencing daemon
and crmd.  Previous fixes in rc5 have also now been confirmed.
 
Help is specifically requested for testing plugin-based clusters,
ACLs, the --ban and --clear commands, and admin actions (such as
moving and stopping resources, calls to stonith_admin) which are hard
to test in an automated manner.

There is one bug open for David's remote nodes feature (involving
managing services on non-cluster nodes), but everything else seems
good.

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

## Details - 1.1.10-rc6

<table>
  <tr><td><strong>Changesets&nbsp;</strong></td> <td>63</td></tr>
  <tr><td><strong>Diff</strong></td> <td>24 files changed, 356 insertions(+), 133 deletions(-)</td></tr>
</table>
<br/>

## Highlights
### Features added in Pacemaker-1.1.10-rc6

  + tools:   crm_mon --neg-location drbd-fence-by-handler
  + pengine: cl#5128 - Support maintenance mode for a single node

### Changes since Pacemaker-1.1.10-rc5

  + cluster: Correctly remove duplicate peer entries
  + crmd: Ensure operations for cleaned up resources don't block recovery
  + pengine: Bug cl#5157 - Allow migration in the absence of some colocation constraints
  + pengine: Delete the old resource state on every node whenever the resource type is changed
  + pengine: Detect constraints with inappropriate actions (ie. promote for a clone)
  + pengine: Do the right thing when admins specify the internal resource instead of the clone
