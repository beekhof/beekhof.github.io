---

title: Pacemaker 1.0.8 Released
tags:
- announce
---
The latest installment of the
[Pacemaker](http://www.clusterlabs.org/wiki/Pacemaker) 1.0 stable series is
now ready for general consumption.

In this release, apart from various bug-fixes, Dejan has split the shell up
into modules. It is anticipated that this will make it easier to maintain
moving forward.

We are now following the published [release schedule](http://www.clusterlabs.org/wiki/ReleaseCalendar) on the clusterlabs
wiki. The next release is planned for mid-June and our main focus is now on
features for 1.1/1.2 (see the [previous post](/blog/2010/new-pacemaker-release-series/)).

Pre-built packages for Pacemaker and it's immediate dependancies are currently
building and will be available for openSUSE, SLES, Fedora, RHEL, CentOS from
the [ClusterLabs Build Area](http://www.clusterlabs.org/rpm/) shortly.

Debian users should check for updates [Martin's repo](http://clusterlabs.org/wiki/Install#Debian) over the coming days and Ubuntu fans can visit [LaunchPad](https://edge.launchpad.net/~ubuntu-ha-maintainers/+archive/ppa) for 8.04 and 9.10 packages.

The [source tarball](http://hg.clusterlabs.org/pacemaker/stable-1.0/archive/Pacemaker-1.0.8.tar.bz2) is also available directly from Mercurial.

General installation instructions are available at from the [ClusterLabs wiki](http://clusterlabs.org/wiki/Install).

### Release Statistics

## Details
<table><tr><td><strong>Changesets&nbsp;</strong></td> <td> 181 </td>
</tr><tr><td><strong>Diff</strong></td> <td> 329 files changed, 22172 insertions(+),
12297 deletions(-)

The size of the diff is significantly impacted by the rearrangement of the
shell </td> </tr></table><br/>

### Changes of note since Pacemaker-1.0.7

  * High: Agents: ping - Prevent shell expansion of '*' when there are files in /var/lib/heartbeat/cores/root (Patch from Sébastien PRUDHOMME)
  * High: ais: Bug lf#2340 - Force rogue child processes to terminate after waiting 2.5 minutes
  * High: ais: Bug lf#2359 - Default expected votes to 2 inside Corosync/OpenAIS plugin
  * High: ais: Bug lf#2359 - expected-quorum-votes not correctly updated after membership change
  * High: ais: Bug rhbz#525552 - Move non-threadsafe calls to setenv() to after the fork()
  * High: crmd: Bug bnc#578644 - Improve handling of cancelled operations caused by resource cleanup
  * High: PE: Bug lf#2317 - Avoid needless restart of primitive depending on a clone
  * High: PE: Bug lf#2358 - Fix master-master anti-colocation
  * High: PE: Bug lf#2361 - Ensure clones observe mandatory ordering constraints if the LHS is unrunnable
  * High: PE: Correctly implement optional colocation between primitives and clone resources
  * High: Shell: add support for xml in cli
  * High: Shell: check timeouts also against the default-action-timeout property
  * High: Shell: edit multiple meta_attributes sets in resource management (lf#2315)
  * High: Shell: improve configure commit (lf#2336)
  * High: Shell: new cibstatus import command (bnc#585471)
  * High: Shell: restore error reporting in options
  * High: Shell: update previous node lookup procedure to include the id where necessary
  * High: Shell: move scores from resource sets to the constraint element (lf#2331)
  * High: Shell: recovery from bad/outdated help index file
  * Medium: ais: getpwnam() is also not thread safe, move after the call to fork()
  * Medium: ais: Set permissions to allow 'to_file' logging to function correctly
  * Medium: Core: Give signal handlers higher priority - patch based on Lars Ellenbergs work
  * Medium: crmd: Bug bnc#578644 - Do not send operation updates for deleted resources
  * Medium: PE: Bug bnc#586710 - Make sure migration ops use the correct meta options (eg. timeouts)
  * Medium: PE: Deprecate the lifetime tag in constraints
  * Medium: Tools: attrd - Only ignore the update if the attributes value is completely stable (ie. supplied, current, and stored all match)
  * Medium: Tools: Bug lf#2302 - Use the same resource printing logic for html and non-html output
  * Medium: Tools: Bug LF#2312 - crm_mon - Prevent zombie child processes when using custom traps (Patch from Bernd Schubert)
  * Medium: Tools: Bug lf#2330 - Add a blank line after the subject to indicate the beginning of the mail body
  * Medium: Tools: Bug lf#2330 - Move the blank line before the body text instead
  * Medium: Tools: Bug lf#2330 - Use \r in addition to \n for line endings
  * Medium: Tools: crm_mon - Add support for older versions of SNMP - Patch derived from the work of sato yuki
  * Medium: Tools: crm_mon - Display the true fail-count, not the effective value
  * Medium: Tools: crm_mon - Use node uname in snmp/smtp/etc events
  * Medium: Tools: hb2openais: add support for corosync (and more)
  * Medium: Shell: add option to control sorting of cib elements (lf#2290)
  * Medium: Shell: do not cache node and resource ids (lf#2368)
  * Medium: Shell: fix commit for new clones of new groups (bnc#585471)
  * Medium: Shell: help: unsort help items
  * Medium: Shell: implement lifetime for rsc migrate and node standby (lf#2353)
  * Medium: Shell: load update should update existing elements
  * Medium: Shell: node attributes update in configure (bnc#582767)
  * Medium: Shell: parse lists not tupples
  * Medium: Shell: Repair "cib cibstatus op" functionality (bnc#585641)
  * Medium: Shell: repair node show (thanks to T. Schraitle) (bnc#587883)
  * Medium: Shell: repare clone/ms cleanup (nbc#583288)
  * Medium: Shell: catch IOErrors when opening files
  * Medium: Shell: check for duplicate children when creating groups (lf#2326)
  * Medium: Shell: do not allow score-attribute in orders
  * Medium: Shell: do not fiddle with cib when there is no cib (bnc#575701)
  * Medium: Shell: do not produce empty resource sets when adding roles/actions
  * Medium: Shell: do not verify empty configurations (lf#2316)
  * Medium: Shell: fix CIB upgrade command (bnc#578637)
  * Medium: Shell: fix exit code for template apply
  * Medium: Shell: fix reference replacement in resource sets
  * Medium: Shell: install crm_cli.txt also in the datadir
  * Medium: Shell: use the CRM_HELP_FILE variable if set

