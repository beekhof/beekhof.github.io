---

title: "Debugging Pacemaker"
date: 2013-05-20 09:08
comments: true
tags:
- tips
- documentation
---

## Where to start

The first thing to do is look in `syslog` for the terms ERROR and WARN, eg.

    grep -e ERROR -e WARN /var/log/messages

If nothing looks appropriate, find the logs from `crmd`

    grep -e crmd\\[ -e crmd: /var/log/messages

If you do not see anything from `crmd`, check your cluster if logging
to syslog is enabled and the syslog configuration to see where it is
being sent.  If in doubt, refer to [Pacemaker Logging](/blog/2013/pacemaker-logging/) for how to obtain more detail.

> Although the `crmd` process is active on all cluster nodes,
> decisions are only occuring on one of them.  The "DC" is chosen
> through the `crmd`'s election process and may move around as nodes
> leave/join the cluster.

> For node failures, you'll always want the logs from the DC (or the
> node that becomes the DC).  
> For resource failures, you'll want the logs from the DC and the node
> on which the resource failed.

Log entries like:

    crmd[1811]:   notice: crm_update_peer_state: cman_event_callback: Node corosync-host-1[1] - state is now lost (was member)

indicate a node is no longer part of the cluster (either because it failed or was shut down)

    crmd[1811]:   notice: crm_update_peer_state: cman_event_callback: Node corosync-host-1[1] - state is now member (was lost)

indicates a node has (re)joined the cluster

    crmd[1811]:   notice: do_state_transition: State transition S_IDLE -> S_POLICY_ENGINE ...
    crmd[1811]:   notice: run_graph: Transition 2 (... Source=/var/lib/pacemaker/pengine/pe-input-473.bz2): Complete
    crmd[1811]:   notice: do_state_transition: State transition S_TRANSITION_ENGINE -> S_IDLE ...

indicates recovery was attempted

    crmd[1811]:   notice: te_rsc_command: Initiating action 36: monitor www_monitor_0 on corosync-host-5
    crmd[1811]:   notice: te_rsc_command: Initiating action 54: monitor mysql_monitor_10000 on corosync-host-4

indicates we performed a resource action, in this case we are checking
the status of the `www` resource on `corosync-host-5` and starting a
recurring health check for `mysql` on `corosync-host-4`.

    crmd[1811]:   notice: te_fence_node: Executing reboot fencing operation (83) on corosync-host-8 (timeout=60000)

indicates that we are attempting to fence `corosync-host-8`.

    crmd[1811]:   notice: tengine_stonith_notify: Peer corosync-host-8 was terminated (st_notify_fence) by corosync-host-1 for corosync-host-1: OK

indicates that `corosync-host-1` successfully fenced `corosync-host-8`.

### Node-level failures

- Did the `crmd` fail to notice the failure?  

  If you do not see any entries from `crm_update_peer_state()`, check
  the corosync logs to see if membership was correct/timely

- Did the `crmd` fail to initiate recovery?

  If you do not see entries from `do_state_transition()` and
  `run_graph()`, then the cluster failed to react at all.  Refer to
  [Pacemaker Logging](/blog/2013/pacemaker-logging/) to for how to
  obtain more detail about why the `crmd` ignored the failure.

- Did the `crmd` fail to perform recovery?  

  If you DO see entries from `do_state_transition()` but the
  `run_graph()` entry(ies) include the text `Complete=0, Pending=0,
  Fired=0, Skipped=0, Incomplete=0`, then the cluster did not think it
  needed to do anything.

  Obtain the file named by `run_graph()`
  (eg. `/var/lib/pacemaker/pengine/pe-input-473.bz2`) either directly
  or from a `crm_report` archive and continue to [debugging the Policy Engine](/blog/2013/debugging-pengine/).

- Was fencing attempted?  

  Check if the `stonith-enabled` property is set to true/1/yes, if so
  obtain file named by `run_graph()`
  (eg. `/var/lib/pacemaker/pengine/pe-input-473.bz2`) either directly
  or from a `crm_report` archive and continue to [debugging the Policy Engine](/blog/2013/debugging-pengine/).

- Did fencing complete?  

  Check the configuration of fencing resources and if so proceed to
  [Debugging Stonith](#tba).

### Resource-level failures

- Did the resource actually fail?  

  If not, check for logs matching the resource name to see why the
  resource agent thought a failure occurred.  

  Check the resource agent source to see what code paths could have
  produced those logs (or the lack of them)

- Did `crmd` notice the resource failure?  

  If not, check for logs matching the resource name to see if the
  resource agent noticed.  

  Check a recurring monitor was configured.

- Did the `crmd` fail to initiate recovery?  

  If you do not see entries from `do_state_transition()` and
  `run_graph()`, then the cluster failed to react at all.  Refer to
  [Pacemaker Logging](/blog/2013/pacemaker-logging/) to for how to
  obtain more detail about why the `crmd` ignored the failure.

- Did the `crmd` fail to perform recovery?  

  If you DO see entries from `do_state_transition()` but the
  `run_graph()` entry(ies) include the text `Complete=0, Pending=0,
  Fired=0, Skipped=0, Incomplete=0`, then the cluster did not think it
  needed to do anything.

  Obtain the file named by `run_graph()`
  (eg. `/var/lib/pacemaker/pengine/pe-input-473.bz2`) either directly
  or from a `crm_report` archive and continue to [debugging the Policy Engine](/blog/2013/debugging-pengine/).

- Did resources stop/start/move unexpectedly or fail to stop/start/move when expected?  

  Obtain the file named by `run_graph()`
  (eg. `/var/lib/pacemaker/pengine/pe-input-473.bz2`) either directly
  or from a `crm_report` archive and continue to [debugging the Policy Engine](/blog/2013/debugging-pengine/).
