---
layout: post
title: "Large Cluster Performance: Redux"
date: 2013-02-18 10:49
comments: true
tags:
- performance
---

Normally I have access to 4 virtual cluster nodes on my laptop, however for the first time since leaving SUSE, I have had the opportunity to test with 8 and 16 node clusters.

> Performance testing doesn't just benefit large clusters, it helps reduce Pacemaker's footprint for smaller ones too

I am very pleased to say that 8-nodes 'just worked'.
The fun began when I tried to step up to 16 nodes.

> When Pacemaker 1.1.9 is released, it can confidently claim to support 16 node clusters

By far the biggest roadbloack turned out to be a [memory allocation issue in libxml2](https://bugzilla.redhat.com/show_bug.cgi?id=906610)'s `xmlNodeDump()` function.
Essentially, CIB updates that would normally complete in 10'ths of a second would, at the worst possible time, begin to take upwards of 18 seconds.
This would cause the CIB to get backed up, updates would start timing out and the cluster would produce even more CIB updates while attempting to recover. Compounding the problem.

> As a result the cluster began falling over even before it could be brought fully online.

I was able to work-around this problem (and make some additional optimizations) with the following two commits:

	55f8a94: Refactor: Use a custom xml-to-string function for performance
	0066122: Refactor: Core: A faster and more consistant digest function

With such large clusters, we also started hitting IPC limits.
If we were lucky, this meant the message didn't get through.

Otherwise, the server (cib) got into a tightly coupled send/recv loop
with a potenitally slow client - causing the cib to get backed up,
updates would start timing out and the cluster would produce even more
CIB updates while attempting to recover. (Are you seeing the pattern
yet?)

To account for this, the cluster now:

* compresses messages that exceed the limit
* uses shared memory for IPC so we know in advance if trying to send the current message would block
* queues messages that would otherwise have blocked (and disconnects clients if they fall too far behind)

These are the relevant commits:

	3c56aa1: Feature: Reliably detect when an IPC message size exceeds the connection's maximum
	14bbe6c: Feature: Compress messages that exceed the configured IPC message limit
	566db4f: Feature: IPC: Use queues to prevent slow clients from blocking the server
	85543e6: Feature: Use shared memory for IPC by default

Additionally, a previous decision to broadcast updates even when nothing changed was determined to be causing significant and unnecessary load throughout the cluster.
This and other updates that made more efficient use of the available system resources also helped:

 	004d515: High: cib: Performance improvements for non-DC nodes
	01baad5: Medium: crmd: Don't go into a tight loop while waiting for something to happen
	27b6306: High: crmd: Only call set_slave() if we were the DC to avoid spamming the the cib
	a3bba8d: Refactor: cib: Construct each notification once, not once per client
	24a7ec2: Refactor: ipc: Allow messages to be constructed once and sent multiple times
	e72c348: Medium: cib: Improve performance by only validating every 20th slave update
	21dfddc: High: crmd: Have cib operation timeouts scale with node count

### Results

> The results were quite remarkable

The graphs below shows the 1, 5 and 15 minute load averages on while the cluster ran each CTS test once.

This is node 1 (of 4) running Pacemaker 1.0.5 back in 2009:

![1.0.5 - 4-nodes](/images/Pacemaker-1.0.5-node-1-of-4.png)

Here is the current code running a CMAN cluster *twice* as big:

![1.1.9 - 8-nodes](/images/Pacemaker-1.1.9-node-1-of-8.png)

> Note the difference in scale on the Y-axis (double in 2009)

The difference is even more pronounced for larger clusters.
Here we have (node 1 of) a 13 node cluster running 1.0.5 again:

![1.0.5 - 13-nodes](/images/Pacemaker-1.0.5-node-1-of-13.png)

> Note the load **almost hits 8**

compared with a 16 node Corosync 2.x cluster with 1.1.9:

![1.1.9 - 16-nodes](/images/Pacemaker-1.1.9-node-1-of-16.png)

> Note the load still **barely spikes above 1** even though the size of the cluster is over 30% larger than it was in 2009.

Even with the benefit of slightly more modern hardware, thats a significant improvement.

### Testing Notes

Unfortunately, these tests could not be a true apples-for-apples comparision.

The 16 node cluster was comprised physical hardware with 8Gb RAM and two:

* model name	: Dual-Core AMD Opteron(tm) Processor 2212 HE
* cpu MHz	: 1000.000
* cache size	: 1024 KB
* bogomips	: 2000.04

The 8 node cluster was comprised of virtual machines with 512Mb RAM and one:

* model name	: AMD Opteron 23xx (Gen 3 Class Opteron)
* cpu MHz	: 2099.998
* cache size	: 512 KB
* bogomips	: 4199.99

The machines from 2009 had 512MB RAM and (we believe) one 2 GHz HT Xeon (which looks comparable to the virtual machines in the 8-node case).
The hardware appears to have met its demise so we're unable to find more details.

If someone has the time/energy/interest to re-run the 1.0.5 tests on modern hardware, please get in touch :-)
