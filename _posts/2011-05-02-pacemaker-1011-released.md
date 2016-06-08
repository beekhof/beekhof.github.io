---

title: Pacemaker 1.0.11 Released
tags:
- announce
---
The latest installment of the
[Pacemaker](http://www.clusterlabs.org/wiki/Pacemaker) 1.0 release series is
now ready for general consumption.

## Details
<table><tr><td><strong>Changesets&nbsp;</strong></td> <td> 85 </td>
</tr><tr><td><strong>Diff</strong></td> <td>500 files changed, 69642 insertions(+),
58270 deletions(-)</td> </tr></table><br/>

Thanks once again to the efforts of Keisuke MORI and NTT, the latest bug fixes
have been back-ported from 1.1

### Highlights
Important changes since Pacemaker-1.0.10 include:

  * cib: Repair the processing of updates sent from peer nodes
  * crmd: All pending operations should be recorded, even recurring ones with high start delays
  * crmd: Bug lf#2509 - Watch for config option changes from the CIB even if we're not the DC
  * crmd: Bug lf#2528 - Introduce a slight delay when creating a transition to allow attrd time to perform its updates
  * crmd: Bug lf#2545 - Ensure notify variables are accurate for stop operations
  * crmd: Bug lf#2559 - Fail actions that were scheduled for a failed/fenced node
  * crmd: Cancel recurring operations while we're still connected to the lrmd
  * crmd: Don't abort transitions when probes are completed on a node
  * crmd: Ensure the CIB is always writable on the DC by removing a timing hole
  * crmd: Update failcount for failed promote and demote operations
  * PE: Bug lf#2495 - Prevent segfault by validating the contents of ordering sets
  * PE: Bug lf#2508 - Correctly reconstruct the status of anonymous cloned groups
  * PE: Bug lf#2544 - Prevent unstable clone placement by factoring in the current node's score before all others
  * PE: Bug lf#2554 - target-role alone is not sufficient to promote resources
  * PE: Ensure fencing of the DC preceeds the STONITH_DONE operation
  * PE: Ensure that fencing has completed for stop actions on stonith-dependent resources (lf#2551)
  * PE: Prevent clones from being stopped because resources colocated with them cannot be active
  * PE: Prevet use-after-free resulting from unintended recursion when chosing a node to promote master/slave resources
  * Shell: don't create empty optional sections (bnc#665131)
  * Tools: Bug lf#2528 - Make progress when attrd_updater is called repeatedly within the dampen interval but with the same value
  * Tools: Prevent crm_resource commands from being lost due to the use of cib_scope_local

You also can see the [full changelog](http://hg.clusterlabs.org/pacemaker/1.0/raw-file/tip/ChangeLog),

As per our [release calendar](http://www.clusterlabs.org/wiki/ReleaseCalendar),
the next 1.0.x release is planned for mid-September.

The [source tarball](http://hg.clusterlabs.org/pacemaker/1.0/archive/Pacemaker-1.0.11.tar.bz2) is also available directly from Mercurial.

Pre-built packages for Pacemaker and it's immediate dependancies are available
immediately for openSUSE 11.2, 11.3, Fedora-13 and EPEL-5 from the
[ClusterLabs Build Area](http://www.clusterlabs.org/rpm/).

Users of more recent distributions are encouraged to use the latest 1.1.x -
either from the [1.1 Build Area](http://www.clusterlabs.org/rpm-next/) or the
distribution directly.

General installation instructions are available at from the [ClusterLabs wiki](http://clusterlabs.org/wiki/Install).
