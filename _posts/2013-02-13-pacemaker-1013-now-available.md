---
layout: post
title: Pacemaker 1.0.13 now available
comments: true
tags:
- announce
---
Thanks once again to the efforts of the fine folks from NTT, the latest bug
fixes have been back-ported from 1.1 and another instalment of the
[Pacemaker](http://www.clusterlabs.org) 1.0 release series is now ready for
general consumption.

## Details
<table><tr><td><strong>Changesets&nbsp;</strong></td> <td> 129 </td>
</tr><tr><td><strong>Diff</strong></td> <td>173 files changed, 12206 insertions(+), 767
deletions(-)</td> </tr></table><br/>

### Highlights
Important changes since Pacemaker-1.0.12 include:

  * cib: Don't halt disk writes if the previous digest is missing
  * cib: Fix coverity RESOURCE_LEAK defect
  * Core: Avoid assertion error when underflowing days of the month in iso8601 date code
  * Core: Correctly determine when an XML file should be decompressed
  * Core: Ensure signals are handled eventually in the absense of timer sources or IPC messages
  * Core: Strip text nodes from on disk xml files
  * crmd: cl#5051 - Fixes file leak in pe ipc connection initialization.
  * crmd: cl#5057 - Restart sub-systems correctly (bnc#755671)
  * crmd: Fast-track shutdown if we couldn't request it via attrd
  * crmd: Leave it up to the PE to decide which ops can/cannot be reload
  * crmd: Prevent use-of-NULL when free'ing empty hashtables
  * crmd: Supply format arguments in the correct order
  * Fix memory leak in cib when writing the cib contents.
  * legacy: Set to the minimum scheduling priority when using SCHED_RR policy (bnc#779259)
  * pengine: Bug #5007, Fixes use of colocation constraints with multi-state resources
  * pengine: Bug cl#5038 - Prevent restart of anonymous clones when clone-max decreases
  * pengine: Bug cl#5101 - Ensure stop order is preserved for partially active groups
  * pengine: cl#5069 - Honor 'on-fail=ignore' even when operation is disabled.
  * pengine: cl#5072 - Fixes monitor op stopping after rsc promotion.
  * pengine: Ensure post-migration stop actions occur before node shutdown
  * pengine: Fix coverity REVERSE_INULL defects
  * pengine: Fix use-after-free errors detected by coverity
  * pengine: Prevent segfault when ensuring unmanaged resources don't prevent shutdown
  * pengine: Reload of a resource no longer causes a restart of dependant resources
  * RA: controld - use the correct dlm_controld when membership comes from corosync directly
  * tools: crm_resource - Fix coverity FORWARD_NULL defect
  * Tools: crm_shadow - Bug cl#5062 - Correctly set argv[0] when forking a shell process

You also can see the [full changelog](https://github.com/ClusterLabs/pacemaker-1.0/blob/master/ChangeLog),

The next 1.0.x release will occur if and when needed (but probably not
before mid-2013).

The [source tarball](https://github.com/ClusterLabs/pacemaker-1.0/tarball/Pacemaker-1.0.13) is also available directly from GitHub.

Users of more most distributions are **encouraged to use the latest 1.1.x
release** - either from the [1.1 Build Area](http://www.clusterlabs.org/rpm-next/) or from the distribution directly.

General installation instructions are available at from the [ClusterLabs wiki](http://clusterlabs.org/wiki/Install).
