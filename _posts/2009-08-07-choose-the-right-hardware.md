---

title: Choose the Right Hardware
tags:
- tips
---
Recently I was asked to help diagnose a cluster that was behaving incredibly
badly.

In this case, it was a 2-node cluster based on OpenAIS and they were
simulating failures. They'd initiate a failure and sure enough, the other node
would recognize and respond appropriately.

So far so good.

The problems started when the failed node rebooted.

When the failed node returned, it came up alone and proceeded to fence the
surviving one. Which came up alone and shot its peer. Which came up alone and
shot its peer.

You get the picture.

Somehow what should have been a simple recovery had escalated into a STONITH
death-match.

To make a long story short, the root of the problem was their managed switch.
The switch, a SMC 8824M, was not correctly handling
[multicast](http://en.wikipedia.org/wiki/Multicast).

When a node rebooted, and only when the node rebooted, it took approximately
two minutes for the multicast group to properly re-form. This was long enough
for the returning node to declare itself a master, fence its peer(s) and begin
starting services. At which point it too would get shot.

The solution was to adjust the _dc-deadtime_ option. _dc-deadtime_ tells the
cluster how long to wait before deciding it is all by itself. Setting this to
an interval long enough for the switch to reliably re-form the multicast group
prevented the death-match from occurring.

To tell the cluster to wait two minutes, run:

    
    crm configure property dc-deadtime=2min
    

In this instance however, the customer was lucky. Simply rebooting the switch
caused the problem to go away.

Which I'm sure made their finance department happy too.

### Lessons learned

  * Switches can have bugs too
  * Choose your hardware carefully, make sure its suitable for the task it is intended for.

