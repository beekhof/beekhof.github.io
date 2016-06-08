---

title: "Debugging the Policy Engine"
date: 2013-05-20 11:49
comments: true
tags:
- tips
- documentation
---

## Finding the right node

The Policy Engine is the component that takes the cluster's current
state, decides on the optimal next state and produces an ordered list
of actions to achieve that state.

You can get a summary of what the cluster did in response to resource
failures and nodes joining/leaving the cluster by looking at the logs
from `pengine`:

    grep -e pengine\\[ -e pengine: /var/log/messages

> Although the `pengine` process is active on all cluster nodes, it is
> only doing work on one of them.  The "active" instance is chosen
> through the `crmd`'s DC election process and may move around as
> nodes leave/join the cluster.

If you do not see anything from `pengine` *at the time the problem
occurs*, continue to the next machine.

If you do not see anything from `pengine` *on any node*, check your
cluster if logging to syslog is enabled and the syslog configuration
to see where it is being sent.  If in doubt, refer to [Pacemaker Logging](/blog/2013/pacemaker-logging/).

Once you have located the correct node to investigate, the first thing
to do is look for the terms ERROR and WARN, eg.

    grep -e pengine\\[ -e pengine: /var/log/messages | grep -e ERROR -e WARN

This will highlight any problems the software encountered.

Next expand the query to all `pengine` logs:

    grep -e pengine\\[ -e pengine: /var/log/messages

The output will look a little like:

    pengine[6132]:   notice: LogActions: Move	 mysql	(Started corosync-host-1 -> corosync-host-4)
    pengine[6132]:   notice: LogActions: Start   www	(corosync-host-6)
    pengine[6132]:   notice: process_pe_message: Calculated Transition 7: /var/lib/pacemaker/pengine/pe-input-4424.bz2
    pengine[6132]:   notice: process_pe_message: Calculated Transition 8: /var/lib/pacemaker/pengine/pe-input-4425.bz2

In the above logs, *transition 7* resulted in `mysql` being moved and `www` being started.
Later, *transition 8* occurred but everything was where it should be and no action was required.

Other notable entries include:

    pengine[6132]:  warning: cluster_status: We do not have quorum - fencing and resource management disabled
    pengine[6132]:   notice: stage6: Scheduling Node corosync-host-1 for shutdown
    pengine[6132]:  warning: stage6: Scheduling Node corosync-host-8 for STONITH

as well as

    pengine[6132]:   notice: LogActions: Start   Fencing      (corosync-host-1 - blocked)

which indicates that the cluster would like to start the `Fencing` resource, but some dependancy is not satisfied.

    pengine[6132]:  warning: determine_online_status: Node corosync-host-8 is unclean

which indicates that either `corosync-host-8` has failed, or a resource on it has failed to stop when requested.

    pengine[6132]:  warning: unpack_rsc_op: Processing failed op monitor for www on corosync-host-4: unknown error (1)

which indicates a health check for the `www` resource failed with a
return code of `1` (aka. `OCF_ERR_GENERIC`).  See [Pacemaker Explained](http://clusterlabs.org/doc/en-US/Pacemaker/1.1-plugin/html/Pacemaker_Explained/s-ocf-return-codes.html)
for more details on OCF return codes.

- Is there anything from the Policy Engine at about the time of the problem?  
  If not, go back to the `crmd` logs and see why no recovery was attempted.

- Did `pengine` log why something happened? does that sound correct?  
  Excellent, thanks for playing.

## Getting more detail from the Policy Engine

The job performed by the Policy engine is a very complex and frequent
task, so to avoid filling up the disk with logs, it only indicates
what it is doing and rarely the reason why.  Normally the *why* can be
found in the `crmd` logs, but it also saves the *current state* (the
cluster configuration and the state of all resources) to disk for
situations when it can't.

These files can later be replayed using `crm_simulate` with a higher
level of verbosity to diagnose issues and, as part of our regression
suite, to make sure they stay fixed afterwards.

Finding these state files is a matter of looking for logs such as

    crmd[1811]:   notice: run_graph: Transition 2 (... Source=/var/lib/pacemaker/pengine/pe-input-473.bz2): Complete
    pengine[1810]:   notice: process_pe_message: Calculated Transition 0: /var/lib/pacemaker/pengine/pe-input-473.bz2

The "correct" entry will depend on the context of your query.

> Please note, sometimes events occur while the `pengine` is
> performing its calculation.  In this situation, the calculation
> logged by `process_pe_message()` is discarded and a new one
> performed.
> As a result, not all transitions/files listed by the
> `pengine` process are executed by the `crmd`.

After obtaining the file named by `run_graph()` or
`process_pe_message()`, either directly or from a `crm_report`
archive, pass it to `crm_simulate` which will display its view of the
cluster at that time:

    crm_simulate --xml-file ./pe-input-473.bz2

- Does the cluster state look correct?  

  If not, file a bug.  It is possible we have misparsed the state of
  the resources, any calculation we make based on this would therefor
  also be wrong.

Next, see what recovery actions the cluster thinks need to be performed:

    crm_simulate --xml-file ./pe-input-473.bz2 --save-graph problem.graph --save-dotfile problem.dot --run

In addition to the normal output, this command creates:

- `problem.graph`, the ordered graph of actions, their parameters and prerequisites
- `problem.dot`, a more human readable version of the same graph focussed on the action ordering.

Open `problem.dot` in `dotty` or `graphviz` to obtain a graphical representation:

* Arrows indicate ordering dependencies
* Dashed-arrows indicate dependencies that are not present in the transition graph
* Actions with a dashed border of any color do not form part of the transition graph
* Actions with a green border form part of the transition graph
* Actions with a red border are ones the cluster would like to execute but cannot run
* Actions with a blue border are ones the cluster does not feel need to be executed
* Actions with orange text are pseudo/pretend actions that the cluster uses to simplify the graph
* Actions with black text are sent to the `lrmd`
* Resource actions have text of the form `${rsc}_${action}_${interval} ${node}`
* Actions of the form `${rsc}_monitor_0 ${node}` is the cluster's way
  of finding out the resource's status **before** we try and start it
  anywhere
* Any action depending on an action with a red border will not be able to execute.
* Loops are really bad. Please report them to the development team.

Check the relative ordering of actions:

- Are there any extra ones?  
  Do they need to be removed from the configuration?  
  Are they implied by the `group` construct?  
- Are there any missing?  
  Are they specified in the configuration?

You can obtain excruicating levels of detial by adding additional `-V` options to the `crm_simulate` command line.

Now see what the cluster thinks the "next state" will look like:

    crm_simulate --xml-file ./pe-input-473.bz2 --save-graph problem.graph --save-dotfile problem.dot --simulate

- Does the new cluster state look correct based on the input and actions performed?  
  If not, file a bug.
