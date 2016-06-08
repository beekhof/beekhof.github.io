---

title: Pacemaker 1.1.5 Released
tags:
- announce
---

The latest installment of the
[Pacemaker](http://www.clusterlabs.org/wiki/Pacemaker) 1.1 release series is
now ready for general consumption.

## Details
<table><tr><td><strong>Changesets&nbsp;</strong></td> <td> 184 </td>
</tr><tr><td><strong>Diff</strong></td> <td>605 files changed, 46103 insertions(+),
26417 deletions(-)</td> </tr></table><br/>

As well as the usual round of bug fixes, see the [full changelog](http://hg.clusterlabs.org/pacemaker/1.1/raw-file/tip/ChangeLog),
S.U.S.E. has implemented support for ACLs. This means that you can now
delegate permission to control parts of the cluster (as defined by you) to
non-root users.

ACLs are still disabled by default, but you can read [their documentation](http://www.clusterlabs.org/doc/acls.html), provide feedback and decide if its something you want to use.

As per our [release calendar](http://www.clusterlabs.org/wiki/ReleaseCalendar),
the next 1.1 release is planned for mid-April and 1.0.11 should be
available in March depending on how quickly we can get the bugfixes
from 1.1 backported.

Pre-built packages for Pacemaker and it's immediate dependancies are available
immediately for openSUSE 11.3, Fedora-14 and EPEL-5 from the [ClusterLabs Build Area](http://www.clusterlabs.org/rpm-next/).

The [source tarball](http://hg.clusterlabs.org/pacemaker/1.1/archive/Pacemaker-1.1.5.tar.bz2) is also available directly from Mercurial.

General installation instructions are available at from the [ClusterLabs wiki](http://clusterlabs.org/wiki/Install).
