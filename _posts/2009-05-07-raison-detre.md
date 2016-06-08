---

title: raison d'etre
tags:
- general
---
This tumbl/blog/thingy exists because I've finally accepted that _"If we build
it, they will come"_ is a fallacy.  The internet is a big place and if you
don't speak up, you'll get lost in the noise of those that do.

So, I'm going to try and use this place to raise awareness of a project that's
very important to me - [Pacemaker](http://clusterlabs.org) - an incredibly
advanced open source, high availability cluster resource manager.

For those not already certified cluster ninjas, a resource manager is the part
of a cluster stack that decides who holds cluster services and what to do when
a failure is detected.

Pacemaker's key features, which I will explore in greater depth over the
coming days/weeks, are:

  * Recovery from node failures (obviously)
  * Built-in detection of resource failures (no need for [mon](http://mon.wiki.kernel.org)
  * Support for [OpenAIS](http://www.openais.org), an industry standard cluster stack
  * Support for [Heartbeat](http://linux-ha.org), a popular alternative to OpenAIS
  * Powerful dependancy model for accurately mapping your environment
  * Supports as many nodes as the cluster messaging layer will allow
  * Proven technology - ships as part of [SLE10](http://novell.com/linux) and [SLE 11 High Availability Extension](http://www.novell.com/products/highavailability/)

If you're interested in open source clustering, check us out at
[http://clusterlabs.org](http://clusterlabs.org) or [irc://irc.freenode.net#linux-cluster](irc://irc.freenode.net#linux-cluster)
