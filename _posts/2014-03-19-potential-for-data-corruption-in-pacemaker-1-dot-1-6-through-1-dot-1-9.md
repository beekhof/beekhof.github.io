---

title: "Potential for data corruption affecting Pacemaker 1.1.6 through 1.1.9"
date: 2014-03-19 13:24
comments: true
tags: 
- announce
---

It has come to my attention that the potential for data corruption
exists in Pacemaker versions 1.1.6 to 1.1.9

> Everyone is strongly encouraged to upgrade to 1.1.10 or later.

Those using RHEL 6.4 or later (or a RHEL clone) should already have
access to 1.1.10 via the normal update channels.

At issue is some faulty logic in a function called
`tengine_stonith_notify()` which can incorrectly add successfully
fenced nodes to a list, causing Pacemaker to subsequently erase that
node's status section when the next DC election occurs.

With the status section erased, the cluster thinks that node is safely
down and begins starting any services it has on other nodes - despite
those already being active.

In order to trigger the logic, the fenced node must:

1. have been the previous DC
1. been sufficently functional to request its own fencing, and
1. the fencing notification must arrive after the new DC has been
   elected, but before it invokes the policy engine

That this is the first we have heard of the issue since the problem
was introduced in August 2011, the above sequence of events is
apparently hard to hit under normal conditions.

Logs symptomatic of the issue look as follows:

    # grep -e do_state_transition -e reboot  -e do_dc_takeover -e tengine_stonith_notify -e S_IDLE /var/log/corosync.log

    Mar 08 08:43:22 [9934] lorien       crmd:     info: do_dc_takeover: 	Taking over DC status for this partition
    Mar 08 08:43:22 [9934] lorien       crmd:   notice: tengine_stonith_notify: 	Peer gandalf was terminated (st_notify_fence) by mordor for gandalf: OK (ref=10d27664-33ed-43e0-a5bd-7d0ef850eb05) by client crmd.31561
    Mar 08 08:43:22 [9934] lorien       crmd:   notice: tengine_stonith_notify: 	Notified CMAN that 'gandalf' is now fenced
    Mar 08 08:43:22 [9934] lorien       crmd:   notice: tengine_stonith_notify: 	Target may have been our leader gandalf (recorded: <unset>)
    Mar 08 09:13:52 [9934] lorien       crmd:     info: do_dc_takeover: 	Taking over DC status for this partition
    Mar 08 09:13:52 [9934] lorien       crmd:   notice: do_dc_takeover: 	Marking gandalf, target of a previous stonith action, as clean
    Mar 08 08:43:22 [9934] lorien       crmd:     info: do_state_transition: 	State transition S_INTEGRATION -> S_FINALIZE_JOIN [ input=I_INTEGRATED cause=C_FSA_INTERNAL origin=check_join_state ]
    Mar 08 08:43:28 [9934] lorien       crmd:     info: do_state_transition: 	State transition S_FINALIZE_JOIN -> S_POLICY_ENGINE [ input=I_FINALIZED cause=C_FSA_INTERNAL origin=check_join_state ]

Note in particular the final entry from `tengine_stonith_notify()`:

    Target may have been our leader gandalf (recorded: <unset>)

If you see this after `Taking over DC status for this partition` but
prior to `State transition S_FINALIZE_JOIN -> S_POLICY_ENGINE`, then
you are likely to have resources running in more than one location
after the next DC election.

The issue was fixed during a routine cleanup prior to Pacemaker-1.1.10 in
[@f30e1e43](https://github.com/ClusterLabs/pacemaker/commit/f30e1e43)
However the implications of what the old code allowed were not fully
appreciated at the time.
