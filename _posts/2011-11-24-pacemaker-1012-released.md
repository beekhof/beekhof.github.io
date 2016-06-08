---

title: Pacemaker 1.0.12 Released
tags:
- announce
---
Thanks once again to the efforts of Keisuke MORI from NTT, the latest bug
fixes have been back-ported from 1.1 and another instalment of the
[Pacemaker](http://www.clusterlabs.org/wiki/Pacemaker) 1.0 release series is
now ready for general consumption.

## Details
<table><tr><td><strong>Changesets&nbsp;</strong></td> <td> 96 </td>
</tr><tr><td><strong>Diff</strong></td> <td>121 files changed, 8617 insertions(+), 988
deletions(-)</td> </tr></table><br/>

### Highlights
Important changes since Pacemaker-1.0.11 include:

  * cib: Call gnutls_bye() and shutdown() when disconnecting from remote TLS connections
  * cib: Remove disconnected remote connections from mainloop
  * crmd: Cancel timers for actions that were pending on dead nodes
  * crmd: Do not wait for actions that were pending on dead nodes
  * crmd: Ensure we do not attempt to perform action on failed nodes
  * PE: Correctly recognise which recurring operations are currently active
  * PE: Demote from Master does not clear previous errors
  * PE: Ensure restarts due to definition changes cause the start action to be re-issued not probes
  * PE: Ensure role is preserved for unmanaged resources
  * PE: Ensure unmanaged resources have the correct role set so the correct monitor operation is chosen
  * PE: Move master based on failure of colocated group
  * pengine: Correctly determine the state of multi-state resources with a partial operation history
  * PE: Only allocate master/slave resources once
  * Shell: implement -w,--wait option to wait for the transition to finish
  * Shell: repair template list command

You also can see the [full changelog](https://github.com/ClusterLabs/pacemaker
-1.0/blob/master/ChangeLog),

I have updated the [release
calendar](http://www.clusterlabs.org/wiki/ReleaseCalendar) and the next 1.0.x
release is planned for mid-May 2012.

The [source tarball](https://github.com/ClusterLabs/pacemaker-1.0/tarball/Pace
maker-1.0.12) is also available directly from GitHub.

Pre-built packages for Pacemaker are available immediately for current
[openSUSE](http://www.opensuse.org/) (12.1, 11.4, 11.3) and
[Fedora](http://fedoraproject.org/) (16, 15, 14) releases as well as
[EPEL-5](http://fedoraproject.org/wiki/EPEL) from the [ClusterLabs Build Area](http://www.clusterlabs.org/rpm/).

Users of more most distributions are **encouraged to use the latest 1.1.x
release** - either from the [1.1 Build Area](http://www.clusterlabs.org/rpm-next/) or from the distribution directly.

General installation instructions are available at from the [ClusterLabs wiki](http://clusterlabs.org/wiki/Install).
