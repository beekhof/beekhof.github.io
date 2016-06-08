---
layout: post
title: Pacemaker and Cluster Filesystems
tags: 
---
There is some confusion out there on how to use Pacemaker with the OCFS2 and
GFS2 cluster filesystems.

Section 8.1 and 8.2 of [Clusters from Scratch](http://www.clusterlabs.org/doc/en-US/Pacemaker/1.1-plugin/html/Clusters_from_Scratch/ch08.html) mentions
some of the issues involved in the context of CMAN, but the principles are
generally applicable.

The most important take-away, is that **it is very important that all parts of
the stack are making decisions based on the same membership and quorum data**.

There were/are three options to achieve this:

  1. have everyone talk to pacemaker 
  2. have everyone talk to cman
  3. have everyone talk to corosync

### Option 1 - Everyone Talks to Pacemaker

This option was written for and is maintained/supported by SUSE but didn't
really gain much traction outside of SLES. It also relies on a pacemaker
plugin that gets loaded into corosync/openais, something that is no longer
possible with corosync 2.x It briefly appeared upstream but once option 2
became possible, option 1 was removed (not by me).

Anyone not paying for OCS2 on SLES is probably best advised to move to option
2 or 3.

Requirements:

  * Filesystems supported: OCFS2
  * Corosync: 1.x
  * Pacemaker: any
  * Other: openais

### Option 2 - Everyone Talks to CMAN

This is what works on most distros (except openSUSE/SLES) today. By virtue of
being part of RHCS and its age, cman is available on most of today's
enterprise distros and is supported by OCFS2 and GFS2.

By modifying Pacemaker to support it, we gained the ability to use GFS2 and
OCFS2 "for free" - without the need for custom dlm, gfs and ocfs controld's.

Requirements:

  * Filesystems supported: GFS2, OCFS2
  * Corosync: 1.x
  * Pacemaker: 1.1.6 or later
  * Other: cman, openais

### Option 3 - Everyone Talks to Corosync 2.0

With RHEL6 to be the last hoorah for CMAN, this is where things are headed
upstream, however the only distro that ships this solution today is Fedora-17
(and shortly 18).

In this scenario, all components obtain membership and quorum directly from
corosync. So far OCFS2 is the only component that hasn't been updated to
support this - they're appear content to continue using their own messaging
and membership layer.

Requirements:

  * Filesystems supported: GFS2
  * Corosync: 2.x
  * Pacemaker: 1.1.7 or later
  * Other: none

### Which Is The Best Option For Me

If you're a SLES customer looking to use OCFS2, absolutely take the _Option 1_
route. For everyone else, although _Option 3_ is architecturally superior,
_Option 2_ is likely to be the safest approach for the next couple of years.

