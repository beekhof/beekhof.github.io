---

title: Pacemaker 1.0.6 Released
tags:
- announce
---
The next installment of the
[Pacemaker](http://www.clusterlabs.org/wiki/Pacemaker) 1.0 stable series is
now ready for general consumption.

In addition to further polishing of the crm shell and CLI tools, this is the
first release to support CoroSync (version 1.1.2 or greater is required).

The "Pacemaker Explained" reference has also been converted to docbook and is
included as part of the tarball (and pre-built packages if the relevant
stylesheets are present at build time).

Pre-built packages for Pacemaker and it's immediate dependancies will be
available for openSUSE, SLES, Fedora, RHEL, CentOS from the [OpenSUSE Build
Service](http://software.opensuse.org/download/server:/ha-clustering) in the
next couple of days depending in how overloaded it is.

Debian users should check for updates 
[Martin's repo](http://clusterlabs.org/wiki/Install#Debian) 
over the coming days and Ubuntu fans can visit
[LaunchPad](https://edge.launchpad.net/~ubuntu-ha-maintainers/+archive/ppa) 
for 8.04 and 9.10 packages.

The [source tarball](http://hg.clusterlabs.org/pacemaker/stable-1.0/archive/Pacemaker-1.0.6.tar.bz2) is also available directly from Mercurial.

General installation instructions are available at from the [ClusterLabs wiki](http://clusterlabs.org/wiki/Install).

### Release Statistics

<table><tr><td><strong>Changesets&nbsp;</strong></td> <td> 185 </td>
</tr><tr><td><strong>Diff</strong></td> <td> 331 files changed, 13858 insertions(+),
3277 deletions(-)</td> </tr></table><br/>

### Project Administrivia

We may switch to a bi-monthly release cycle. If you have any thoughts on this
(for or against), please get in touch.

### Changes of note since Pacemaker-1.0.5

  * High: cib: Correctly clean up when both plaintext and tls remote ports are requested
  * High: ais: Avoid excessive load by checking for dead children every 1s (instead of 100ms)
  * High: ais: Bug lf#2199 - Prevent expected-quorum-votes from being populated with garbage
  * High: ais: Bug rh#525589 - Prevent shutdown deadlocks when running on CoroSync
  * High: ais: Gracefully handle changes to the AIS nodeid
  * High: ais: Prevent deadlock - dont try to release IPC message if the connection failed
  * High: ais: Ubuntu needs a leading zero for directory modes
  * High: cib: For validation errors, send back the full CIB so the client can display the errors
  * High: cib: Prevent use-after-free for remote plaintext connections
  * High: cib: Repair the ability to connect to the cluster from non-cluster machines
  * High: Core: Bug lf#2169 - Allow dtd/schema validation to be disabled
  * High: crmd: Bug bnc#527530 - Wait for the transition to complete before leaving S_TRANSITION_ENGINE
  * High: crmd: Bug lf#2201 - Guard against possible cause of a segfault
  * High: crmd: Prevent use-after-free with LOG_DEBUG_3
  * High: Extras: Add sctp support to the controld RA
  * High: PE: Bug bnc#515172 - Provide better defaults for lt(e) and gt(e) comparisions
  * High: PE: Bug lf#2106 - Not all anonymous clone children are restarted after configuration change
  * High: PE: Bug lf#2170 - stop-all-resources option had no effect
  * High: PE: Bug lf#2171 - Prevent groups from starting if they depend on a complex resource which can't
  * High: PE: Bug lf#2197 - Allow master instances placemaker to be influenced by colocation constraints
  * High: PE: Disable resource management if stonith-enabled=true and no stonith resources are defined
  * High: PE: Don't include master score if it would prevent allocation
  * High: PE: Make sure promote/demote pseudo actions are created correctly
  * High: PE: Prevent target-role from promoting more than master-max instances
  * High: shell: Add allow-migrate as allowed meta-attribute (bnc#539968)
  * High: tools: bnc#547579,547582 - crm: status section editing support
  * High: Tools: crm: add semantic checks depending on the meta-data from resource agents
  * High: Tools: crm: improve processing of group edit and constraints
  * High: Tools: crm: improve the edit command
  * High: Tools: pingd - Fix a number of critical bugs (patch via Kazunori INOUE)
  * Med: xml: Mask the "symmetrical" attribute on rsc_colocation constraints (bnc#540672)
  * Medium (bnc#520707): Tools: crm: new templates ocfs2 and clvm
  * Medium (LF 2164): Tools: hb_report: expand the crm status command
  * Medium (LF 2184): Tools: crm: extend ptest command
  * Medium (LF 2185): Tools: crm: add resource promote/demote commands
  * Medium (LF 2198): Tools: crm: add node fence command
  * Medium: ais: Attempt to enable core file generation if it was disabled
  * Medium: ais: Include version details in plugin name
  * Medium: Build: Re-enable asciidoc documentation
  * Medium: Build: Shell templates arent documentation
  * Medium: cib: Remove delay for remote plaintext connections
  * Medium: Core: Disable syslog for any process that doesn't want its arguments logged
  * Medium: crmd: Requery the resource metadata after every start operation
  * Medium: cts: add --benchmark for scalability tests
  * Medium: cts: Prepare for corosync testing
  * Medium: Extra: Include SNMP MIB file for crm_mon (from Michael Schwartzkopff)
  * Medium: PE: Bug lf#2178 - Indicate unmanaged clones
  * Medium: PE: Bug lf#2180 - Include node information for all failed ops
  * Medium: PE: Bug lf#2189 - Incorrect error message when unpacking simple ordering constraint
  * Medium: PE: Correctly log resources that would like to start but can't
  * Medium: PE: Correctly log the state of orphaned clone instances
  * Medium: PE: If no migrate_(from|to) action is defined, look for migrate instead
  * Medium: PE: Only re-instate target-role if it is less than the calculated one
  * Medium: PE: Provide details for the maintenance-mode option
  * Medium: PE: Stop ptest from logging to syslog
  * Medium: Tools: attrd_updater - Suppress all logging with --quiet
  * Medium: Tools: crm: add extra flag to CibObject for invalid objects
  * Medium: Tools: crm: do return cached resources dom node
  * Medium: Tools: crm: expand template documentation
  * Medium: Tools: crm: first child of a removed parent inherits constraints
  * Medium: Tools: crm_attribute - Suppress all logging with --quiet
  * Medium: Tools: crm_shadow - log diffs to stdout instead of stderr
  * Medium: Tools: Use -q as the short form for --quiet (for consistency)

