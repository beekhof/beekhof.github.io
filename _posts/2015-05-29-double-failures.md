---

date: 2015-05-29 13:34
title: Double Failure - Get out of Jail Free? Not so Fast
published: true
excerpt: It’s tempting to explain away some recovery scenarios, but it's not always appropriate.
category:
tags: 
- cluster
- openstack
- concepts
---

Occasionally you might hear someone explain away a failure/recovery scenario with "that's a double failure, we can't/don't protect against those".

There are certainly situations where this is true.
A node failure combined with a fencing device failure will and should prevent a cluster from recovering services on that node.

*However!*

It doesn't mean we can ignore the failure.
Nor does it it make it acceptable to forget that services on the failed node still need to be recovered one day.

Playing the "double failure" card also requires the failures to be in different layers.
Notice that the example above was for a node failure and fencing failure.

The failure of a second node while recovering from the first doesn't count (unless it was your last one).


Just something to keep in mind in case anyone was thinking about designing something to support highly available openstack instances...
