---
layout: post
title: Large Cluster Performance
tags: 
- performance
---
Over the last few days, I've spent a bunch of time improving Pacemaker's
performance in large clusters.

This involved profiling the CIB and Policy Engine, identifying and optimizing
hotspots and improving algorithm designs.

Since most of my work is done in virtual machines, it wasn't possible to use
oprofile. Strictly speaking oprofile worked, but without hardware performance
counters the results weren't very helpful. I also tried gprof, but that is
more about counting calls rather than time spent.

Eventually I switched to callgrind and when combined with a tool Tim found
called [Gprof2Dot](http://code.google.com/p/jrfonseca/wiki/Gprof2Dot) and/or
kcachegrind, finally got the data I was looking for.

To do your own profiling, simply set _PCMK_callgrind_enabled_ to either _yes_
or to the name of a Pacemaker daemon you wish to profile. Eg.
_PCMK_callgrind_enabled=cib_

Overall, the CIB (which is the main bottleneck in a large cluster) and the
Policy Engine are about 70% faster.

The improvements will be available with 1.1.4 is released next month, or from
our [1.1 code repository](http://hg.clusterlabs.org/pacemaker/1.1/) right now.

A summary of the various changes and description of future work is below. Any
assistance in further optimization would be appreciated :-)

-- Andrew

## PE

Use case:

* 100 nodes 
* 100 clones, clone-max=100 (10,000 effective resources) 
* 100 resource location constraints

Baseline: with probes 20-30 minutes  
Baseline: without probes 28s

### Phase 1

Use hashtables instead of lists for stores the available nodes for a resource  
New time without probes: 18s

### Phase 2

Defer creation of deletion,promote and demote constraints until they are
needed  
New time without probes: 13s

### Phase 3

Use g_list_prepend() instead of g_list_append() for the list of ordering
constraints  
New time without probes: 5s

### Phase 4

New algorithm for determining which clone instances need probing  
New time with probes: 31s

### Future work

  * Further improve the algorithm for determining which resources need to be probed
  * Further optimize the algorithm for enforcing ordering constraints

## CIB

The CIB was harder to profile. Rather than give it one large task to chew
through and see how long it took using a few printf's to provide granularity,
I had to run it through a profiler while it was operating in a real cluster
and see where most of the time was being spent.

### Phase 1

Remove most uses of cib_msg_copy(), reduced the amount of needless copying.

Phase speedup: 10%

### Phase 2

Compression costs a LOT, don't do it unless we're hitting message limits. For
now, use 256k as the threshold at which compression kicks in. The previous
limit was 10k, compressing 184 of 1071 messages accounted for 23% of the total
CPU used by the cib.

Each time we validated the CIB, we were re-reading and re-parsing the RelaxNG
schema, which accounted for 28% of the CIB's CPU usage on the DC. We now read
it once and cache the result for the life of the CIB process.

Phase speedup: 51%

### Phase 3

Push detection of group and set ordering changes to (the less busy) slave
instances. This detection was costing 15% of the CIB's total CPU time on the
DC.

Phase speedup: 15%

### Future work

The majority of CPU spent by the CIB is in post-processing.

  * Detecting what changed so we can minimize the network load: _diff_xml_object_, 35.5% CPU time
  * Calculating the current digest so peers can verify the diffs and detect ordering changes: _calculate_xml_digest_, 31% CPU time

