---

title: Pacemaker removed from OBS
tags:
- announce
---
Today I removed Pacemaker from server:ha-clustering on the openSUSE build
service.

I lost patience with the service some time ago and the project has been
providing pre-built packages from [clusterlabs](http://www.clusterlabs.org/rpm) ever since (see our [install page](http://www.clusterlabs.org/wiki/Install) for more details).

It seems no-one else has had the time or patience to keep the build service
updated since my departure so, after noticing their age and the fact that they
no longer even build on the majority of targets, I made the decision to remove
them.

Hopefully this will help avoid confusing those wanting the latest Pacemaker
software.

