---
layout: post
title: Pacemaker Release Roundup
tags: 
---
It may have seemed quiet since July, but things were actually so busy that I
couldn't find the time to publicize our new releases.

First up, the long awaited 1.0.10 is finally here. Thanks once again to the
hard work of Keisuke MORI from NTT, 1.0.10 contains all the bug fixes from the
recent 1.1.3 and 1.1.4 releases. You can preview the list of updates with the
new online [change log](http://hg.clusterlabs.org/pacemaker/stable-1.0/raw-file/tip/ChangeLog).

In addition to [general bugfixes](http://hg.clusterlabs.org/pacemaker/1.1/raw-file/tip/ChangeLog), the big news in 1.1.3 was the addition of a [master
control process](/blog/2010/introducing-the-pacemaker-master-control-process-for-corosync-based-clusters/) and support for cman. Cman support allows us to run on top of a
traditional RHCS cluster stack - replacing just the rgmanager component (more
details on this in a subsequent post).

1.1.3 also introduced a new logging system inspired by the kernel and a PoC
from Lars Ellenberg. It enables us to selectively enable logs for specific
files, functions and even individual lines. Eventually this should result in
less being logged by default.

The successor to 1.1.3 was all about
[performance](/blog/2010/large-cluster-performance/). In 1.1.4 we managed to speed up the CIB and Policy
Engine by about 80% each. So if you have 100's of resources, you _really_ want
to be using this version (the changes were far too invasive to consider
including in a 1.0 release).

Packages for all three releases are available from the
[rpm](http://www.clusterlabs.org/rpm) and [rpm-next](http://www.clusterlabs.org/rpm-next) repositories on clusterlabs.org

In other news, I have also recently updated the [release calendar](http://www.clusterlabs.org/wiki/ReleaseCalendar) for 2011.

