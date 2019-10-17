---
date: 2019-10-11 13:55
title: Savaged by Softdog, a Cautionary Tale
excerpt: Hardware is imperfect, and software contains bugs. Don't use software based watchdogs and expect to survive the latter.
published: false
tags:
- high availability
- watchdog
---

Hardware is imperfect, and software contains bugs. When node level failures occur, the work required from the cluster does not decrease - affected workloads need to be restarted, putting additional stress on surviving peers and making it important to recover the lost capacity.

Additionally, some of workloads may require at-most-one semantics.  Failures affecting these kind of workloads risk data loss and/or corruption if "lost" nodes remain at least partially functional.  For this reason we need to know that the node has reached a safe state before initiating recovery of the workload.  

The process of putting the node into a safe state is called fencing, and we prefer power based methods because they provide the best chance of also recovering capcity without human involvement.

There are two categories of fencing which I will call _direct_ and _indirect_ but could equally be called _active_ and _passive_.

Direct methods involve action on the part of surviving peers, such interacting with an IPMI or iLO device, whereas indirect methods rely on the failed node to somehow recognise it is in an unhealthy state take steps to enter a safe state on its own.  

The most common form of indirect fencing is the use of a [watchdog](https://en.wikipedia.org/wiki/Watchdog_timer). The watchdog's timer is reset every N seconds unless quorum is lost or the part of the software stack fails.  If the timer (usually some multiple of N) expires, then the the watchdog will panic (not shutdown) the machine. 

When done right, watchdogs can allow survivors to safely assume that missing nodes have entered a safe state after a defined period of time.  

However when relying on indirect fencing mechanisms, it is important to recognise that in the absence of out-of-band communication such as disk based heartbeats, surviving peers have absolutely no ability to validate that the lost node ever reaches a safe state, surviving peers are making an *assumption* when they start recovery.   There is a risk it didn’t happen as planned and the cost of getting it wrong is data corruption and/or loss.

Nothing is without risk though.  Someone with an overdeveloped sense of paranoia and an infinite budget could buy all of Amazon, plus Microsoft and Google for redundancy, to host a static website - and still be undone by an asteroid.  The goal of HA is not to eliminate risk, but reduce it to an acceptable level.  What constitutes an acceptable risk varies person-to-person, project-to-project, and company-to-company, however as a community we encourage people to start by eliminating [single points of failure (SPoF)](https://en.wikipedia.org/wiki/Single_point_of_failure).

In the absence of direct fencing mechanisms, we like hardware based watchdogs because as a self-contained device they can panic the machine without involvement from the host OS.  If the watchdog fails, the node is still healthy and data loss can only occur through failure of additional nodes.  In the event of a power outage, they also loose power but the node is already safe. A network failure is no longer a SPoF and would require a software bug (incorrect quorum calculations for example) in order to present a problem.  

There is one last class of failures, software bugs, that are the primary concern HA and kernel experts whenever Softdog is put forward in situations where already purchased cluster machines lack both power management and watchdog hardware.

Softdog malfunctions originating in software can take two forms - resetting a machine when it should not have (false positive), and not resetting a machine when it should have (false negative). False positives will reduce overall availability due to repeated failovers, but the integrity of the system and its data will remain intact.

More concerning is the possibility for a single software bug to both cause a node to become unavailable and prevent softdog from recovering the system.   One contrived example is this piece of code which, when executed inside the kernel on the same CPU as the watchdog timer, would prevent the software watchdog from firing by staving the it of the CPU.

```c
        unsigned long long i;
        local_irq_disable();
        for (i = 0; i < 100000000000ULL; i++) {
                asm volatile ("nop");
        }
        local_irq_enable();
```

There are many similar examples like this and the software watchdog can't do anything about them. Likewise if the kernel encounters a bug and corrupts its structures, there's no guarantee that the software watchdog will be called.  [XXXXX insert link to an example RHBZ XXXXX] 

> If the customer needs guaranteed reboot, they should install a hardware watchdog.
>
>   &mdash; Mikulas Patocka (Red Hat kernel engineer)

The greatest danger of softdog is that most of the time it appears to work.  For months or years it will reboot your machines in response to network and software outages, only to fail you when just the wrong conditions are met.  

Imagine some approximation of the above loop existing in an obscure corner of a device driver, or your codebase.  Rarely triggered, but one day you get unlucky and the loop is triggered and lands on the same CPU as softdog.

Just like all the other times it causes the cluster software to go wonky, but the surviving peers detect it, wait a minute or two, and then begin recovery.  Application services are started, volumes are mounted, database replicas are promoted to master, VIPs are brought up, and requests start being processed.  

However unlike all the other times, the failed peer is still active.  The softdog CPU is stalled, but there are dozens of others keeping the application services responsive and nothing has removed VIPs or demoted masters.

At this point, your best case scenario is that database and storage replication is broken.  Requests from some clients will go to the failed node, and some will go to its replacement.  Both will succeed, volumes and databases will be updated independently of what happened on the other peer.  Reads will start to return stale or otherwise inaccurate data, and incorrect decisions will be made based on them.  No transactions will be lost, however the longer the split remains, the further the datasets will drift apart and the more work it will be to reconcile them by hand once the situation is discovered.

Things get worse if replication doesn’t break.  Now you have the prospect of uncoordinated parallel access to your datasets.  Even if database locking is still somehow working, eventually those changes are persisted to disk and there is nothing to prevent both sides from writing different versions of the same backing file due to non-overlapping database updates.  

Depending on the timing and scope of the updates, you could get:

- only whole file copies from the second writer and loose transactions from the first,
- whole file copies from a mixture of hosts, leading to a corrupted on-disk representation,
- files which contain a mixture of bits from both hosts, also leading to a corrupted on-disk representation, or
- all of the above. 

Ironically an admin’s first instinct, to restart the node or database and see if that fixes the situation, might instead wipe out the only remaining consistent copy of their data (asuming the entire database fits in memory).  At which point all transactions since the previous backup are lost.

To mitigate this situation, you would either need *very* frequent backups, or add a SCSI based fencing mechanism to ensure exclusive access to shared storage, and a network based mechanism to prevent requests from reaching the failed peer.

Or you could just use a hardware watchdog (or even better, a [network power switch](https://www.apc.com/shop/us/en/categories/power-distribution/rack-power-distribution/metered-rack-pdu/N-wj7jiz)).
