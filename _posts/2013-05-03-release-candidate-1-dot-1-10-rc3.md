---

title: "Release candidate: 1.1.10-rc3"
date: 2013-05-23 10:32
comments: true
tags:
- announce
---

Announcing the third [release candidate](/blog/2013/release-candidate-1-dot-1-10-rc1/) for Pacemaker 1.1.10

This RC is a result of work in several problem areas reported by users, some of which date back to 1.1.8: 

* manual fencing confirmations
* potential problems reported by Coverity
* the way anonymous clones are displayed
* handling of resource output that includes non-printing characters
* handling of `on-fail=block`

Please keep the bug reports coming in.  There is a good chances that
this will be the final release candidate and 1.1.10 will be tagged on
May 30th.

Help is specifically requested for testing plugin-based clusters, ACLs
and admin actions (such as moving and stopping resources, calls to
stonith_admin) which are hard to test in an automated manner.

To build `rpm` packages for testing:

1. Clone the current sources:

       # git clone --depth 0 git://github.com/ClusterLabs/pacemaker.git
       # cd pacemaker

1. Install dependancies

       [Fedora] # sudo yum install -y yum-utils
       [ALL]	# make rpm-dep

1. Build Pacemaker

       # make rc

1. Copy and deploy as needed

## Details - 1.1.10-rc3

<table>
  <tr><td><strong>Changesets&nbsp;</strong></td> <td>116</td></tr>
  <tr><td><strong>Diff</strong></td> <td>59 files changed, 707 insertions(+), 408 deletions(-)</td></tr>
</table>
<br/>

## Highlights
### Features added in Pacemaker-1.1.10-rc3

  * PE: Display a list of nodes on which stopped anonymous clones are not active instead of meaningless clone IDs
  * PE: Suppress meaningless IDs when displaying anonymous clone status

### Changes since Pacemaker-1.1.10-rc2

  + Bug cl#5133 - pengine: Correctly observe on-fail=block for failed demote operation
  + Bug cl#5151 - Ensure node names are consistently compared without case
  + Check for and replace non-printing characters with their octal equivalent while exporting xml text
  + cib: CID#1023858 - Explicit null dereferenced
  + cib: CID#1023862 - Improper use of negative value
  + cib: CID#739562 - Improper use of negative value
  + cman: Our daemons have no need to connect to pacemakerd in a cman based cluster
  + crmd: Do not record pending delete operations in the CIB
  + crmd: Ensure pending and lost actions have values for last-run and last-rc-change
  + crmd: Insert async failures so that they appear in the correct order
  + crmd: Store last-run and last-rc-change for fail operations
  + Detect child processes that terminate before our SIGCHLD handler is installed
  + fencing: CID#739461 - Double close
  + fencing: Correctly broadcast manual fencing ACKs
  + fencing: Correctly mark manual confirmations as complete
  + fencing: Do not send duplicate replies for manual confirmation operations
  + fencing: Restore the ability to manually confirm that fencing completed
  + lrmd: CID#1023851 - Truncated stdio return value
  + lrmd: Don't complain when heartbeat invokes us with -r
  + pengine: Correctly handle resources that recover before we operate on them
  + pengine: Re-initiate _active_ recurring monitors that previously failed but have timed out
  + xml: Restore the ability to embed comments in the cib
