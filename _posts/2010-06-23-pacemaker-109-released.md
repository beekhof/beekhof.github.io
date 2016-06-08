---

title: Pacemaker 1.0.9 Released
tags:
- announce
---
The latest installment of the
[Pacemaker](http://www.clusterlabs.org/wiki/Pacemaker) 1.0 stable series is
now ready for general consumption.

Coinciding with 1.0.9 is a new version of Corosync (1.2.5). Included in both
are some important fixes that should resolve most of the startup issues people
have been seeing. Also included in this release are the fixes for issues
reported by [Valgrind](http://valgrind.org/) and
[Coverity](http://www.coverity.com/products/static-analysis.html).

As per our [release calendar](http://www.clusterlabs.org/wiki/ReleaseCalendar),
the next 1.0 release is planned for mid-September and 1.1.3 will be
available in late July.

> I'd like to particularly thank Keisuke MORI for his help with this release.
> Keisuke-san has taken on the role of Patch Manager for 1.0, so it is because
> of his hard work that we have backports of all the bugfixes from 1.1 :-)
>
> This change has enabled me to focus on 1.1 and, I hope, be slightly more
> responsive to bug reports and questions on the mailing list(s).

Pre-built packages for Pacemaker and it's immediate dependancies are available
immediately for openSUSE, SLES, Fedora, RHEL, CentOS from the [ClusterLabs Build Area](http://www.clusterlabs.org/rpm/).

Regular updaters may also have noticed the expanded version scheme used by
packages on clusterlabs.org. The build scripts now automatically bump the
version numbers when rebuilding the stack. This usually occurs when new
versions of corosync, cluster-glue or heartbeat come out.

Versions are now of the form: x.y.x-a.b

  * _x.y.z_ is the upstream version (this is the only time the tarball is changed)
  * _a_ indicates the number of spec file changes (ie. changes to dependancies)
  * _b_ indicates how many times the package has been rebuilt with unchanged tarballs and spec files

So the following version: pacemaker-1.0.9-1.4 would mean the fourth rebuild of
the initial spec file for the upstream version _1.0.9_ of Pacemaker.

Debian users should check for updates [Martin's repo](http://clusterlabs.org/wiki/Install#Debian) over the coming days and
Ubuntu fans can visit [LaunchPad](https://edge.launchpad.net/~ubuntu-ha-maintainers/+archive/ppa) for 8.04 and 9.10 packages.

The [source tarball](http://hg.clusterlabs.org/pacemaker/stable-1.0/archive/Pacemaker-1.0.9.tar.bz2) is also available directly from Mercurial.

General installation instructions are available at from the [ClusterLabs wiki](http://clusterlabs.org/wiki/Install).

### Release Statistics

<table><tr><td><strong>Changesets&nbsp;</strong></td> <td> 152 </td>
</tr><tr><td><strong>Diff</strong></td> <td>266 files changed, 14324 insertions(+), 3842
deletions(-)</td> </tr></table><br/>

### Changes of note since Pacemaker-1.0.8

  * High: ais: Ensure the list of active processes sent to clients is always up-to-date
  * High: ais: Fix previous commit, actually return a result in get_process_list()
  * High: ais: Fix two more uses of getpwnam() in non-thread-safe locations
  * High: ais: Look for the correct conf variable for turning on file logging
  * High: ais: Need to find a better and thread-safe way to set core_uses_pid. Disable for now.
  * High: ais: Use the threadsafe version of getpwnam
  * High: cib: Also free query result for xpath operations that return more than one hit
  * High: cib: Fix the application of unversioned diffs
  * High: cib: Remove old developmental error logging
  * High: Core: Bug lf#2414 - Prevent use-after-free reported by valgrind when doing xpath based deletions
  * High: Core: Fix memory leak in replace_xml_child() reported by valgrind
  * High: Core: fix memory leaks exposed by valgrind
  * High: crmd: Bug 2401 - Improved detection of partially active peers
  * High: crmd: Bug lf#2379 - Ensure the cluster terminates when the PE is not available
  * High: crmd: Bug lf#2414 - Prevent use-after-free of the PE connection after it dies
  * High: crmd: Bug lf#2439 - cancel_op() can also return HA_RSCBUSY
  * High: crmd: Bug lf#2439 - Handle asynchronous notification of resource deletion events
  * High: crmd: Do not allow the target_rc to be misused by resource agents
  * High: crmd: Do not ignore action timeouts based on FSA state
  * High: crmd: Ensure we dont get stuck in S_PENDING if we loose an election to someone that never talks to us again
  * High: crmd: Fix memory leaks exposed by valgrind
  * High: crmd: Remove race condition that could lead to multiple instances of a clone being active on a machine
  * High: crmd: Send erase_status_tag() calls to the local CIB when the DC is fenced, since there is no DC to accept them
  * High: PE: Bug lf#1959 - Fail unmanaged resources should not prevent other services from shutting down
  * High: PE: Bug lf#2383 - Combine failcounts for all instances of an anonymous clone on a host
  * High: PE: Bug lf#2384 - Fix intra-set colocation and ordering
  * High: PE: Bug lf#2403 - Enforce mandatory promotion (colocation) constraints
  * High: PE: Bug lf#2412 - Correctly locate clone instances by their prefix
  * High: PE: Bug lf#2422 - Ordering dependencies on partially active groups not observed properly
  * High: PE: Bug lf#2424 - Use notify oepration definition if it exists in the configuration
  * High: PE: Bug lf#2433 - No services should be stopped until probes finish
  * High: PE: Do not be so quick to pull the trigger on nodes that are coming up
  * High: PE: Fix colocation for interleaved clones
  * High: PE: Fix colocation with partially active groups
  * High: PE: Fix memory leaks reported by valgrind
  * High: PE: Make the current data set a global variable so it does not need to be passed around everywhere
  * High: PE: Prevent endless loop when looking for operation definitions in the configuration
  * High: PE: Rewrite native_merge_weights() to avoid Fix use-after-free
  * High: Shell: always reload status if working with the cluster (bnc#590035)
  * High: Tools: crm_mon - fix memory leaks exposed by valgrind
  * Medium: ais: Correctly set logfile permissions in all cases
  * Medium: ais: create the final directory too for resource agents (bnc#603190)
  * Medium: ais: Make sure debug messages make it into the logfiles too
  * Medium: Build: Do not enable the -ansi compiler option by default, prevents use of strtoll()
  * Medium: cib: Bug lf#2352 - Changes to group order are not detected or broadcast to peers
  * Medium: cib: Correctly free the cib contents at signoff when in file-based mode
  * Medium: cib: xpath - Allow all hits to be deleted, allow the no_children option to return multiple hits
  * Medium: PE: Bug lf#2391 - Ensure important options (notify, unique, etc) are always exposed during resource operations
  * Medium: PE: Bug lf#2410 - Do not complain about missing agents during probes of a-symetric clusters
  * Medium: PE: Bug lf#2426 - stop-all-resources should not apply to stonith resources
  * Medium: PE: Bug lf#2435 - Support colocation sets with negative scores
  * Medium: PE: Check for use-of-NULL in dump_node_scores()
  * Medium: PE: Do not overwrite existing meta attributes (like timeout) for notify operations
  * Medium: PE: Ensure deallocated resources are stopped
  * Medium: PE: If there are no compatible peers when interleaving clones, ensure the instance is stopped
  * Medium: PE: Ignore colocation weights from clone instances
  * Medium: RA: SystemHealth: exit properly when the required software is not installed (bnc#587940)
  * Medium: Shell: do not error on missing resource agent with asymmetrical clusters (lf#2410)
  * Medium: Shell: do not verify empty configurations (bnc#602711)
  * Medium: shell: find hb_delnode in correct directory
  * Medium: Shell: observe op_defaults when verifying primitives (bnc#590033)
  * Medium: Shell: on no id match the first of property-like elements (lf#2420)
  * Medium: Shell: skip resource checks for property-like elements (lf#2420)
  * Medium: Shell: verify meta attributes and properties (bnc#589867)
  * Medium: Shell: verify only changed elements on commit (bnc#590033)
  * Medium: Tools: crm_mon: refresh screen on terminal resize (bnc#589811)

