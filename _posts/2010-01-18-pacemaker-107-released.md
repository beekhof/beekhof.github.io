---

title: Pacemaker 1.0.7 Released
tags:
- announce
---
The latest installment of the
[Pacemaker](http://www.clusterlabs.org/wiki/Pacemaker) 1.0 stable series is
now ready for general consumption.

In this release, we've made a number improvements to clone handling -
particularly the way ordering constraints are processed - as well as some
really nice improvements to the shell.

The next 1.0 release is anticipated to be in mid-March. We will be switching
to a bi-monthly release schedule to begin focusing on development for the next
stable series (more details soon). If you have feature requests, now is the
time to voice them and/or provide patches :-)

Pre-built packages for Pacemaker and it's immediate dependancies are currently
building and will be available for openSUSE, SLES, Fedora, RHEL, CentOS from
the [ClusterLabs Build Area](http://www.clusterlabs.org/rpm/) shortly.

Debian users should check for updates [Martin's repo](http://clusterlabs.org/wiki/Install#Debian) over the coming days and
Ubuntu fans can visit [LaunchPad](https://edge.launchpad.net/~ubuntu-ha-maintainers/+archive/ppa) for 8.04 and 9.10 packages.

The [source tarball](http://hg.clusterlabs.org/pacemaker/stable-1.0/archive/Pacemaker-1.0.7.tar.bz2) is also available directly from Mercurial.

General installation instructions are available at from the [ClusterLabs wiki](http://clusterlabs.org/wiki/Install).

### Release Statistics

<table><tr><td><strong>Changesets&nbsp;</strong></td> <td> 193 </td>
</tr><tr><td><strong>Diff</strong></td> <td> 220 files changed, 15933 insertions(+),
8782 deletions(-)</td> </tr></table><br/>

### Changes of note since Pacemaker-1.0.6

  * High: PE: Bug 2213 - Ensure groups process location constraints so that clone-node-max works for cloned groups
  * High: PE: Bug lf#2153 - non-clones should not restart when clones stop/start on other nodes
  * High: PE: Bug lf#2209 - Clone ordering should be able to prevent startup of dependant clones
  * High: PE: Bug lf#2216 - Correctly identify the state of anonymous clones when deciding when to probe
  * High: PE: Bug lf#2225 - Operations that require fencing should wait for 'stonith_complete' not 'all_stopped'.
  * High: PE: Bug lf#2225 - Prevent clone peers from stopping while another is instance is (potentially) being fenced
  * High: PE: Correctly anti-colocate with a group
  * High: PE: Correctly unpack ordering constraints for resource sets to avoid graph loops
  * High: Tools: crm: load help from crm_cli.txt
  * High: Tools: crm: resource sets (bnc#550923)
  * High: Tools: crm: support for comments (LF 2221)
  * High: Tools: crm: support for description attribute in resources/operations (bnc#548690)
  * High: Tools: hb2openais: add EVMS2 CSM processing (and other changes) (bnc#548093)
  * High: Tools: hb2openais: do not allow empty rules, clones, or groups (LF 2215)
  * High: Tools: hb2openais: refuse to convert pure EVMS volumes
  * High: cib: Ensure the loop for login message terminates
  * High: cib: Finally fix reliability of receiving large messages over remote plaintext connections
  * High: cib: Fix remote notifications
  * High: cib: For remote connections, default to CRM_DAEMON_USER since thats the only one that the cib can validate the password for using PAM
  * High: cib: Remote plaintext - Retry sending parts of the message that did not fit the first time
  * High: crmd: Ensure batch-limit is correctly enforced
  * High: crmd: Ensure we have the latest status after a transition abort
  * High (bnc#547579,547582): Tools: crm: status section editing support
  * High: shell: Add allow-migrate as allowed meta-attribute (bnc#539968)
  * Medium: Build: Do not automatically add -L/lib, it could cause 64-bit arches to break
  * Medium: PE: Bug lf#2206 - rsc_order constraints always use score at the top level
  * Medium: PE: Only complain about target-role=master for non m/s resources
  * Medium: PE: Prevent non-multistate resources from being promoted through target-role
  * Medium: PE: Provide a default action for resource-set ordering
  * Medium: PE: Silently fix requires=fencing for stonith resources so that it can be set in op_defaults
  * Medium: Tools: Bug lf#2286 - Allow the shell to accept template parameters on the command line
  * Medium: Tools: Bug lf#2307 - Provide a way to determin the nodeid of past cluster members
  * Medium: Tools: crm: add update method to template apply (LF 2289)
  * Medium: Tools: crm: direct RA interface for ocf class resource agents (LF 2270)
  * Medium: Tools: crm: direct RA interface for stonith class resource agents (LF 2270)
  * Medium: Tools: crm: do not add score which does not exist
  * Medium: Tools: crm: do not consider warnings as errors (LF 2274)
  * Medium: Tools: crm: do not remove sets which contain id-ref attribute (LF 2304)
  * Medium: Tools: crm: drop empty attributes elements
  * Medium: Tools: crm: exclude locations when testing for pathological constraints (LF 2300)
  * Medium: Tools: crm: fix exit code on single shot commands
  * Medium: Tools: crm: fix node delete (LF 2305)
  * Medium: Tools: crm: implement -F (--force) option
  * Medium: Tools: crm: rename status to cibstatus (LF 2236)
  * Medium: Tools: crm: revisit configure commit
  * Medium: Tools: crm: stay in crm if user specified level only (LF 2286)
  * Medium: Tools: crm: verify changes on exit from the configure level
  * Medium: ais: Some clients such as gfs_controld want a cluster name, allow one to be specified in corosync.conf
  * Medium: cib: Clean up logic for receiving remote messages
  * Medium: cib: Create valid notification control messages
  * Medium: cib: Indicate where the remote connection came from
  * Medium: cib: Send password prompt to stderr so that stdout can be redirected
  * Medium: cts: Fix rsh handling when stdout is not required
  * Medium: doc: Fill in the section on removing a node from an AIS-based cluster
  * Medium: doc: Update the docs to reflect the 0.6/1.0 rolling upgrade problem
  * Medium: doc: Use Publican for docbook based documentation
  * Medium: fencing: stonithd: add metadata for stonithd instance attributes (and support in the shell)
  * Medium: fencing: stonithd: ignore case when comparing host names (LF 2292)
  * Medium: tools: Make crm_mon functional with remote connections
  * Medium: xml: Add stopped as a supported role for operations
  * Medium: xml: Bug bnc#552713 - Treat node unames as text fields not IDs
  * Medium: xml: Bug lf#2215 - Create an always-true expression for empty rules when upgrading from 0.6

