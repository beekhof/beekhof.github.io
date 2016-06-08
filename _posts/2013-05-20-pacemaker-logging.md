---

title: "Pacemaker Logging"
date: 2013-05-20 13:43
comments: true
tags:
- tips
- documentation
---

### Normal operation

Pacemaker inherits most of its logging setting from either CMAN or
Corosync - depending on what its running on top of.

In order to avoid spamming syslog, Pacemaker only logs a summary of
its actions (NOTICE and above) to syslog.

If the level of detail in syslog is insufficient, you should enable a
cluster log file.  Normally one is configured by default and it
contains everything except debug and trace messages.

To find the location of this file, either examine your CMAN
(`cluster.conf`) or Corosync (`corosync.conf`) configuration file or
look for syslog entries such as:

    pacemakerd[1823]:   notice: crm_add_logfile: Additional logging available in /var/log/cluster/corosync.log

If you do not see a line like this, either update the cluster
configuration or set `PCMK_debugfile` in `/etc/sysconfig/pacemaker`

> `crm_report` also knows how to find all the Pacemaker related logs
> and blackbox files

If the level of detail in the cluster log file is still insufficient,
or you simply wish to go blind, you can turn on debugging in
Corosync/CMAN, or set `PCMK_debug` in `/etc/sysconfig/pacemaker`.

A minor advantage of setting `PCMK_debug` is that the value can be a
comma-separated list of processes which should produce debug logging
instead of a global yes/no.

### When an ERROR occurs

Pacemaker includes support for a blackbox.

When enabled, the blackbox contains a rolling buffer of all logs (not
just those sent to syslog or a file) and is written to disk after a
crash or assertion failure.

The blackbox recorder can be enabled by setting `PCMK_blackbox` in
`/etc/sysconfig/pacemaker` or at runtime by sending SIGUSR1. Eg.

    killall -USR1 crmd

When enabled you'll see a log such as:

    crmd[1811]:   notice: crm_enable_blackbox: Initiated blackbox recorder: /var/lib/pacemaker/blackbox/crmd-1811

If a crash occurs, the blackbox will be available at that location.
To extract the contents, pass it to `qb-blackbox`:

    qb-blackbox /var/lib/pacemaker/blackbox/crmd-1811

Which produces output like:

    Dumping the contents of /var/lib/pacemaker/blackbox/crmd-1811
    [debug] shm size:5242880; real_size:5242880; rb->word_size:1310720
    [debug] read total of: 5242892
    Ringbuffer:
     ->NORMAL
     ->write_pt [5588]
     ->read_pt [0]
     ->size [1310720 words]
     =>free [5220524 bytes]
     =>used [22352 bytes]
    ...
    trace   May 19 23:20:55 gio_read_socket(368):0: 0x11ab920.5 1 (ref=1)
    trace   May 19 23:20:55 pcmk_ipc_accept(458):0: Connection 0x11aee00
    info    May 19 23:20:55 crm_client_new(302):0: Connecting 0x11aee00 for uid=0 gid=0 pid=24425 id=0e943a2a-dd64-49bc-b9d5-10fa6c6cb1bd
    debug   May 19 23:20:55 handle_new_connection(465):2147483648: IPC credentials authenticated (24414-24425-14)
    ...
    [debug] Free'ing ringbuffer: /dev/shm/qb-create_from_file-header

When an ERROR occurs you'll also see the function and line number that produced it such as:

    crmd[1811]: Problem detected at child_death_dispatch:872 (mainloop.c), please see /var/lib/pacemaker/blackbox/crmd-1811.1 for additional details
    crmd[1811]: Problem detected at main:94 (crmd.c), please see /var/lib/pacemaker/blackbox/crmd-1811.2 for additional details

Again, simply pass the files to `qb-blackbox` to extract and query the contents.

> Note the a counter is added to the end so as to avoid name collisions.

### Diving into files and functions

In case you have not already guessed, all logs include the name of the
function that generated them.  So:

    crmd[1811]:   notice: crm_update_peer_state: cman_event_callback: Node corosync-host-1[1] - state is now lost (was member)

came from the function `crm_update_peer_state()`.

To obtain more detail from that or any other function, you can set
`PCMK_trace_functions` in `/etc/sysconfig/pacemaker` to a comma
separated list of function names. Eg.

    PCMK_trace_functions=crm_update_peer_state,run_graph

For a bigger stick, you may also activate trace logging for all the
functions in a particular source file or files by setting
`PCMK_trace_files` as well.

    PCMK_trace_files=cluster.c,election.c

These additional logs are sent to the cluster log file.
Note that enabling tracing options also alters the output format.

Instead of:

    crmd:  notice: crm_cluster_connect: 	Connecting to cluster infrastructure: cman

the output includes file and line information:

    crmd: (   cluster.c:215   )  notice: crm_cluster_connect: 	Connecting to cluster infrastructure: cman


### But wait there's still more

Still need more detail?  You're in luck!  The blackbox can be dumped
at any time, not just when an error occurs.

First, make sure the blackbox is active (we'll assume its the `crmd` that needs to be debugged):

    killall -USR1 crmd

Next, discard any previous contents by dumping them to disk

    killall -TRAP crmd

now cause whatever condition you're trying to debug, and send `-TRAP`
when you're ready to see the result.

    killall -TRAP crmd

You can now look for the result in syslog:

    grep -e crm_write_blackbox: /var/log/messages

This will include a filename containing the trace logging:

    crmd[1811]:   notice: crm_write_blackbox: Blackbox dump requested, please see /var/lib/pacemaker/blackbox/crmd-1811.1 for contents
    crmd[1811]:   notice: crm_write_blackbox: Blackbox dump requested, please see /var/lib/pacemaker/blackbox/crmd-1811.2 for contents

To extract the trace loging for our test, pass the most recent file to `qb-blackbox`:

    qb-blackbox /var/lib/pacemaker/blackbox/crmd-1811.2

At this point you'll probably want to use grep :)
