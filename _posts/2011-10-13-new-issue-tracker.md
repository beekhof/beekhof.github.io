---
layout: post
title: New Issue Tracker
tags: 
---
Since it's clearly not acceptable for our issue tracker to be offline for
months at a time, it is time to replace the Bugzilla instance hosted by the
Linux Foundation with something else.

One candidate that came close was the github issue tracker, but alas it
doesn't support attachments. The end result is that we now have an instance of
[Bugzilla v4](http://www.bugzilla.org/) at:

[http://bugs.clusterlabs.org](http://bugs.clusterlabs.org)

Bug numbers start at 5000.

This avoids clashing with older ones and _may_ enable us to import the old
ones if it ever comes back up again. I would advise people to assume this wont
happen and to re-create any unresolved issues.

