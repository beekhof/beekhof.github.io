---
layout: post
title: Pacemaker 1.1.7 Now Available
tags: 
---
After much hard work, the latest installment of the
[Pacemaker](http://www.clusterlabs.org/wiki/Pacemaker) 1.1 release series is
now ready for general consumption.

## Details
<table><tr><td><strong>Changesets&nbsp;</strong></td> <td> 513 </td>
</tr><tr><td><strong>Diff</strong></td> <td>1171 files changed, 90472 insertions, 19368
deletions</td> </tr></table><br/>

As well as the usual round of bug fixes, see the [full changelog](https://raw.github.com/ClusterLabs/pacemaker/master/ChangeLog),
this new release brings:

  * Support for Corosync 2.0
  * Logging optimisations (less of it and less work performed for logs that wont be printed)
  * The ability to specify that A starts after ( B or C or D )
  * Support for advanced fencing topologies: eg. kdump || (network && disk) || power
  * Resource templates and tickets have been promoted to the stable schema
  * Support for gracefully giving up resources depending on a ticket

As per our [release calendar](http://www.clusterlabs.org/wiki/ReleaseCalendar), the next 1.1
release is planned for mid-July.

Packages for all current editions of Fedora have been built and will be
appearing shortly in the update channels. Other distributions will follow when
their schedules allow it.

The [source tarball (tar.gz)](https://nodeload.github.com/ClusterLabs/pacemaker/tarball/Pacemaker-1.1.7) is also available directly from GitHub.

General installation instructions are available at from the [ClusterLabs wiki](http://clusterlabs.org/wiki/Install).

