---
layout: post
title: Pacemaker 1.0.5 Released
tags:
- announce
---
I'm back from vacation so it's time for another Pacemaker bug-fix release.
Testing went flawlessly and so without further ado, here it is…

Pre-built packages for Pacemaker and it's immediate dependancies will be
available for openSUSE, SLES, Fedora, RHEL, CentOS from the [OpenSUSE Build
Service](http://software.opensuse.org/download/server:/ha-clustering) once it
catches up.

Debian users should check for updates [Martin's
repo](http://clusterlabs.org/wiki/Install#Debian) over the coming days and
Ubuntu fans can visit [LaunchPad](https://edge.launchpad.net/~ubuntu-ha-
maintainers/+archive/ppa) for 8.04 and 9.10 packages.

The [source tarball](http://hg.clusterlabs.org/pacemaker/stable-1.0/archive/46
2f1569a437.tar.bz2) is also available directly from Mercurial.

General installation instructions are available at from the [ClusterLabs
wiki](http://clusterlabs.org/wiki/Install).

### Release Statistics

## Details
<table><tr><td><strong>Changesets&nbsp;</strong></td> <td> 90 </td>
</tr><tr><td><strong>Diff</strong></td> <td>185 files changed, 11051 insertions(+), 1206
deletions(-)</td> </tr></table><br/>

### Project Administrivia

The next release will be in early September and there will focus on
performance. Specifically I'll be looking at CPU usage, logging and
performance in larger clusters (>8 nodes).

### Changes of note since Pacemaker-1.0.4

  * Update source tarball to revision: 462f1569a437 (stable-1.0) tip
  * High (bnc#507255): Tools: crm: implement date expressions
  * High: ais: Fix cluster connection when using corosync 1.0
  * High: ais: fix compilation against latest corosync
  * High: Build: Fix compilation when snmp and esmtp are not available
  * High: Core: Show help text and exit with rc 1 if option processing failed
  * High: crmd: Terminate if we are ever evicted from the membership
  * High: crmd: Unset any existing DC value before querying for a new one
  * High: lrm: Look in the correct location for stonith agents
  * High: PE: Bug 2160 - Dont shuffle clones due to colocation
  * High: PE: Bug bnc#515172 - Correctly process location constraint rules which contain multiple expressions
  * High: PE: Bug bnc#515172 - Fix the boolean-op attribute of rules
  * High: PE: Fix reload for master/slave resources
  * High: PE: New implementation of the resource migration (not stop/start) logic
  * High: PE: Only prevent migration if the clone dependancy is stopping/starting on the target node
  * High: Tools: crm: new display type (uppercase keywords)
  * High: Tools: crm: support for color output
  * High: Tools: crm_resource - Advertise --move instead of --migrate
  * High: Tools: Differentiate between --help and an unknown option
  * Medium: cib: Supply an empty status section for replace operations
  * Medium: crmd: Note that dc-deadtime can be used to mask the brokeness of some switches
  * Medium: Extra: Add tools, an RA and tests for the System Health feature written by Mark Hamzy
  * Medium: Extra: New node connectivity RA that uses system ping and attrd_updater
  * Medium: PE: Prevent use-of-NULL in find_first_action()
  * Medium: Tools: crm: fix the verify exit code properly
  * Medium: Tools: crm_resource - Prevent use-of-NULL by requiring a resource name for the -A and -a options
