---

title: "Changes to the Remote Wire Protocol in 1.1.11"
date: 2013-10-31 12:04
comments: true
tags: 
---

Unfortunately the current wire protocol used by `pacemaker_remoted`
for exchanging messages was found to be suboptimal and we have taken
the decision to change it now before it becomes widely adopted.

We attempted to do this in a backwards compatibile manner, however the
two methods we tried were either overly complicated and fragile, or
not possible due to the way the released `crm_remote_parse_buffer()`
function operated.

The changes include a versioned binary header that contains the size
of the header, payload and total message, control flags and a
big/little-endian detector.

These changes will appear in the upstream repo shortly and ship in
1.1.11.  Anyone for this will be a problem is encouraged to get in
contact to discuss possible options.

For RHEL users, any version on which `pacemaker_remoted` is supported
will have the new versioned protocol.  That means 7.0 and potentially
a future 6.x release.

