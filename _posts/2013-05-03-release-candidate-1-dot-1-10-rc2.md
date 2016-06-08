---
layout: post
title: "Release candidate: 1.1.10-rc2"
date: 2013-05-03 12:12
comments: true
tags:
- announce
---

Announcing the second [release candidate](/blog/2013/release-candidate-1-dot-1-10-rc1/) for Pacemaker 1.1.10

> No major changes have been introduced, just some fixes for a few
  niggles that were discovered since RC1.

Unless blocker bugs are found, this will be the final release
candidate and 1.1.10 will be tagged on May 10th.

Help is specifically requested for testing plugin-based clusters, ACLs
and admin actions (such as moving and stopping resources) which are
hard to test in an automated manner.

To build `rpm` packages for testing:

1. Clone the current sources:

       # git clone git://github.com/ClusterLabs/pacemaker.git
       # cd pacemaker

1. Install dependancies

   On Fedora/RHEL and its derivatives, you can do this by running:

       # yum install -y yum-utils
       # make yumdep

   Otherwise you will need to investigate the spec file and/or wait for `rpmbuild` to report missing packages.

1. Build Pacemaker

       # make rpm

1. Copy and deploy as needed

## Details - 1.1.10-rc2

<table>
  <tr><td><strong>Changesets&nbsp;</strong></td> <td>31</td></tr>
  <tr><td><strong>Diff</strong></td> <td>30 files changed, 687 insertions(+), 138 deletions(-)</td></tr>
</table>
<br/>

## Highlights
### Features added in Pacemaker-1.1.10-rc2

N/A

### Changes since Pacemaker-1.1.10-rc1

  * Bug cl#5152 - Correctly clean up fenced nodes during membership changes
  * Bug cl#5153 - Correctly display clone failcounts in crm_mon
  * Bug cl#5154 - Do not expire failures when on-fail=block is present
  * cman: Skip cman_pre_stop in the init script if fenced is not running
  * Core: Ensure the last field in transition keys is 36 characters
  * crm_mon: Check if a process can be daemonized before forking so the parent can report an error
  * crm_mon: Ensure stale pid files are updated when a new process is started
  * crm_report: Correctly collect logs when 'uname -n' reports fully qualified names
  * crm_resource: Allow --cleanup without a resource name
  * init: Unless specified otherwise, assume cman is in use if cluster.conf exists
  * mcp: inhibit error messages without cman
  * pengine: Ensure per-node resource parameters are used during probes
  * pengine: Implement the rest of get_timet_now() and rename to get_effective_time
