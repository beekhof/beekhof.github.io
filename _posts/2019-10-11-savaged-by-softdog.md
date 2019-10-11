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

Additionally, some workloads may require at-most-one semantics.  Failures affecting these kind of workloads risk data loss and/or corruption if "lost" nodes remain at least partially functional.  For this reason we need to know that the node has reached a safe state before initiating recovery of the workload.  The process of putting the node into a safe state is called fencing.

There are two categories of fencing which I will call _direct_ and _indirect_ but could equally be called _active_ and _passive_.

Direct methods involve action on the part of surviving peers, such interacting with an IPMI or iLO device, whereas indirect methods rely on the failed node to somehow recognise it is in an unhealthy state take steps to enter a safe state on its own.  

The most common form of indirect fencing is the use of a [watchdog](https://en.wikipedia.org/wiki/Watchdog_timer). The watchdog's timer is reset every N seconds unless quorum is lost or the part of the software stack fails.  If the timer (usually some multiple of N) expires, then the the watchdog will panic (not shutdown) the machine. 

When done right, watchdogs can allow survivors to safely assume that missing nodes have entered a safe state after a defined period of time.  

However when relying on indirect fencing mechanisms, it is important to recognise that in the absence of out-of-band communication such as disk based heartbeats, surviving peers have absolutely no ability to validate that the lost node ever reaches a safe state, surviving peers are making an *assumption* when they start recovery.   There is a risk it didn’t happen as planned and the cost of getting it wrong is data corruption and/or loss.

Nothing is without risk though.  Someone with an overdeveloped sense of paranoia and an infinite budget could buy all of Amazon, plus Microsoft and Google for redundancy, to host a static website - and still be undone by an asteroid.  The goal of HA is not to eliminate risk, but reduce it to an acceptable level.  What constitutes an acceptable risk varies person-to-person, project-to-project, and company-to-company, however as a community we encourage people to start by eliminating [single points of failure (SPoF)](https://en.wikipedia.org/wiki/Single_point_of_failure).

In the absence of direct fencing mechanisms, we like hardware based watchdogs because as a self-contained device they can panic the machine without involvement from the host OS.  If the watchdog fails, the node is still healthy and data loss can only occur through failure of additional nodes.  In the event of a power outage, they also loose power but the node is already safe. A network failure is no longer a SPoF and would require a software bug (incorrect quorum calculations for example) in order to present a problem.  Kernel panics, userspace hangs, and resource starvation issues all prevent the watchdog from being tickled and result in machine death.

Its this last class of failures that concern HA and kernel experts whenever Softdog is put forward in situations where already purchased cluster machines lack both power management and watchdog hardware.

If the kernel encounters a bug and corrupts its structures, there's no guarantee that the software watchdog will be called. 

> If the user is serious about reliability, he should get a hardware watchdog.
> - Mikulas Patocka (Red Hat kernel engineer)

For example, this piece of code, when executed inside the kernel on the 
same CPU as the watchdog timer, would prevent the software watchdog from 
firing.

````
        unsigned long long i;
        local_irq_disable();
        for (i = 0; i < 100000000000ULL; i++) {
                asm volatile ("nop");
        }
        local_irq_enable();
````

There are many similar examples like this and the software watchdog can't 
do anything about them. 

> If the customer needs guaranteed reboot, he should  install a hardware watchdog.
> - Mikulas Patocka (Red Hat kernel engineer)

